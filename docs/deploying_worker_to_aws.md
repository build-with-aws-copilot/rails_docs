!!! info
    By default, backend services will be launched in public subnet, and therefore have internet access. You can also launch backend services in private subnet, and use NAT gateway to access the internet.

## Add Environment Variables to AWS

```
copilot secret init
What would you like to name this secret? [? for help] REDIS_HOST
What is the value of secret REDIS_HOST in environment prod?
What is the value of secret REDIS_HOST in environment staging?
```

```
copilot secret init
What would you like to name this secret? [? for help] REDIS_PORT
What is the value of secret REDIS_PORT in environment prod?
What is the value of secret REDIS_PORT in environment staging?
```

## Setup Sidekiq

config/routes.rb
```
require 'sidekiq/web'

Rails.application.routes.draw do
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  mount Sidekiq::Web => '/sidekiq'

  resources :books, only: %i[index show]

  resources :orders, only: %i[index show]

  # Defines the root path route ("/")
  # root "articles#index"
  root "rails/welcome#index"
end
```

config/initializers/sidekiq.rb
```
Sidekiq.configure_server do |config|
  config.redis = { url: "redis://#{ENV['REDIS_HOST']}:#{ENV['REDIS_PORT']}/0" }
end

Sidekiq.configure_client do |config|
  config.redis = { url: "redis://#{ENV['REDIS_HOST']}:#{ENV['REDIS_PORT']}/0" }
end
```

config/sidekiq_worker.yml
```
# Jobs in low priority queue are not done until default queue is empty.
# The order of the queues in this file is the order in which jobs are done.

:concurrency: 5
:queues:
  - default
  - low
```

config/environments/production.rb
```
config.active_job.queue_adapter = :sidekiq
```

## Create Dockerfile.sidekiq

```
# syntax = docker/dockerfile:1.4

# Make sure RUBY_VERSION matches the Ruby version in .ruby-version and Gemfile
ARG RUBY_VERSION=2.7.2
FROM ruby:$RUBY_VERSION-slim as base

# Rails app lives here
WORKDIR /rails

# Set production environment
ENV RAILS_ENV="production" \
    BUNDLE_WITHOUT="development:test" \
    BUNDLE_DEPLOYMENT="1"

# Update gems and bundler
RUN gem update --system --no-document && \
    gem install -N bundler


# Throw-away build stage to reduce size of final image
FROM base as build

# Install packages needed to build gems
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y build-essential libpq-dev pkg-config redis-tools

# Install application gems
COPY --link Gemfile Gemfile.lock ./
RUN bundle install && \
    bundle exec bootsnap precompile --gemfile && \
    rm -rf ~/.bundle/ $BUNDLE_PATH/ruby/*/cache $BUNDLE_PATH/ruby/*/bundler/gems/*/.git

# Copy application code
COPY --link . .

# Precompile bootsnap code for faster boot times
RUN bundle exec bootsnap precompile app/ lib/

# Precompiling assets for production without requiring secret RAILS_MASTER_KEY
RUN SECRET_KEY_BASE=DUMMY ./bin/rails assets:precompile


# Final stage for app image
FROM base

# Install packages needed for deployment
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y libsqlite3-0 postgresql-client && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Run and own the application files as a non-root user for security
RUN useradd rails --home /rails --shell /bin/bash
USER rails:rails

# Copy built artifacts: gems, application
COPY --from=build /usr/local/bundle /usr/local/bundle
COPY --from=build --chown=rails:rails /rails /rails

# Deployment options
ENV RAILS_LOG_TO_STDOUT="1" \
    RAILS_SERVE_STATIC_FILES="true"

# Entrypoint sets up the container.
# ENTRYPOINT ["/rails/bin/docker-entrypoint"]

# Start the server by default, this can be overwritten at runtime
EXPOSE 3000
CMD ["bundle", "exec", "sidekiq", "-C", "/rails/config/sidekiq_worker.yml"]
```

## Worker Service
```
export AWS_PROFILE=copilot.application
copilot svc init
Which service type best represents your service's architecture? Backend Service
What do you want to name this service? worker
```
A manifest file will be generated under `copilot/worker/manifest.yml`

## Deploy Worker

Now we're ready to deploy worker to AWS. Run
```
copilot svc deploy --app rails70 --env staging --name worker # push to staging
copilot svc deploy --app rails70 --env prod --name worker # push to production
```

## Move Deployment to GitHub Actions
### Staging Environment

Add following to `./github/workflows/staging.yml`

```
name: staging worker deployment

on:
  push:
    branches: [ "develop" ]

jobs:
  copilot:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install copilot
        run: |
          mkdir -p $GITHUB_WORKSPACE/bin
          # download copilot
          curl -Lo copilot-linux https://github.com/aws/copilot-cli/releases/download/v1.27.0/copilot-linux && \
          # make copilot bin executable
          chmod +x copilot-linux && \
          # move to path
          mv copilot-linux $GITHUB_WORKSPACE/bin/copilot && \
          # add to PATH
          echo "$GITHUB_WORKSPACE/bin" >> $GITHUB_PATH

      - name: prod worker deployment
        run: copilot svc deploy -n worker -e staging -a rails70
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID_TOOLS }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY_TOOLS }}
          AWS_REGION: us-west-2
          DOCKER_BUILDKIT: 1
```

### Production Deployment

Add following `./github/workflows/prod.yml`

```
name: production worker deployment

on:
  push:
    branches: [ "main" ]

jobs:
  copilot:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install copilot
        run: |
          mkdir -p $GITHUB_WORKSPACE/bin
          # download copilot
          curl -Lo copilot-linux https://github.com/aws/copilot-cli/releases/download/v1.27.0/copilot-linux && \
          # make copilot bin executable
          chmod +x copilot-linux && \
          # move to path
          mv copilot-linux $GITHUB_WORKSPACE/bin/copilot && \
          # add to PATH
          echo "$GITHUB_WORKSPACE/bin" >> $GITHUB_PATH

      - name: prod worker deployment
        run: copilot svc deploy -n worker -e prod -a rails70
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID_TOOLS }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY_TOOLS }}
          AWS_REGION: us-west-2
          DOCKER_BUILDKIT: 1
```

## Sidekiq Running on Production

<img width="1511" alt="Screen Shot 2023-05-15 at 10 56 41 pm" src="https://github.com/build-with-aws-copilot/rails_docs/assets/129698988/fb2cc97a-71ed-4a6d-93ef-c4ea13de2f13">
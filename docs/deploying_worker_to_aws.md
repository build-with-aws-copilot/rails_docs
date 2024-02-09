!!! info
    By default, backend services will be launched in public subnet, and therefore have internet access. You can also launch backend services in private subnet, and use NAT gateway to access the internet.

## Add Environment Variables to AWS

Create environment variables for redis, so sidekiq and connect it to process jobs:

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

## Worker Service
```
export AWS_PROFILE=copilot.application
copilot svc init
Which service type best represents your service's architecture? Backend Service
What do you want to name this service? worker
```
A manifest file will be generated under `copilot/worker/manifest.yml`.

We can reuse the Dockerfile but override the `command`. So when the container starts, it will be running sidekiq, rather than web server:

`command: ["bundle", "exec", "sidekiq", "-t", "15", "-C", "/rails/config/sidekiq_worker.yml"]`

copilot/worker/manifest.yml
```
# The manifest for the "worker" service.
# Read the full specification for the "Backend Service" type at:
#  https://aws.github.io/copilot-cli/docs/manifest/backend-service/

# Your service name will be used in naming your resources like log groups, ECS services, etc.
name: worker
type: Backend Service

# Your service is reachable at "http://worker.${COPILOT_SERVICE_DISCOVERY_ENDPOINT}:3000" but is not public.

# Configuration for your containers and service.
image:
  port: 3000
  build:
    dockerfile: /Dockerfile
    args:
      AWS_S3_ACCESS_KEY_ID: ${AWS_S3_ACCESS_KEY_ID}
      AWS_S3_SECRET_ACCESS_KEY: ${AWS_S3_SECRET_ACCESS_KEY}
      S3_BUCKET_NAME: ${S3_BUCKET_NAME}
      AWS_S3_REGION: ${AWS_S3_REGION}
      CLOUDFRONT_ENDPOINT: ${CLOUDFRONT_ENDPOINT}
      APPLE_PEM_PATH: ${APPLE_PEM_PATH}
      APPLE_TEAM_ID: ${APPLE_TEAM_ID}
      APPLE_KEY_ID: ${APPLE_KEY_ID}
      REDIS_HOST: ${REDIS_HOST}
      REDIS_PORT: ${REDIS_PORT}
      AWS_REGION: ${AWS_REGION}
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      AIRBRAKE_PROJECT_ID: ${AIRBRAKE_PROJECT_ID}
      AIRBRAKE_PROJECT_KEY: ${AIRBRAKE_PROJECT_KEY}
      T_AUTHENTICATION_SECRET: ${T_AUTHENTICATION_SECRET}
  
command: ["bundle", "exec", "sidekiq", "-t", "15", "-C", "/rails/config/sidekiq_worker.yml"]
cpu: 1024       # Number of CPU units for the task.
memory: 2048    # Amount of memory in MiB used by the task.
platform: linux/x86_64     # See https://aws.github.io/copilot-cli/docs/manifest/backend-service/#platform
count: 1       # Number of tasks that should be running in your service.
exec: true     # Enable running commands in your container.
network:
  connect: true # Enable Service Connect for intra-environment traffic between services.

secrets:
  REDIS_HOST: /copilot/${COPILOT_APPLICATION_NAME}/${COPILOT_ENVIRONMENT_NAME}/secrets/REDIS_HOST
  REDIS_PORT: /copilot/${COPILOT_APPLICATION_NAME}/${COPILOT_ENVIRONMENT_NAME}/secrets/REDIS_PORT
```

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
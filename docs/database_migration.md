The Dockerfile has this line
```
ENTRYPOINT ["/rails/bin/docker-entrypoint"]
```
If we look at the content of `docker-entrypoint`
```
#!/bin/bash -e

# If running the rails server then create or migrate existing database
if [ "${*}" == "./bin/rails server" ]; then
  ./bin/rails db:prepare
fi

exec "${@}"
```

That means when a server starts, migration will be run. If there are multiple servers, migration will be run multiple times. This is not ideal, we only want migration to be run once during a deployment.

Remove this line from `Dockerfile` and `Dockerfile.sidekiq`
```
ENTRYPOINT ["/rails/bin/docker-entrypoint"]
```

Add migration to GitHub Actions:
```
name: production webserver deployment

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

      - name: prod webserver deployment
        run: copilot svc deploy -n webserver -e prod -a rails70
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID_TOOLS }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY_TOOLS }}
          AWS_REGION: us-west-2
          DOCKER_BUILDKIT: 1

      - name: prod migration
        run: copilot svc exec -n webserver -e prod -a rails70 -c './bin/rails db:prepare RAILS_ENV=production'
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID_TOOLS }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY_TOOLS }}
          AWS_REGION: us-west-2
          DOCKER_BUILDKIT: 1
```

!!! info
    The migration is run after code has been deployed. If you place the migration before the deployment, since the code hasn't been deployed, nothing will be run.
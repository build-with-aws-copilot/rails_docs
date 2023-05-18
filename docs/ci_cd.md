Place the following secrets in your GitHub settings. They are the credentials from `[copilot.application]` named profile.
```
key: AWS_ACCESS_KEY_ID_APP
value: AKIUJFHFJDFKFUBHUE88

key: AWS_SECRET_ACCESS_KEY_APP
value: +Wv3oDIUJSDHCNVBOLQKDIFUDIIIUDIKBK+vUmqc
```

## Staging Deployment

Create a file `./github/workflows/staging.yml`
```
name: staging webserver deployment

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

      - name: staging deployment
        run: copilot svc deploy --app rails70 --env staging --name webserver
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID_TOOLS }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY_TOOLS }}
          AWS_REGION: us-west-2
          DOCKER_BUILDKIT: 1
```
When you commit to `develop`, GitHub will trigger the script and deploy your code to staging account.

## Production Deployment

Create a file `./github/workflows/prod.yml`
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

      - name: prod deployment
        run: copilot svc deploy --app rails70 --env production --name webserver
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID_TOOLS }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY_TOOLS }}
          AWS_REGION: us-west-2
          DOCKER_BUILDKIT: 1
```
When you commit to `main`, GitHub will trigger the script and deploy your code to prod account.

## Deployment on GitHub Actions
<img width="1508" alt="Screen Shot 2023-05-18 at 7 11 47 pm" src="https://github.com/build-with-aws-copilot/rails_docs/assets/129698988/f27cda12-3aa6-4998-a8f5-df5986c4e3a5">

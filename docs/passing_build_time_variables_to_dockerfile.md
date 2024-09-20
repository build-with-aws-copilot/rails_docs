If you need to pass build time variables to Dockerfile, you can do the following:

## In manifest.yml
In manifest.yml, add an `args` section under `image`:

```
image:
  # Docker build arguments. For additional overrides: https://aws.github.io/copilot-cli/docs/manifest/lb-web-service/#image-build
  # Port exposed through your container to route traffic to it.
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
```
These variables need to be provided by the environment in which the manifest is run. For example, we build docker image in CICD, so our CICD needs to provide these values.

In Semaphore, we can provide these values in secrets:

![Screenshot 2024-09-21 at 08 46 09](https://github.com/user-attachments/assets/7256a012-e932-47ff-aab3-bf2ae61b6b9c)

## In Dockerfile
Once the variables are added to manifest, they will be passed on to Docker when building image. We can use these variables in Dockerfile:

```
ARG AWS_S3_ACCESS_KEY_ID=$AWS_S3_ACCESS_KEY_ID
ARG AWS_S3_SECRET_ACCESS_KEY=$AWS_S3_SECRET_ACCESS_KEY
ARG S3_BUCKET_NAME=$S3_BUCKET_NAME
ARG AWS_S3_REGION=$AWS_S3_REGION
ARG CLOUDFRONT_ENDPOINT=$CLOUDFRONT_ENDPOINT
ARG APPLE_PEM_PATH=$APPLE_PEM_PATH
ARG APPLE_TEAM_ID=$APPLE_TEAM_ID
ARG APPLE_KEY_ID=$APPLE_KEY_ID
ARG REDIS_HOST=$REDIS_HOST
ARG REDIS_PORT=$REDIS_PORT
ARG AWS_REGION=$AWS_REGION
ARG AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
ARG AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
ARG AIRBRAKE_PROJECT_ID=$AIRBRAKE_PROJECT_ID
ARG AIRBRAKE_PROJECT_KEY=$AIRBRAKE_PROJECT_KEY
ARG T_AUTHENTICATION_SECRET=$T_AUTHENTICATION_SECRET
```
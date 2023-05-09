!!! info
    Here I'm going to create a Redis cluster with AWS UI. But of course you can use IaC tools such as Terraform, CDK...etc.

Create a Redis cluster. It must be placed in the same VPC.

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
What is the value of secret REDIS_HOST in environment prod?
What is the value of secret REDIS_HOST in environment staging?
```
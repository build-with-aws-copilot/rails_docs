We will create 3 accounts on AWS

1. Application account: our Rails app will be initialised in this account. Docker images will be pushed to this account (Elastic Container Registry)
2. Staging account: for Rails staging environment
3. Production account: for Rails production environment

## Create Application Account

After application account is created, create an IAM user with Administrator Access. Console access can be disabled.

## Create Staging Account

After staging account is created, create an IAM user with Administrator Access. Console access can be disabled.

## Create Production Account

After production account is created, create an IAM user with Administrator Access. Console access can be disabled.

<figure markdown>
  ![IAM user](https://user-images.githubusercontent.com/166879/235662104-e5de62a7-15cf-4897-b272-9cde14408f90.png)
  <figcaption>An exmaple of IAM user</figcaption>
</figure>

!!! warning
    It is not a good practise to give IAM user administrator access; we should give it the least priviledge. But for now we will let Copilot do its thing, for example creating Docker repository and IAM roles. Once everything has setup, we can delete the IAM users in staging and production. As for the IAM user in application account, we can use [Access Analyzer](https://www.youtube.com/watch?v=SJQSWeogUWs) to give it the least priviledge.

## Create AWS Named Profiles

Now you've created 3 IAM users. Add their credentials to `~/.aws/credentials` and `~/.aws/config` on your local


~/.aws/credentials
```
[copilot.application]
aws_access_key_id=AKIUJFHFJDFKFUBHUE88
aws_secret_access_key=+Wv3oDIUJSDHCNVBOLQKDIFUDIIIUDIKBK+vUmqc

[copilot.staging]
aws_access_key_id=AKLKMDJCHUUDHIDIUE67
aws_secret_access_key=eUdT8KIMDJFIOUJSHEHWEJQAZSDSEJFJFMFbyUrl

[copilot.prod]
aws_access_key_id=AKIJUYHNBGTRFVBVC904
aws_secret_access_key=oWzVIUEURHFDKDJFKUHXBCNVJFJIIFKDFIUxxfqj
```

~/.aws/config
```
[profile copilot.application]
region=us-west-2

[profile copilot.staging]
region=us-west-2

[profile copilot.prod]
region=us-west-2
```

## Create Copilot App
Earlier this page, we mentioned that our app will be initialized in application account. Let's initialize the app now.

Run the following commands in your local
```
export AWS_PROFILE=copilot.application
export AWS_REGION=us-west-2
```
Initialize the app. We will name our app `rails70`
```
copilot app init rails70
```

## Create Copilot Environments

## Create Copilot Services
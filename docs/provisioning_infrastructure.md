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
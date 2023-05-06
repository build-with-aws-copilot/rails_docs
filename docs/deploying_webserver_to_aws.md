Let's recap what we have done so far. We have

1. created Copilot app in AWS application account
2. created production environment in AWS production account
3. created staging environment in AWS staging account
4. created webserver and worker manifests

We're ready to deploy our Rails app to AWS.

## Configure Rails

Add following to `application.rb`. AWS Load Balancer will generate a domain, but we don't know what it is yet. So we will allow any domain for now.

```
config.hosts = [
  /.*/,
]
```

## Add Secrets to AWS
```
copilot secret init
What would you like to name this secret? [? for help] RAILS_MASTER_KEY
What is the value of secret RAILS_MASTER_KEY in environment prod? # paste the master key from config/master.key to here
```

After secrets are added, add following to webserver manifests so the containers can use these environment variables.

webserver/manifest
```
secrets:
    RAILS_MASTER_KEY: /copilot/${COPILOT_APPLICATION_NAME}/${COPILOT_ENVIRONMENT_NAME}/secrets/RAILS_MASTER_KEY
```

## Deploy Webserver

Now we're ready to deploy webserver to AWS. Run
```
copilot svc deploy --app rails70 --env staging --name webserver # push to staging
copilot svc deploy --app rails70 --env prod --name webserver # push to production
```

!!! info
    You need to have Docker running on your local. Copilot will build a docker image, deploy it to ECR and run the image in ECS.

!!! info
    We only need to build the image manually for the first time. Later we will move the building/deployment process to Github Actions.

<figure markdown>
  ![Successful Deployment](https://user-images.githubusercontent.com/129698988/236616504-0f657bef-a746-4fe7-b5b9-6ef00998a445.png)
  <figcaption>A Successful Deployment</figcaption>
</figure>

You will see an URL at the end of the deployment. That's the URL of the Load Balancer which Copilot has provisioned for us. Open up the URL:
<img width="1512" alt="Screen Shot 2023-05-06 at 7 46 15 pm" src="https://user-images.githubusercontent.com/129698988/236616719-b3f60cbd-b4c8-4135-b869-0b9b4ecc26b3.png">

Congratulations, you have deployed your first container to AWS 🎉

!!! info
    When Copilot is deploying the services, you can check the log on AWS to see if there's any error
    <img width="1506" alt="Screen Shot 2023-05-06 at 7 39 51 pm" src="https://user-images.githubusercontent.com/129698988/236616846-92d927ed-8751-4834-bec0-272f52a6c560.png">

    If there is, don't waste time waiting for Copilot to return error. Go to CloudFormation, delete the task, fix the error and redeploy (`copilot svc deploy`). 
    <img width="1506" alt="Screen Shot 2023-05-06 at 7 40 15 pm" src="https://user-images.githubusercontent.com/129698988/236616875-9b7d396b-919b-418c-9411-7195713c1602.png">

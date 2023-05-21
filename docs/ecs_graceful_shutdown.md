When deploying to ECS, ECS will receive `SIGTERM`, it is a signal asking containers to get ready to be terminated. Default wait time is 30 seconds. After 30 seconds, the containers will receive `SIGKILL` and the containers will be stopped. If our Sidekiq takes 40 seconds on average to process jobs, we should extend the wait time a bit longer so Sidekiq can finish processing jobs without being interrupted.

The default wait time `stopTimeout` can be overridden in Copilot. We will extend wait time to 50 seconds:

Run
```
copilot svc override
```

This will generate the following files:
```
copilot/webserver/overrides/cfn.patches.yml
copilot/worker/overrides/cfn.patches.yml
```

Add the following command to the yml
```
- op: add
  path: /Resources/TaskDefinition/Properties/ContainerDefinitions/0/StopTimeout
  value: 50
```
After next deployment, we can go to ECS tasks and verify that `stopTimeout` has been added:
<img width="1509" alt="Screen Shot 2023-05-14 at 8 55 52 pm" src="https://github.com/build-with-aws-copilot/rails_docs/assets/129698988/2b5023c6-5990-45fa-9b6e-46b06de657af">

We will also pass `-t` parameter to worker containers. `-t` is the number of seconds Sidekiq is expected to finish processing jobs. If average job processing time is 40 seconds, we will pass `-t 40`:

Dockerfile.sidekiq
```
CMD ["bundle", "exec", "sidekiq", "-t", "40", "-C", "/rails/config/sidekiq_worker.yml"]
```
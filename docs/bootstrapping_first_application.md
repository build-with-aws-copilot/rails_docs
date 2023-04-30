## Create a new Rails app
Run
```
rails new rails70
```
Add `dockerfile-rails` to `Gemfile`
```
gem "dockerfile-rails", "~> 1.2.5", :group => :development
```
We will run docker on x86-64 linux on ECS, so we need to add platform support for x86-64
```
bundle lock --add-platform x86_64-linux
```

Run `bundle install`

## Generate Dockerfile

`dockerfile-rails` is a Rails generator to produce Dockerfiles.

Run
```
bin/rails generate dockerfile
```
to generate a Dockerfile:
<script src="https://gist.github.com/build-with-aws-copilot/e3e4c10c228b7319402ea228b57528d0.js"></script>



!!! info
    This Dockerfile uses `--link`, so the syntax has to be at least 1.4 (line 1).

!!! info
    `dockerfile-rails` supports many different parameters, I encourage you to check out [its demo](https://github.com/rubys/dockerfile-rails/blob/main/DEMO.md) and the [supported parameters](https://github.com/rubys/dockerfile-rails/blob/main/lib/generators/dockerfile_generator.rb).

## Test Dockerfile on local
!!! info
    You need to have Docker installed on your local

Add a default route to `routes.rb`
```
root "rails/welcome#index"
```
Run the following command
```
docker buildx build . -t rails70
docker run -p 3000:3000 -e RAILS_MASTER_KEY=$(cat config/master.key) --rm rails70
```

Open up your browser and go to `localhost:3000`
<img width="1506" alt="Screen Shot 2023-04-30 at 7 41 48 pm" src="https://user-images.githubusercontent.com/129698988/235346500-0bd6b718-10de-49c0-93db-976ac60d0c21.png">
Contgratulations, you have successfully running Rails container on your local. Next we will deploy the same image to AWS.

## Deploy to AWS
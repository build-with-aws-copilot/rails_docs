!!! info
    Here I'm going to create Postgres with AWS UI. But of course you can use IaC tools such as Terraform, CDK...etc.

## Create Postgres

Create a Postgres in staging and production accounts.
<img width="1507" alt="Screen Shot 2023-05-15 at 9 48 12 pm" src="https://github.com/build-with-aws-copilot/rails_docs/assets/129698988/b516571c-0262-4cb4-aca9-223ca774dcd0">

## Add Environment Variables
```
copilot secret init
What would you like to name this secret? [? for help] RDS_HOSTNAME
What is the value of secret REDIS_HOST in environment prod?
What is the value of secret REDIS_HOST in environment staging?

copilot secret init
What would you like to name this secret? [? for help] RDS_PORT
What is the value of secret REDIS_HOST in environment prod?
What is the value of secret REDIS_HOST in environment staging?

copilot secret init
What would you like to name this secret? [? for help] RDS_DB_NAME
What is the value of secret REDIS_HOST in environment prod?
What is the value of secret REDIS_HOST in environment staging?

copilot secret init
What would you like to name this secret? [? for help] RDS_USERNAME
What is the value of secret REDIS_HOST in environment prod?
What is the value of secret REDIS_HOST in environment staging?

copilot secret init
What would you like to name this secret? [? for help] RDS_PASSWORD
What is the value of secret REDIS_HOST in environment prod?
What is the value of secret REDIS_HOST in environment staging?
```

## Security Group
Add ECS's security group to allowed inbound traffic, so ECS can talk to Postgres.
<img width="1510" alt="Screen Shot 2023-05-15 at 10 38 28 pm" src="https://github.com/build-with-aws-copilot/rails_docs/assets/129698988/ced3a424-9c8c-4e0c-8d4c-b161e9debdad">

## Update Database Config
Update `config/database.yml`

```
# SQLite. Versions 3.8.0 and up are supported.
#   gem install sqlite3
#
#   Ensure the SQLite 3 gem is defined in your Gemfile
#   gem "sqlite3"
#
default: &default
  adapter: postgresql
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  timeout: 5000

development:
  <<: *default
  database: rails70_development

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: rails70_test

production:
  <<: *default
  database: <%= ENV['RDS_DB_NAME'] %>
  username: <%= ENV['RDS_USERNAME'] %>
  password: <%= ENV['RDS_PASSWORD'] %>
  host: <%= ENV['RDS_HOSTNAME'] %>
  port: <%= ENV['RDS_PORT'] %>
```
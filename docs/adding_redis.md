!!! info
    Here I'm going to create a Redis server with AWS UI. But of course you can use IaC tools such as Terraform, CDK...etc.

Create a Redis server. It must be placed in the same VPC.

Cluster mode should be disabled

<img width="1510" alt="Screen Shot 2023-05-14 at 10 42 58 pm" src="https://github.com/build-with-aws-copilot/rails_docs/assets/129698988/ee8adfc6-b8f1-4f91-a137-d3aa39f2dd7c">

## Security Group

Allow traffic from ECS' security group

<img width="1510" alt="Screen Shot 2023-05-14 at 10 35 45 pm" src="https://github.com/build-with-aws-copilot/rails_docs/assets/129698988/154b3f36-ddc6-4f7f-b676-3ed8d41c9b2b">

<img width="1510" alt="Screen Shot 2023-05-14 at 10 36 31 pm" src="https://github.com/build-with-aws-copilot/rails_docs/assets/129698988/02dc1649-3a7b-4b93-a587-6b04ac07db98">
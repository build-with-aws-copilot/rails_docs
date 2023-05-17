Request a new certificate on ACM
<img width="1508" alt="Screen Shot 2023-05-15 at 10 44 43 pm" src="https://github.com/build-with-aws-copilot/rails_docs/assets/129698988/28b2b793-6da2-4a06-86b8-051417484970">

Add CNAME to DNS provider
<img width="841" alt="Screen Shot 2023-05-17 at 9 30 56 pm" src="https://github.com/build-with-aws-copilot/rails_docs/assets/129698988/44692075-987c-4dff-b158-90893f793833">

Update security group of ALB (allow 443)

Add listener 443 to ALB, attach the SSL/TLS certificate, and forward it to target group
<img width="1508" alt="Screen Shot 2023-05-15 at 10 45 09 pm" src="https://github.com/build-with-aws-copilot/rails_docs/assets/129698988/32993384-8bdb-4535-8846-9a9c68a8c29b">

<img width="1508" alt="Screen Shot 2023-05-15 at 10 45 58 pm" src="https://github.com/build-with-aws-copilot/rails_docs/assets/129698988/9ec01a58-7831-4406-9c02-d6672d3453dc">

Update port 80 rules (2 rules), redirect them to 443

<img width="1508" alt="Screen Shot 2023-05-15 at 10 45 34 pm" src="https://github.com/build-with-aws-copilot/rails_docs/assets/129698988/ee3607f8-8ccc-4e4d-9bd4-e9cfda2e371f">
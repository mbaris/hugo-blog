---
title: "Infrastructure as Code"
date: 2021-04-25T00:02:25+02:00
draft: false
tags: ["devops", "infrastructure"]
description: "Infrastructure as code is the practice of managing and provisioning resources through definition files"
---

_Infrastructure as Code_ is the practice of managing and provisioning resources through version controlled definition files.

Things to note beforehand
- Cloud providers like AWS, Azure and GCP allow us to provision new resources in mere minutes or even seconds.
- Multiple deployments to production every day is not an exceptional thing anymore
- We want to use a similar infrastructure for multiple environments(dev,staging,prod etc.) and regions
- Infrastructure on cloud does not only mean hosting applications on a virtual machine
  It also includes services we can use like data stores, queues/topics, network components, access rights/permission models and much more
- Utilizing these resources efficiently will help us move forward much faster

## What problem does it solve?

While cloud computing provides us an easy and fast way to get access to new resources, we still have to create, keep an inventory of and decommission them when necessary.
In a traditional on-premise setup, we would document and apply all the steps manually when we need to install and run services on these servers.
However, this approach does not scale well if we want to get the full benefits of using cloud providers.
Because development and maintenance of an application on cloud can easily require hundreds of resources across different accounts. 
So we definitely need some kind of proper automation.

## How does it work?
* Create configuration files or scripts defining required resources, and their relations with each other. 
  These can be written in bash or domain specific languages like [Terraform/HCL](https://www.terraform.io) or [Cloudformation Templates](https://aws.amazon.com/cloudformation/resources/templates/)
* Store them in a version control system like git. This can be done in two ways and both has their advantages/disadvantages.
  - Along with the service using these resources
  - In a centralized repository which contains all resources used by the company
* Integrate these tools/scripts to our CI/CD pipeline
* Applying the same configuration should always give the same result regardless of the state of the infrastructure when we started. 
This is a broader concept called [Idempotency](https://en.wikipedia.org/wiki/Idempotence) and it is a really important in IaC
  

### Imperative vs Declarative
It is important to note that Infrastructure as Code can be implemented with both approaches effectively. 

Using an imperative approach to create an AWS Lambda function would look like below
``` bash
aws lambda create-function \
    --function-name my-function \
    --runtime nodejs12.x \
    --zip-file fileb://my-function.zip \
    --handler exports.test \
    --role arn:aws:iam::123456789012:role/service-role/iam_for_lambda
```

While with a declarative approach, it would look like this.

``` HCL
resource "aws_lambda_function" "my-function" {
  filename      = "lambda_function_payload.zip"
  function_name = "lambda_function_name"
  role          = aws_iam_role.iam_for_lambda.arn
  handler       = "exports.test"
  source_code_hash = filebase64sha256("my-function.zip")
  runtime = "nodejs12.x"
}
```

## Why should we use it?
### Consistency
Even if we are really careful, any manual process involving humans is error prone. 
Getting the same result every time you need a new environment prevents so many problems like having different configurations 
for test and prod environments or wasting time on errors due to misconfigurations 
### Speed
If we properly integrate infrastructure as code to our build pipelines, our resource provisioning can be completely automated.
Not spending time on creating resources means more time efficient development and deployment processes. 
### Accountability
We already have best practices regarding source control. With peer reviews and pull requests, any change in the infrastructure is visible and obvious
We can track down what changed with which deployment and also revert to a previous version easily if we need to.
### Scalability
We might need to deploy our services to completely new regions to expand or new environments for reasons like development, tests or demos.
With infrastructure as code, we are not bound to number of people who can create infrastructure and manage them for our team.

## Conclusion
My first experience with IaC was an in-house tool made by engineers in the company, and it saved so much time and effort when we were developing Opsgenie. 
I can't imagine how difficult and stressful deployments would be if we were manually creating database tables, iam policies and queues every other day.

More recently, I have been using Terraform at work and what you can build with it is impressive. Especially since there are so many modules available to use.
I will take a look at [creating infrastructure with](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/guides/custom-service-endpoints#localstack) [localstack](https://github.com/localstack/localstack) on a personal project soon

There is obviously a learning curve for everyone involved, but it is certainly worth the effort.  
In my opinion, this is one thing every software company should spend some time on.

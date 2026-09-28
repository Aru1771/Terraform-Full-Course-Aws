Topic: for_each with Map
=========================

1. What is a Map?

A map contains key-value pairs.

For example:

    environments = {
      dev   = "t3.micro"
      stage = "t3.small"
      prod  = "t3.medium"
    }

Use for_each with the Map
----------------------------

We can give the map directly to for_each:

    for_each = var.environments
Terraform will process each key-value pair.

For our example:

    dev   → t3.micro
    stage → t3.small
    prod  → t3.medium

Two Important Values
---------------------

When using for_each with a map, we have:

each.key

The key.

    dev
    stage
    prod

each.value

The value.

    t3.micro
    t3.small
    t3.medium

So:

    Map
     ↓
    each.key   → key
    each.value → value


variables.tf
    
    variable "instance_types" {
      type = map(string)
    }

    variable "ami_id" {
      type = string
    }

terraform.tfvars

    instance_types = {
      dev   = "t3.micro"
      stage = "t3.small"
      prod  = "t3.medium"
    }

  
    ami_id = "your-ami-id"


main.tf


    resource "aws_instance" "app" {
      for_each = var.instance_types
    
      ami           = var.ami_id
      instance_type = each.value
    
      tags = {
        Name = "app-${each.key}"
      }
    }


What happens?

Terraform sees:

    dev   → t3.micro
    stage → t3.small
    prod  → t3.medium

First resource

    each.key   = "dev"
    each.value = "t3.micro"

Terraform creates:

    Name = "app-dev"
    Type = "t3.micro"

Second resource

    each.key   = "stage"
    each.value = "t3.small"

Creates:

    Name = "app-stage"
    Type = "t3.small"

Third resource

    each.key   = "prod"
    each.value = "t3.medium"

Creates:

    Name = "app-prod"
    Type = "t3.medium"


Resource Addresses

With count, you learned:

    aws_instance.app[0]
    aws_instance.app[1]
    aws_instance.app[2]

With for_each + map, Terraform uses the keys:

    aws_instance.app["dev"]
    aws_instance.app["stage"]
    aws_instance.app["prod"]

This is an important difference.


Topic 4 Practice
-----------------
Create a map containing three environments and their EC2 instance types:

dev   → t3.micro
stage → t3.small
prod  → t3.medium

Use:

for_each = var.instance_types

Then use:

each.key

for the EC2 name and:

each.value

for the EC2 instance type.

Run:

terraform plan

Your expected resource addresses should look like:

aws_instance.app["dev"]
aws_instance.app["stage"]
aws_instance.app["prod"]

Focus only on these two concepts for Topic 4:

each.key   → map key
each.value → map value

Send your terraform plan output when you're done, and we'll do the code review + interview questions.

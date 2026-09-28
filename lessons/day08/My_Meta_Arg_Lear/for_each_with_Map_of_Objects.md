Topic: for_each with Map of Objects
====================================


First, what is a Map of Objects?

You already learned map of objects in the Terraform Data Types topic.

A map has:

    key → value

When the value is an object, we get:

    key → object

Example:

    applications = {
      payment = {
        environment = "prod"
        port        = 8080
      }
    
      user = {
        environment = "dev"
        port        = 8081
      }
    }

Think of it like:

    Map
    │
    ├── payment → Object
    │              ├── environment
    │              └── port
    │
    └── user → Object
                 ├── environment
                 └── port


Why use for_each?
-------------------

We want Terraform to create one resource for each object.

So:

    for_each = var.applications

Terraform sees:

    payment → object
    user    → object

and creates:

    aws_instance.app["payment"]
    aws_instance.app["user"]

each.key and each.value
-------------------------
This is the most important part.

For:

    payment → {
      environment = "prod"
      port        = 8080
    }

Terraform gives us:

each.key

which is:

    payment

And:

each.value

is the entire object:

    {
      environment = "prod"
      port        = 8080
    }

Therefore, to get something inside that object:

    each.value.environment

or:

    each.value.port



Simple AWS Example
-------------------

variables.tf

    variable "applications" {
      type = map(object({
        environment = string
        port        = number
      }))
    }

    variable "ami_id" {
      type = string
    }


terraform.tfvars

    applications = {
      payment = {
        environment = "prod"
        port        = 8080
      }
    
      user = {
        environment = "dev"
        port        = 8081
      }
    }
    
    ami_id = "your-ami-id"

main.tf

    resource "aws_instance" "app" {
      for_each = var.applications
    
      ami           = var.ami_id
      instance_type = "t3.micro"
    
      tags = {
        Name        = each.key
        Environment = each.value.environment
        Port        = each.value.port
      }
    }


Understand One Resource
------------------------
For:

    payment = {
      environment = "prod"
      port        = 8080
    }

Terraform gets:

    each.key
       ↓
    "payment"

and:

    each.value
       ↓
    {
      environment = "prod"
      port        = 8080
    }

Then:

    each.value.environment

gives:

    prod

and:

    each.value.port

gives:

    8080

🧠 Easy Memory
----------------

For map of objects:

    for_each
       ↓
      Map
       ↓
    ┌───────────────┐
    │ key → object  │
    └───────────────┘
       ↓       ↓
    each.key  each.value
                 ↓
              object
                 ↓
          attribute

Example:

    each.value.environment

means:

Get the environment attribute from the current object's value.

Practice
--------

Create two applications:

payment
user

Each should have:

environment
port

Use:

type = map(object({
  environment = string
  port        = number
}))

Then create EC2 instances using:

for_each = var.applications

Use:

each.key

for the application name.

Use:

each.value.environment

for the environment.

Use:

each.value.port

for the port tag.

Your Terraform resources should have addresses similar to:

aws_instance.app["payment"]
aws_instance.app["user"]
🎯 Remember the difference

Topic 4 — Map:

each.key
each.value

where each.value was a simple value:

dev → t3.micro

Topic 5 — Map of Objects:

each.key
each.value.attribute

Example:

payment → {
  environment = "prod"
  port = 8080
}

So:

each.key

→ payment

each.value.environment

→ prod

each.value.port

→ 8080

That's the core of Topic 5.

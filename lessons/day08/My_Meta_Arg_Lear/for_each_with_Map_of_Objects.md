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

We want Terraform to create one resource for each application.

So:

for_each = var.applications

Terraform sees:

payment → object
user    → object

and creates:

aws_instance.app["payment"]
aws_instance.app["user"]

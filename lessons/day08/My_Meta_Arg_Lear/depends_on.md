depends_on
==========

What is depends_on?
-------------------

depends_on tells Terraform:

    "Create this resource only after that resource has been created."

It creates an explicit dependency between resources.

Simple example

Suppose we have:

    Security Group
          ↓
    EC2 Instance

We want Terraform to create the Security Group first and then the EC2 instance.

We can use:

    depends_on = [
      aws_security_group.app
    ]

Why do we need depends_on?
--------------------------

Terraform is normally smart enough to understand dependencies.

For example:

    resource "aws_instance" "app" {
      security_groups = [aws_security_group.app.name]
    }

Terraform sees:

    aws_instance.app
            ↓
    references
            ↓
    aws_security_group.app

So Terraform automatically understands:

    Security Group → EC2

This is called an implicit dependency.

You don't need depends_on here.

Explicit Dependency
-------------------

Sometimes the dependency isn't visible from the resource arguments.

Example:

    IAM Role
       ↓
    Application

Suppose your application needs the IAM role to exist first, but your Terraform configuration doesn't directly reference the role.

You can explicitly tell Terraform:

    depends_on = [
      aws_iam_role.app
    ]

Now Terraform knows:

    aws_iam_role.app
           ↓
       must exist
           ↓
    aws_instance.app

Simple Example
---------------

    resource "aws_iam_role" "app" {
      name = "app-role"
    
      assume_role_policy = jsonencode({
        Version = "2012-10-17"
    
        Statement = [{
          Effect = "Allow"
          Principal = {
            Service = "ec2.amazonaws.com"
          }
          Action = "sts:AssumeRole"
        }]
      })
    }

Then:

    resource "aws_instance" "app" {
      ami           = var.ami_id
      instance_type = "t3.micro"
    
      depends_on = [
        aws_iam_role.app
      ]
    }

Terraform understands:

    IAM Role
       ↓
    depends_on
       ↓
    EC2 Instance

So Terraform creates the IAM role first.

depends_on Syntax
------------------

The syntax is:

    depends_on = [
      resource_type.resource_name
    ]

Example:

    depends_on = [
      aws_iam_role.app
    ]

For multiple dependencies:

    depends_on = [
      aws_iam_role.app,
      aws_security_group.app
    ]

Then Terraform waits for both.

    IAM Role ─────────┐
                      ↓
                 EC2 Instance
                      ↑
    Security Group ───┘


Implicit vs Explicit Dependency
-------------------------------

This is very important for interviews.

Implicit dependency

Terraform automatically understands it because one resource references another.

    resource "aws_instance" "app" {
      security_group_id = aws_security_group.app.id
    }

Terraform sees:

    EC2 → references → Security Group

So it knows the dependency.

Explicit dependency

We manually tell Terraform:

    depends_on = [
      aws_iam_role.app
    ]

Terraform now knows:

    IAM Role → EC2

even if there isn't a direct reference.

depends_on = "Wait for this resource first."
-------------------------------------------------

    Resource A
        ↓
    depends_on
        ↓
    Resource B

Terraform creates:

    A → B

🎯 Topic 7 Practice
----------------------
Let's create a simple AWS scenario.

Resource 1 — Security Group
----------------------------

    resource "aws_security_group" "app" {
      name = "app-sg"
    }
    
Resource 2 — EC2
-----------------

    resource "aws_instance" "app" {
      ami           = var.ami_id
      instance_type = "t3.micro"
    
      depends_on = [
        aws_security_group.app
      ]
    
      tags = {
        Name = "app-server"
      }
    }

Your task

Understand what Terraform should do:

    aws_security_group.app
            ↓
          first
            ↓
    aws_instance.app
           ↓
         second

Then run:

    terraform plan

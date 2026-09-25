What is count?
--------------

count is a Terraform meta-argument that allows you to create multiple instances of the same resource.

Without count:

    resource "aws_instance" "app" {
      ami           = "ami-xxxxxxxx"
      instance_type = "t3.micro"
    }

Terraform creates one EC2 instance.

With:

    resource "aws_instance" "app" {
      count = 3
    
      ami           = "ami-xxxxxxxx"
      instance_type = "t3.micro"
    }

Terraform creates:

    aws_instance.app[0]
    aws_instance.app[1]
    aws_instance.app[2]


Why do we use count?

Imagine you need:

    3 Dev servers

Without count, you might write:

      resource "aws_instance" "dev1" {
        ...
      }
      
      resource "aws_instance" "dev2" {
        ...
      }
      
      resource "aws_instance" "dev3" {
        ...
      }


That's repetitive.

With count:

    resource "aws_instance" "dev" {
      count = 3
    
      ami           = "ami-xxxxxxxx"
      instance_type = "t3.micro"
    }

Terraform creates all three.

Main idea:

    count = create N instances of the same resource.

Simple Example Without AWS
---------------------------
Let's first understand the concept using a simple resource.

    resource "local_file" "app" {
      count    = 3
      filename = "app-${count.index}.txt"
      content  = "Hello Terraform"
    }

Terraform creates:

    app-0.txt
    app-1.txt
    app-2.txt

Notice something important:

    count = 3

creates three instances.

The count value itself tells Terraform how many.


AWS DevOps Example
-----------------
Suppose your DevOps team needs 3 EC2 instances for application servers.

    resource "aws_instance" "app" {
      count = 3
    
      ami           = var.ami_id
      instance_type = "t3.micro"
    
      tags = {
        Name = "app-server"
      }
    }

Terraform understands this as:

    aws_instance.app[0]
    aws_instance.app[1]
    aws_instance.app[2]

Important Point

count belongs inside the resource block:

    resource "aws_instance" "app" {
    
      count = 3
    
      ...
    }

It is a meta-argument, not an AWS-specific argument.

That's why many Terraform resources can use it.

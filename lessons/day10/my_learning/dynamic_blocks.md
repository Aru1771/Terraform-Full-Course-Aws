Topic 5 — Dynamic Blocks
==========================

Let's start with the basics and keep it practical.

1. What is a Dynamic Block?
   
        A dynamic block allows Terraform to create multiple nested blocks automatically based on a collection such as a list, set, or map.

Think of it as:

    for_each for nested blocks

For example, normally you might have to write:

    resource "aws_security_group" "app" {
      name = "app-sg"
    
      ingress {
        from_port   = 80
        to_port     = 80
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }
    
      ingress {
        from_port   = 443
        to_port     = 443
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
      }
    }

If you have many ingress rules, repeating the block becomes inconvenient.

With a dynamic block, we can generate those ingress blocks from a variable.

2. Basic Syntax

        dynamic "ingress" {
          for_each = var.ingress_rules
        
          content {
            from_port   = ingress.value.from_port
            to_port     = ingress.value.to_port
            protocol     = ingress.value.protocol
            cidr_blocks = ingress.value.cidr_blocks
          }
        }

The important structure is:
    
    dynamic "BLOCK_NAME"
           ↓
        for_each
           ↓
        content
           ↓
     values from each item

3. Simple Example

        Variable:
        variable "ingress_rules" {
          type = list(object({
            port     = number
            protocol = string
            cidr     = string
          }))
        }

Values:

    ingress_rules = [
      {
        port     = 80
        protocol = "tcp"
        cidr     = "0.0.0.0/0"
      },
      {
        port     = 443
        protocol = "tcp"
        cidr     = "0.0.0.0/0"
      }
    ]

Dynamic block:

    dynamic "ingress" {
      for_each = var.ingress_rules
    
      content {
        from_port   = ingress.value.port
        to_port     = ingress.value.port
        protocol     = ingress.value.protocol
        cidr_blocks = [ingress.value.cidr]
      }
    }

Terraform effectively generates:

    ingress {
      from_port   = 80
      to_port     = 80
      protocol     = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
    
    ingress {
      from_port   = 443
      to_port     = 443
      protocol     = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }

4. Important Difference
You already know:

for_each

      creates multiple resources:

resource "aws_instance" "app" {
  for_each = var.applications
}

Dynamic blocks create multiple nested blocks inside one resource:

    resource "aws_security_group" "app" {
    
      dynamic "ingress" {
        for_each = var.ingress_rules
    
        content {
          ...
        }
      }
    }

Easy way to remember:

    for_each → multiple resources
    dynamic → multiple nested blocks

5. One More Important Point

Inside the dynamic block:

    dynamic "ingress" {
      for_each = var.ingress_rules
    
      content {
        from_port = ingress.value.port
      }
    }

Here:

    ingress        → dynamic block name
    ingress.value  → current item

Similar to how you previously used:
each.value

with for_each.


Suppose your application needs:
    
    HTTP   → 80
    HTTPS  → 443
    SSH    → 22

Instead of writing three ingress blocks manually, we'll store the rules in a list(object):

    variable "ingress_rules" {
      type = list(object({
        port     = number
        protocol = string
        cidr     = string
      }))
    }

Values:

    ingress_rules = [
      {
        port     = 80
        protocol = "tcp"
        cidr     = "0.0.0.0/0"
      },
      {
        port     = 443
        protocol = "tcp"
        cidr     = "0.0.0.0/0"
      },
      {
        port     = 22
        protocol = "tcp"
        cidr     = "10.0.0.0/16"
      }
    ]

Then:
    
    resource "aws_security_group" "app" {
      name = "application-sg"
    
      dynamic "ingress" {
        for_each = var.ingress_rules
    
        content {
          from_port   = ingress.value.port
          to_port     = ingress.value.port
          protocol     = ingress.value.protocol
          cidr_blocks = [ingress.value.cidr]
        }
      }
    }

Terraform creates three nested ingress blocks automatically.

Remember this pattern

    list(object)
         ↓
    dynamic
         ↓
    for_each
         ↓
    content
         ↓
    ingress.value


Task:

main.tf
         
    resource "aws_security_group" "my_sg" {
        name = "app_sg"
    
    
        dynamic "ingress" {
            for_each = var.ingress_rules
    
            content{
                from_port = ingress.value.from_port
                to_port = ingress.value.to_port
                protocol = ingress.value.protocol
                cidr_blocks = ingress.value.cidr_blocks
                
            }
        }
    
    
    }

var.tf

    variable "ingress_rules" {
        type = list(object({
            from_port = number
            to_port = number
            protocol = string
            cidr_blocks = list(string)
        }))
    }


.tfvars

    ingress_rules = [ 
    
        {
            from_port = 80
            to_port = 80
            protocol = "tcp"
            cidr_blocks = [ "0.0.0.0/0" ]
    
        },
    
        {
            from_port = 443
            to_port = 443
            protocol = "tcp"
            cidr_blocks = [ "0.0.0.0/0" ]
        }
    ]





SG and Ingress Rules for Multiple-ENV:
----------------------------------------

main.tf:


    resource "aws_security_group" "app" {
    
      for_each = var.environments
    
      name = "${each.key}-app-sg"
    
      dynamic "ingress" {
    
        for_each = each.value.ingress_rules
    
        content {
          from_port   = ingress.value.from_port
          to_port     = ingress.value.to_port
          protocol    = ingress.value.protocol
          cidr_blocks = ingress.value.cidr_blocks
        }
      }
    
      lifecycle {
        create_before_destroy = true
      }
    }


var.tf

    variable "environments" {
      type = map(object({
        ingress_rules = list(object({
          from_port   = number
          to_port     = number
          protocol    = string
          cidr_blocks = list(string)
        }))
      }))
    }

.tfvars


    environments = {
    
      dev = {
        ingress_rules = [
          {
            from_port   = 80
            to_port     = 80
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]
          },
          {
            from_port   = 443
            to_port     = 443
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]
          },
          {
            from_port   = 22
            to_port     = 22
            protocol    = "tcp"
            cidr_blocks = ["10.0.0.0/16"]
          }
        ]
      }
    
      stage = {
        ingress_rules = [
          {
            from_port   = 80
            to_port     = 80
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]
          },
          {
            from_port   = 443
            to_port     = 443
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]
          },
          {
            from_port   = 22
            to_port     = 22
            protocol    = "tcp"
            cidr_blocks = ["10.0.0.0/16"]
          },
          {
            from_port   = 8080
            to_port     = 8080
            protocol    = "tcp"
            cidr_blocks = ["10.0.0.0/16"]
          },
          {
            from_port   = 9090
            to_port     = 9090
            protocol    = "tcp"
            cidr_blocks = ["10.0.0.0/16"]
          }
        ]
      }
    
      prod = {
        ingress_rules = [
          {
            from_port   = 80
            to_port     = 80
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]
          },
          {
            from_port   = 443
            to_port     = 443
            protocol    = "tcp"
            cidr_blocks = ["0.0.0.0/0"]
          },
          {
            from_port   = 22
            to_port     = 22
            protocol    = "tcp"
            cidr_blocks = ["10.0.0.0/16"]
          }
        ]
      }
    }

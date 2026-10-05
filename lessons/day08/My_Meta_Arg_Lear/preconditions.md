Topic 2: Preconditions
========================


Now we're starting Topic 2 — Preconditions.

1. What is a precondition?
   
        A precondition is a rule that Terraform checks before creating or updating a resource.

Think of it as:

        "Before Terraform creates/updates this resource, check whether this condition is true."

If the condition is true:

    Precondition
         ↓
    TRUE ✅
         ↓
    Terraform continues

If the condition is false:

    Precondition
         ↓
    FALSE ❌
         ↓
    Terraform stops

2. Basic syntax
   
        A precondition is placed inside a lifecycle block:

          resource "aws_instance" "app" {
          
            ami           = var.ami_id
            instance_type = var.instance_type
          
            lifecycle {
          
              precondition {
                condition     = var.environment == "prod"
                error_message = "Environment must be prod."
              }
          
            }
          }
          
          The important part is:
          precondition {
            condition     = ...
            error_message = "..."
          }

3. Very simple example

Suppose you have:

    variable "environment" {
      default = "prod"
    }

And:

    lifecycle {
      precondition {
        condition     = var.environment == "prod"
        error_message = "This resource can only be created in production."
      }
    }

Terraform checks:

    var.environment
          ↓
    Is it "prod"?
          ↓
       ┌──┴──┐
      YES    NO
       ↓      ↓
    Continue  STOP ❌

4. What happens if the condition fails?

Suppose:

     environment = "dev"

but the condition says:

    condition = var.environment == "prod"

Then:

    "dev" == "prod"
           ↓
          false
           ↓
    Terraform stops ❌

Terraform shows the error_message:
Environment must be prod.

The resource won't be created/updated through that operation.


5. Practical DevOps example — Environment validation

Imagine your production EC2 must always use a production environment:

    resource "aws_instance" "app" {
    
      ami           = var.ami_id
      instance_type = var.instance_type
    
      lifecycle {
    
        precondition {
          condition     = var.environment == "prod"
          error_message = "Production EC2 must use environment = prod."
        }
    
      }
    
      tags = {
        Name        = "payment-server"
        Environment = var.environment
      }
    }

If someone runs:

    environment = "dev"

Terraform blocks the operation.
This protects you from an incorrect configuration.


6. Preconditions can validate resource attributes

It doesn't have to be only variables.

For example:

      resource "aws_instance" "app" {
      
        ami           = var.ami_id
        instance_type = var.instance_type
      
        lifecycle {
      
          precondition {
            condition     = var.instance_type == "t3.micro"
            error_message = "This application must use t3.micro."
          }
      
        }
      }

Terraform checks the condition before proceeding.


7. DevOps example — Application port

Let's connect this with your map(object) knowledge.

Suppose:

    variable "applications" {
      type = map(object({
        environment = string
        port        = number
      }))
    }
    
    And:
    resource "aws_instance" "app" {
    
      for_each = var.applications
    
      ami           = var.ami_id
      instance_type = var.instance_type
    
      lifecycle {
    
        precondition {
          condition     = each.value.port > 0 && each.value.port <= 65535
          error_message = "Application port must be between 1 and 65535."
        }
    
      }
    
      tags = {
        Name        = "${each.key}-server"
        Environment = each.value.environment
        Port        = each.value.port
      }
    }

Now imagine:
    
    payment → 8080 ✅
    user    → 8081 ✅
    order   → 70000 ❌

Terraform checks each application:

    payment
      ↓
    8080 valid
      ↓
    Continue ✅
    
    user
      ↓
    8081 valid
      ↓
    Continue ✅
    
    order
      ↓
    70000 invalid
      ↓
    Precondition fails ❌


This is a great real-world use case because you're using:

    map(object)
          +
    for_each
          +
    each.value
          +
    precondition


8. Precondition vs prevent_destroy

These can look similar, but they are completely different.

    prevent_destroy
    lifecycle {
      prevent_destroy = true
    }

Means:

    Don't allow Terraform to destroy this resource.

precondition

    lifecycle {
      precondition {
        condition     = ...
        error_message = "..."
      }
    }

Means:

    Before Terraform creates or updates the resource, verify this condition.

So:

    prevent_destroy
          ↓
    Protect destruction
    
    precondition
          ↓
    Validate before operation

9. Preconditions vs replace_triggered_by

You just learned replace_triggered_by.

replace_triggered_by

    Something changes
           ↓
    Replace this resource

precondition
    
    Before operation
           ↓
    Check condition
           ↓
    Pass → continue
    Fail → stop

🧠 Easy memory
Remember:

      replace_triggered_by
      → REPLACE
      
      prevent_destroy
      → PROTECT
      
      precondition
      → CHECK BEFORE


🎯 Topic 2 Practice Task

Let's combine today's topic with your previous knowledge.

Use your existing application structure:

    variable "applications" {
      type = map(object({
        environment = string
        port        = number
      }))
    
      default = {
        payment = {
          environment = "prod"
          port        = 8080
        }
    
        user = {
          environment = "dev"
          port        = 8081
        }
    
        order = {
          environment = "stage"
          port        = 8082
        }
      }
    }

Create:

    resource "aws_instance" "app" {
      for_each = var.applications
    
      # your AMI
      # your instance type
    
      lifecycle {
    
        precondition {
          # condition
          # error message
        }
      }
    
      tags = {
        # use each.key
        # use each.value
      }
    }

Requirement
Your precondition must make sure:
Application port must be between 1 and 65535.

Use:
each.value.port

Hint:

    condition = each.value.port >= 1 && each.value.port <= 65535

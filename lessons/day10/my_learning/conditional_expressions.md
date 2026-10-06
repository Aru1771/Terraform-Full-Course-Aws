What is a Conditional Expression?

A conditional expression allows Terraform to choose one value or another based on a condition.

The basic syntax is:

    condition ? true_value : false_value

Think:
 
             condition
                ↓
           ┌────┴────┐
          TRUE      FALSE
           ↓          ↓
       value-1     value-2

2. Very simple example

environment = "prod"

    instance_type = var.environment == "prod" ? "t3.medium" : "t3.micro"

Terraform checks:

    environment == "prod"?
              ↓
            TRUE
              ↓
       t3.medium

If:

    environment = "dev"

then:

    environment == "prod"?
              ↓
            FALSE
              ↓
        t3.micro

Easy memory

    condition ? YES : NO

3. Why is it useful?

Instead of writing separate logic, you can select a value dynamically.

For example:

    instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"

Means:

    If production → use t3.large; otherwise → use t3.micro.

This is very common in Terraform configurations.

4. DevOps example — Production vs Development
        
        variable "environment" {
          type = string
        }

        resource "aws_instance" "app" {
          ami = var.ami_id
        
          instance_type = var.environment == "prod" ? "t3.medium" : "t3.micro"
        
          tags = {
            Environment = var.environment
          }
        }
        
 If:

environment = "prod"

Terraform uses:
t3.medium

If:
environment = "dev"

Terraform uses:
t3.micro

5. Conditional expression with Boolean

You can also choose between true and false.

enable_monitoring = var.environment == "prod" ? true : false

But there is a simpler way:

    enable_monitoring = var.environment == "prod"

Because the comparison itself already returns:

    true / false

Still, understanding the conditional syntax is important.

6. Conditional expression with strings

        backup_type = var.environment == "prod" ? "daily" : "weekly"

Result:

    prod → daily
    dev  → weekly


7. Conditional expression with numbers

replicas = var.environment == "prod" ? 3 : 1

Result:

    prod → 3
    dev  → 1

Notice the values are numbers:
    
    3
    1

not:

    "3"
    "1"

This connects directly to the Number data type you learned earlier.


8. Conditional expression with your map(object)

Now let's combine today's topic with your previous complex data types.

You already have:

    applications = {
      payment = {
        environment = "prod"
        port        = 8080
        replicas    = 3
      }
    
      user = {
        environment = "dev"
        port        = 9090
        replicas    = 2
      }
    }

With:

    for_each = var.application

you can use:

    instance_type = each.value.environment == "prod" ? "t3.medium" : "t3.micro"

Terraform evaluates each application separately.

Payment

    payment
      ↓
    environment = prod
      ↓
    prod == prod
      ↓
    TRUE
      ↓
    t3.medium

User

    user
      ↓
    environment = dev
      ↓
    dev == prod
      ↓
    FALSE
      ↓
    t3.micro

So you get:

    payment → t3.medium
    user    → t3.micro

This is a great combination of:

    map(object)
    +
    for_each
    +
    each.value
    +
    conditional expression

9. Conditional expression with your previous Meta-Arguments

You can also combine it with lifecycle-related logic.

For example:

    tags = {
      Environment = each.value.environment
      ServerType  = each.value.environment == "prod" ? "production" : "non-production"
    }

Result:

    payment → ServerType = production
    user    → ServerType = non-production
    order   → ServerType = non-production


10. Nested conditional expressions

Terraform allows more than one condition:

    instance_type = var.environment == "prod" ? "t3.large" : var.environment == "stage" ? "t3.medium" : "t3.micro"

This means:

    prod  → t3.large
    stage → t3.medium
    other → t3.micro

But ⚠️ don't overuse nested conditional expressions because they become difficult to read.

For now, focus mainly on:

    condition ? true_value : false_value

🧠 Easy Memory

Remember:

    condition ? TRUE : FALSE

Example:

    var.environment == "prod" ? "t3.medium" : "t3.micro"

Read it as:

    If environment is prod, use t3.medium, otherwise use t3.micro.


🎯 Topic 4 Practice Task

Let's combine Data Types + Meta-Arguments + Conditional Expressions.

Use your existing:

    variable "application" {
      type = map(object({
        name        = string
        environment = string
        port        = number
        replicas    = number
    
        monitoring = object({
          enabled = bool
          path    = string
        })
      }))
    }

And:

    resource "aws_instance" "app" {
      for_each = var.application
    
      ami = var.ami_id
    
      # Your task:
      # Choose instance type based on environment
    }

Requirements
Production
If:
environment = "production"

use:
t3.medium

Development
If:
environment = "development"

use:
t3.micro

Staging
If:
environment = "staging"

use:
t3.small

Hint
You can start with:

    instance_type = each.value.environment == "production" ? "t3.medium" : ...

You'll need to complete the logic for development and staging.
🎯 Bonus Task
Add a tag:
DeploymentType = ...

Expected result:
payment → production
user    → development
order   → staging

But use a conditional expression to produce:
production environment → "PROD"
anything else          → "NON-PROD"

So:
payment → PROD
user    → NON-PROD
order   → NON-PROD

Your task
Write the complete aws_instance resource yourself.

Topic 3: Postconditions
=========================

1. What is a Postcondition?

        A postcondition is a rule Terraform checks after a resource has been created or updated.

Think of it like:

    Precondition = Check before
    Postcondition = Check after

Simple flow
Precondition
     ↓
Check before operation
     ↓
Create / Update resource
     ↓
Postcondition
     ↓
Check result

If the postcondition passes:
✅ Resource is valid

If it fails:
❌ Terraform reports an error

2. Syntax
A postcondition is also placed inside lifecycle:
resource "aws_instance" "app" {

  ami           = var.ami_id
  instance_type = var.instance_type

  lifecycle {

    postcondition {
      condition     = self.instance_type == "t3.micro"
      error_message = "Instance type must be t3.micro."
    }

  }
}

Notice this:
self.instance_type

Why self?
self refers to the current resource.
So:
self.instance_type

means:
"The instance_type of this resource."

3. Precondition vs Postcondition
This is the most important concept.
Precondition
Checks before Terraform performs the operation.
Input/configuration
      ↓
Precondition
      ↓
Create/Update

Postcondition
Checks after Terraform performs the operation.
Create/Update
      ↓
Postcondition
      ↓
Validate result

Easy memory
PRE  → BEFORE
POST → AFTER

4. Simple example
resource "aws_instance" "app" {

  ami           = var.ami_id
  instance_type = "t3.micro"

  lifecycle {

    postcondition {
      condition     = self.instance_type == "t3.micro"
      error_message = "The EC2 instance must use t3.micro."
    }

  }
}

Terraform creates/updates the resource and then checks:
self.instance_type == "t3.micro"

If true:
✅ Postcondition passed

If false:
❌ Postcondition failed

5. Why is this useful?
Imagine Terraform configuration looks correct, but the actual value returned by the provider is not what you expect.
A postcondition allows you to say:
"After Terraform creates this resource, verify that the result satisfies this requirement."

This is useful for validating important infrastructure properties.
6. Real DevOps example
Suppose your application requires a resource to be deployed in a specific environment.
resource "aws_instance" "app" {

  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Environment = var.environment
  }

  lifecycle {

    postcondition {
      condition     = self.tags["Environment"] == "prod"
      error_message = "The application must be deployed with Environment=prod."
    }

  }
}

After the resource is created, Terraform checks:
Actual resource
      ↓
Environment tag
      ↓
Is it "prod"?
      ↓
YES → ✅
NO  → ❌

7. self vs each.value
Since you've already learned for_each, this is important.
Suppose:
for_each = var.applications

Your input is:
payment → prod → 8080
user    → dev  → 8081
order   → stage → 8082

each.value
Represents the input configuration:
each.value.port

means:
The port specified in my var.applications.

self
Represents the current resource:
self.instance_type

means:
The instance type of the resource being checked.

Think:
each.value
    ↓
Input

self
    ↓
Current resource

8. Precondition + Postcondition together
You can use both.
resource "aws_instance" "app" {

  for_each = var.applications

  ami           = var.ami_id
  instance_type = var.instance_type

  lifecycle {

    precondition {
      condition     = each.value.port >= 1 && each.value.port <= 65535
      error_message = "Port must be between 1 and 65535."
    }

    postcondition {
      condition     = self.instance_type != ""
      error_message = "Instance type must be configured."
    }

  }
}

Flow:
Input
  ↓
PRECONDITION
  ↓
Create/Update
  ↓
POSTCONDITION
  ↓
Final result

9. Compare the three concepts you've learned
You now have:
replace_triggered_by
Other resource changes
        ↓
Replace this resource

precondition
Before operation
        ↓
Check condition
        ↓
Fail → Stop

postcondition
After operation
        ↓
Check result
        ↓
Fail → Report failure

🧠 Easy interview memory
PRE  → Check BEFORE
POST → Check AFTER
REPLACE → Replace when another resource changes

🎯 Topic 3 Practice Task
Let's combine Postconditions + your previous Data Types + Meta-Arguments.
Use:
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

  ami           = var.ami_id
  instance_type = var.instance_type

  lifecycle {

    postcondition {
      # your condition here
      # your error message here
    }

  }

  tags = {
    Name        = "${each.key}-server"
    environment = each.value.environment
    port        = each.value.port
  }
}

Requirement
After the resource is created/updated, verify that:
The EC2 instance has an instance type configured.

Use:
self.instance_type

Hint
You can check:
self.instance_type != ""

Your error message can be something like:
"Instance type must be configured."

🔥 Bonus
After completing the basic task, add the precondition from yesterday too:
Precondition:
port must be 1–65535

Postcondition:
instance_type must not be empty

That gives you a complete:
map(object)
     ↓
for_each
     ↓
precondition
     ↓
EC2 creation
     ↓
postcondition

Write the complete resource yourself and send it to me. I'll review it and then we'll do the Postconditions interview questions.

What is replace_triggered_by?
------------------------------

replace_triggered_by is a lifecycle argument that tells Terraform:

"If this other resource or attribute changes, replace this resource too."

Simple idea:

      Resource A changes
             ↓
      replace_triggered_by
             ↓
      Replace Resource B

It creates a replacement dependency between resources.

Why do we need it?

Imagine you have:

    EC2 Application Server
            +
    Launch Template

Your EC2 doesn't directly reference the Launch Template in a way Terraform can automatically use to decide replacement.

But you want:

"Whenever my launch template changes, replace my application server."

You can use:

      lifecycle {
        replace_triggered_by = [
          aws_launch_template.app
        ]
      }

Now:

    Launch Template changes
            ↓
    Terraform detects change
            ↓
    EC2 replacement triggered

Basic Syntax

    resource "aws_instance" "app" {
    
      ami           = var.ami_id
      instance_type = "t3.micro"
    
      lifecycle {
        replace_triggered_by = [
          aws_launch_template.app
        ]
      }
    }

The important part is:

    replace_triggered_by = [
      aws_launch_template.app
    ]

It tells Terraform:

If aws_launch_template.app changes, replace this aws_instance.app.

Very simple example

Let's imagine two resources:

      resource "aws_security_group" "app" {
        name = "app-sg"
      }
      
      resource "aws_instance" "app" {
        ami           = var.ami_id
        instance_type = "t3.micro"
      
        lifecycle {
          replace_triggered_by = [
            aws_security_group.app
          ]
        }
      }

Now imagine the security group changes in a way that Terraform considers a change.

Because of:

    replace_triggered_by = [
      aws_security_group.app
    ]

Terraform will also plan to replace the EC2.

Think:

    Security Group
          ↓
       changed
          ↓
    replace_triggered_by
          ↓
    EC2 replacement

Difference between depends_on and replace_triggered_by

This is very important for interviews.

depends_on

Controls creation/update ordering.

    Security Group
          ↓
    Create first
          ↓
    EC2

It basically says:

"Wait for this resource."

replace_triggered_by

Controls replacement behavior.

    Security Group changes
          ↓
    EC2 gets replaced

It basically says:

"If this changes, replace me."

Quick comparison
Meta-argument	Purpose
depends_on	Wait for another resource
replace_triggered_by	Replace this resource when another resource changes

Important difference from create_before_destroy

You already learned:

create_before_destroy = true

That does not decide when replacement happens.

It decides how the replacement happens.

    replace_triggered_by
            ↓
    Should I replace?
            ↓
    YES
            ↓
    create_before_destroy
            ↓
    How should I replace?
            ↓
    Create NEW → Destroy OLD

That's a very useful way to remember them.


Real DevOps example

    Imagine:
    
    Application
        ↓
    EC2
        ↓
    Application configuration

Suppose you manage application configuration separately:

    resource "aws_ssm_parameter" "app_config" {
      name  = "/app/config/version"
      type  = "String"
      value = var.app_version
    }

And you want the EC2 to be replaced whenever the application configuration version changes.

You could conceptually use:

    resource "aws_instance" "app" {
      ami           = var.ami_id
      instance_type = "t3.micro"
    
      lifecycle {
        replace_triggered_by = [
          aws_ssm_parameter.app_config
        ]
      }
    }

Then:

    app_version changes
           ↓
    SSM parameter changes
           ↓
    replace_triggered_by
           ↓
    EC2 replacement

This can be useful when a resource needs to be recreated whenever some related infrastructure/configuration changes.

One more important point

replace_triggered_by does not mean:

"Update the resource."

It means:

"Replace the resource."

So the result is conceptually:

    OLD RESOURCE
         ↓
    DESTROY
         +
    CREATE NEW RESOURCE

Or, if you also have:

create_before_destroy = true

then:

    CREATE NEW
         ↓
    DESTROY OLD


Actual code:
-----------

            resource "aws_security_group" "my_sg" {
                name = "app_sg"
            
            }
            
            resource "aws_instance" "my_instance" {
                ami = var.ami_id
                instance_type = var.instance_type
                for_each = var.applications
            
                lifecycle {
                    create_before_destroy = true
                    
                    replace_triggered_by =  [
                        aws_security_group.my_sg
                    ]
                    
                }
            
                tags = {
                    Name = "${each.key}-${var.instance_name}"
                    environment = each.value.environment
                    port = each.value.port
                }
            }




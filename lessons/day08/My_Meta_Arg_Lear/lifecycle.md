lifecycle
==========

What is lifecycle ?

Normally Terraform decides how to change a resource based on your configuration.

With lifecycle, we can tell Terraform:

    "Follow this special behavior when changing or destroying this resource."

Basic syntax:

resource "aws_instance" "app" {

    resource "aws_instance" "app" {
          lifecycle {
            # lifecycle rules
          }
        
        }

Terraform's lifecycle rules can help control what happens during changes.

prevent_destroy: Don't allow Terraform to destroy this resource.


The three important lifecycle rules:
-------------------------------------

create_before_destroy: Create the replacement before destroying the old resource.
--

                resource "aws_launch_template" "app" {
                name_prefix   = "payment-"
                image_id      = var.ami_id
                instance_type = "t3.micro"
              
                lifecycle {
                  create_before_destroy = true
                }
              }

* when Terraform needs to replace a resource, the normal sequence can be:

      Old Resource
           ↓
      Destroy
           ↓
      Create New Resource
* if i modify any field in the above resource and apply it the tf will delete the resource and re-create the resource by default.
* but when i use the life cycle rule called **create_before_destroy** it will create the new resource with modified configrarionfirst then it will delete the old resource

prevent_destroy: Prevent Terraform from destroying the resource.
---

    resource "aws_instance" "app" {
    
      ami           = var.ami_id
      instance_type = "t3.micro"
    
      lifecycle {
        prevent_destroy = true
      }
    
      tags = {
        Name = "prod-server"
      }
    }

* if i try to delete this ec2 instance it will not allow me to delete it.


ignore_changes: Tell Terraform to ignore changes to specified attributes. "Don't try to change this attribute when its value changes outside Terraform."
-----

    resource "aws_autoscaling_group" "app" {
      min_size         = 2
      max_size         = 10
      desired_capacity = 4
    
      lifecycle {
        ignore_changes = [
          desired_capacity
        ]
    }

Suppose AWS Auto Scaling changes the desired capacity from:

    4 → 7

* if i chaned the desired capacity from the console. tf detect the chnages but it will not try to change it back to tf desired state. it will keep the changes
  which we have done in console.

      Terraform says: 4
      AWS says:       7
             ↓
      ignore_changes
             ↓
      Terraform accepts the external change

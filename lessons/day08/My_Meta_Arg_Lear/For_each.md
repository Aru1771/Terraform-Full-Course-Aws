What is for_each?
------------------
for_each means:

     "Create one resource for each value in my collection."
     
for_each is a Terraform meta-argument used to create multiple instances of a resource from a collection.

For this topic, we'll use a set of strings.
------------------------------------------
      
      variable "environments" {
        type = set(string)
      }
      
      environments = ["dev", "stage", "prod"]


main.tf 
-------


    resource "aws_instance" "app-1" {
      for_each = var.environment
      ami = var.ami_id
      instance_type = var.instance_type
      tags = {
        Name = "${var.name}-${each.value}"
      }
    }

var.tf
------


    variable "environments" {
            type = set(string)
          }

values.tfvars
---------------

      environments = ["dev", "stage", "prod"]


The mechanisam of for_each:
-----------------------------
    for values dev --> it will create one resource
    for value prod --> it will create one resource
    for value stage --> it will create one resource

What is each.value?
-------------------

each.value means:

          The current value being processed by for_each.

for each value in the set it will create one resource.


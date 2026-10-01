resource "aws_instance" "app-1" {

  /* for_each = toset(var.application)*/
 /*count = 3*/


  for_each = var.application
  ami = var.ami_id
  instance_type = var.instance_type


  depends_on =  [

    aws_security_group.app_sg

  ]

  lifecycle {
    create_before_destroy = true

    ignore_changes = [

      tags

    ]
  }



  tags = {
    Name = "${each.key}-${var.name}"
    environemnt = each.value.environment
    port = each.value.port
  }
}


resource "aws_security_group" "app_sg" {
  name = var.sg_name
  description = var.sg_description
}



# tf.vars
--------

application = {

payment = {

environment = "prod",
port = 8080
},

user = {

environment = "dev",
port = 8081
},

order = {

environment = "stage"
port = 8082
}

}




# variable.tf
--------------


variable "ami_id" {
type = string

}

variable "instance_type" {
type = string
}

/*variable "ec2_count" {
type = number
}*/

variable "name" {
type = string
}


variable "sg_name" {
  type = string
}

variable "sg_description" {
  type = string
}



/*variable "environment" {
  type = set(string)
}*/


/*variable "instance_type" {
  type = map(string)
}*/



variable "application" {
  type = map(object({
    environment = string
    port = number
  }))
}
"variable.tf

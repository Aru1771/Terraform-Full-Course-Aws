Terraform Functions — Day 1: Fundamentals
===========================================

Theory — What are Terraform functions?
--------------------------------------

* A Terraform function is a built-in operation that takes input values, processes them, and returns a result.
* For example, suppose you have an AWS application with three EC2 instances and want to know how many instance names are configured.

* Instead of counting manually, you can use a function:

      length(["web-1", "web-2", "web-3"])

Result:

     3


Here, length() counts the elements in the list.

Other examples:

    upper("empower")


Result: "EMPOWER"

    max(10, 20, 30)


Result: 30

    tostring(8081)


Result: "8081"

Function syntax
---------------

The general syntax is:

    function_name(argument1, argument2, ...)


For example:

    length("Terraform")

Important:

    Terraform functions are not resources. 
    You don't create an AWS resource by calling length() or upper(). 
    Functions calculate values that can be used in variables, locals, resource arguments, outputs and expressions.


Your first functions
--------------------

Let's begin with four functions. We'll study more function families in the upcoming days.

Function 1: length()

* Returns the number of characters in a string or the number of elements in a collection.

      length("Terraform") 


Result: 9

Function 2: upper()

Converts a string to uppercase.

    upper("empower")


Result: "EMPOWER"
 

Related function:

    lower("PRODUCTION")


Result: "production"

* AWS use case: Normalize environment names or text values before using them in naming conventions. 
* Remember that AWS resource names have service-specific naming rules, so uppercase isn't appropriate everywhere.

Function 3: tostring()

Converts a supported value to a string.

    tostring(8081)


Result: "8081"

* AWS use case: Produce string values for configuration templates or outputs when a string is required.


Function 4: max()

Returns the largest of its numeric arguments.

    max(2, 4, 8)


Result: 8

* AWS use case: Calculate a minimum operational threshold or choose the largest value from a set of numeric configuration inputs.
* These functions return values. They do not create resources or modify your AWS environment.

Real-world AWS example

Imagine you're configuring an application deployment with these inputs:

    variable "application_name" {
      type    = string
      default = "empower"
    }
    
    variable "environment" {
      type    = string
      default = "prod"
    }
    
    variable "ports" {
      type    = list(number)
      default = [8081, 8082, 8083]
    }


You can use functions inside locals:

    locals {
      app_name_upper = upper(var.application_name)
    
      port_count = length(var.ports)
    
      primary_port = tostring(var.ports[0])
    }
    

These locals evaluate to:

    app_name_upper = "EMPOWER"
    port_count     = 3
    primary_port   = "8081"


Why is this useful?

    - upper() transforms the application name.
    - length() counts the configured ports.
    - tostring() converts the selected port to a string.

* No AWS resources are created by these expressions. They prepare values that other Terraform configuration can consume.

* Practise upper(), length(), tostring(), and max() in terraform console, then attempt the separate integrated revision challenge at the end.

Task:


    1. Convert "empower" to uppercase.
    Function: {question.fn}
    2. Count the elements in ["dev", "stage", "prod", "uat"].
    Function: {question.fn}
    3. Convert the number 443 into a string.
    Function: {question.fn}
    4. Find the largest number among 12, 45, 27, and 9.
    Function: {question.fn}
    5. Count these five application ports: [8080, 8081, 8082, 8083, 8084].
    Function: {question.fn}



vars.tf

    variable "application" {
    
      type = string
    }
    
    variable "environments" {
      type = list(string)
    
    }
    
    variable "port" {
      type = number
    
    }
    
    variable "values" {
      type = list(number)
    
    }
    
    variable "ports" {
      type = list(number)
      
    }

.tfvars

    application = "empower"
    environments = ["dev", "stage", "prod", "uat"]
    port = 443
    values = [12, 45, 27, 9]
    ports = [2343, 8907, 7654, 4532]

locals.tf

    locals {
      application_name = upper(var.application)
    
      app_environment = length(var.environments)
    
      port = tostring(var.port)
    
      values = max(var.values...)
    
    
      app_ports = length(var.ports)
    }

output.tf

    output "application_name" {
        value = local.application_name
      
    }
    
    output "application_environment" {
        value = local.app_environment
    }
    
    output "app_port" {
        value = local.port
    }
    
    output "app_values" {
        value = local.values
    }
    
    output "app_ports" {
        value = local.app_ports
      
    }
    
    

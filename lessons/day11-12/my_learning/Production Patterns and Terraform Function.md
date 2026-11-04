Day 12 — Production Patterns and Terraform Function Mastery
=============================================================

The goal is not just to remember individual functions. You should be able to combine them to solve real AWS DevOps engineering problems.

We'll use an Empower application running on AWS as our production scenario.

* Theory — Production-grade Terraform patterns

Pattern 1: Use locals to centralize reusable logic
----------------------------------------------------
In production Terraform

For example:
    
    variable "environment" {
      type    = string
      default = "prod"
    }
    
    locals {
      application_name = "Empower"
    
      common_tags = {
        Application = local.application_name
        Environment = var.environment
        ManagedBy   = "Terraform"
      }
    
      resource_prefix = lower(
        "${local.application_name}-${var.environment}"
      )
    }
    
    
* Why is this useful?

      - One place to maintain common values.
      - Consistent resource naming.
      - Consistent tags across AWS resources.
      - Easier maintenance.

Pattern 2: Validate optional values using try() and can():
----------------------------------------------------------

* Imagine your application configuration contains optional settings.

      locals {
        app_config = {
          name        = "Empower"
          environment = "prod"
          port        = "8081"
        }
      
        application_port = try(
          tonumber(local.app_config.port),
          8080
        )
      
        is_valid_port = can(
          tonumber(local.app_config.port)
        )
      }


Here:

    - tonumber() converts the port string to a number.
    - try() provides a fallback if conversion fails.
    - can() checks whether the conversion succeeds.
* Production use: Handling optional configuration values safely.

Remember: can() checks whether an expression can be evaluated without an error. 
It does not validate every business rule, such as whether a port is within an acceptable range.


Pattern 3: Dynamic infrastructure configuration
-----------------------------------------------
Imagine your application has multiple environments.

      locals {
        environments = {
          dev = {
            instance_type = "t3.micro"
            replicas      = 1
          }
      
          stage = {
            instance_type = "t3.small"
            replicas      = 2
          }
      
          prod = {
            instance_type = "t3.large"
            replicas      = 4
          }
        }
      
        selected_environment = local.environments["prod"]
      }
      

Now access values using:
local.selected_environment.instance_type
Output:
t3.large

And:
local.selected_environment.replicas
Output:
4

* Production use: Reuse one Terraform configuration for development, staging, and production.

Pattern 4: Combine for_each with Terraform functions
------------------------------------------------------

Instead of manually creating every subnet, calculate subnet CIDRs and create resources from a map.

    locals {
      vpc_cidr = "10.0.0.0/16"
    
      subnet_config = {
        public_1  = { netnum = 0, type = "public" }
        public_2  = { netnum = 1, type = "public" }
        private_1 = { netnum = 10, type = "private" }
        private_2 = { netnum = 11, type = "private" }
      }
    
      subnets = {
        for name, config in local.subnet_config :
        name => {
          cidr = cidrsubnet(local.vpc_cidr, 8, config.netnum)
          type = config.type
        }
      }
    }
    

Now you can create subnet resources:

      resource "aws_subnet" "app" {
        for_each = local.subnets
      
        vpc_id            = aws_vpc.main.id
        cidr_block        = each.value.cidr
        availability_zone = data.aws_availability_zones.available.names[
          each.key == "public_1" || each.key == "private_1" ? 0 : 1
        ]
      
        tags = {
          Name = each.key
          Type = each.value.type
        }
      }


This combines:

    - for expression — builds a subnet map.
    - cidrsubnet() — calculates subnet CIDRs.
    - for_each — creates multiple AWS subnet resources.
    - Conditional expression — selects an Availability Zone.
The resulting subnet allocation is:

      Subnet	  CIDR	       Type
      public_1	10.0.0.0/24	  Public
      public_2	10.0.1.0/24	  Public
      private_1	10.0.10.0/24	Private
      private_2	10.0.11.0/24	Private

* This example creates subnet resources, but you would still need route tables, routes, and appropriate internet/NAT gateway configuration to make them function as public or private subnets.

Pattern 5: Deployment artifact validation
-----------------------------------------
Suppose your Jenkins pipeline produces a ZIP file.

You can calculate its checksum:
    
    locals {
      artifact_path = "${path.module}/empower-app.zip"
    
      artifact_checksum = filesha256(
        local.artifact_path
      )
    }


You can expose that value:

    output "artifact_checksum" {
      value = local.artifact_checksum
    }


If the artifact changes, the checksum normally changes too.

Production use: Pass the checksum into deployment metadata or use it to trigger a controlled redeployment.

Pattern 6: Generate JSON deployment metadata
----------------------------------------------
Combine your functions to create deployment information.

    locals {
      deployment_metadata = {
        application = "Empower"
        environment = var.environment
        port        = 8081
        deployment_date = formatdate(
          "YYYY-MM-DD",
          plantimestamp()
        )
        checksum = local.artifact_checksum
      }
    
      deployment_json = jsonencode(
        local.deployment_metadata
      )
    }
    
    output "deployment_json" {
      value = local.deployment_json
    }
    

Now Terraform can produce JSON metadata that can be consumed by another deployment process.

Practical task — Build a production-style Terraform configuration

Let's combine these patterns into one small project.

Project requirements:

    1. Create an AWS VPC with CIDR 10.0.0.0/16.
    2. Create two public and two private subnets.
    3. Use for_each to create subnets.
    4. Use cidrsubnet() to calculate subnet CIDRs.
    5. Use environment-specific EC2 instance types.
    6. Generate common AWS tags.
    7. Calculate the application ZIP checksum.
    8. Generate JSON deployment metadata.
Project structure

    terraform-functions-day12/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── empower-app.zip


Step 1 — variables.tf

    variable "environment" {
      type    = string
      default = "prod"
    
      validation {
        condition = contains(
          ["dev", "stage", "prod"],
          var.environment
        )
        error_message = "Environment must be dev, stage, or prod."
      }
    }


Step 2 — main.tf

      provider "aws" {
        region = "ap-south-1"
      }
      
      data "aws_availability_zones" "available" {
        state = "available"
      }
      
      locals {
        application_name = "Empower"
      
        environment_config = {
          dev = {
            instance_type = "t3.micro"
            replicas      = 1
          }
      
          stage = {
            instance_type = "t3.small"
            replicas      = 2
          }
      
          prod = {
            instance_type = "t3.large"
            replicas      = 4
          }
        }
      
        selected_config = local.environment_config[var.environment]
      
        common_tags = {
          Application = local.application_name
          Environment = var.environment
          ManagedBy   = "Terraform"
        }
      
        vpc_cidr = "10.0.0.0/16"
      
        subnet_config = {
          public_1  = { netnum = 0,  az_index = 0, type = "public" }
          public_2  = { netnum = 1,  az_index = 1, type = "public" }
          private_1 = { netnum = 10, az_index = 0, type = "private" }
          private_2 = { netnum = 11, az_index = 1, type = "private" }
        }
      
        subnets = {
          for name, config in local.subnet_config :
          name => {
            cidr = cidrsubnet(local.vpc_cidr, 8, config.netnum)
            az   = data.aws_availability_zones.available.names[config.az_index]
            type = config.type
          }
        }
      
        artifact_checksum = filesha256(
          "${path.module}/empower-app.zip"
        )
      
        deployment_metadata = {
          application   = local.application_name
          environment   = var.environment
          instance_type = local.selected_config.instance_type
          replicas      = local.selected_config.replicas
          checksum      = local.artifact_checksum
          planned_date  = formatdate("YYYY-MM-DD", plantimestamp())
        }
      }
      
      resource "aws_vpc" "main" {
        cidr_block           = local.vpc_cidr
        enable_dns_support   = true
        enable_dns_hostnames = true
      
        tags = merge(local.common_tags, {
          Name = "${local.application_name}-${var.environment}-vpc"
        })
      }
      
      resource "aws_subnet" "app" {
        for_each = local.subnets
      
        vpc_id            = aws_vpc.main.id
        cidr_block        = each.value.cidr
        availability_zone = each.value.az
      
        tags = merge(local.common_tags, {
          Name = "${each.key}-${var.environment}"
          Type = each.value.type
        })
      }
      

Step 3 — outputs.tf

    output "vpc_id" {
      value = aws_vpc.main.id
    }
    
    output "subnet_cidrs" {
      value = {
        for name, subnet in aws_subnet.app :
        name => subnet.cidr_block
      }
    }
    
    output "selected_instance_type" {
      value = local.selected_config.instance_type
    }
    
    output "deployment_metadata_json" {
      value = jsonencode(local.deployment_metadata)
    }


Step 4 — Execute
    
    Make sure empower-app.zip exists before running Terraform.
    terraform init
    terraform fmt
    terraform validate
    terraform plan


Review the plan before applying. This project creates actual AWS resources if you apply it, so check your AWS account, permissions, and potential costs first.

Real-world AWS DevOps use cases:
---------------------------------

    Production requirement	Terraform functions/patterns
    Generate consistent resource names	lower(), format()
    Select environment configuration	lookup(), try(), map access
    Generate subnet CIDRs	cidrsubnet(), cidrsubnets()
    Detect artifact changes	filesha256()
    Generate deployment metadata	jsonencode()
    Handle optional configuration	try(), can()
    Generate common resource tags	merge()
    Select deployment values	Conditional expressions
    Create multiple AWS resources	for_each and for expressions

Production engineering lesson: Terraform functions help transform and validate configuration data. 
They don't replace good infrastructure design, testing, state management, or security practices.

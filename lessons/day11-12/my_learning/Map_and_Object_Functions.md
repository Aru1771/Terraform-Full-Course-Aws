Day 4: Map and Object Functions
===============================

* Today we'll learn how to access, combine, transform, and safely read Terraform maps and objects in real AWS DevOps scenarios.

Today's learning objectives:

    - Extract map keys and values.
    - Retrieve values safely with lookup().
    - Merge configuration maps with merge().
    - Build maps using zipmap() and tomap().
    - Handle missing or optional attributes using try().

Functions: keys(), values(), lookup(), merge(), zipmap(), tomap(), try().

Theory — Map and object basics:

* A map stores values under keys, typically with a consistent value type. An object stores named attributes that can have different types.

      variable "app_config" {
        type = map(string)
      
        default = {
          environment = "prod"
          application = "empower"
          owner       = "devops"
        }
      }

An object can have a defined structure:

    variable "server" {
      type = object({
        name          = string
        instance_count = number
        monitoring    = bool
      })
    
      default = {
        name           = "empower-prod"
        instance_count = 2
        monitoring     = true
      }
    }


* This distinction matters when you define complex Terraform inputs such as your map(object(...)) environment configurations.

# Learn the functions

* A. keys() and values()

* keys() returns the keys of a map in lexicographical order.

      keys({
        dev  = "t3.micro"
        prod = "t3.large"
        stage = "t3.small"
      })

Result:

    ["dev", "prod", "stage"]


* values() returns the corresponding values in the same key order.

      values({
        dev   = "t3.micro"
        prod  = "t3.large"
        stage = "t3.small"
      })

Result:

    ["t3.micro", "t3.large", "t3.small"]

* AWS DevOps use case: Inspect configured environments or instance types and create useful outputs for review.

# lookup() — retrieve a value with a default

Syntax:

    lookup(map, key, default)

Example:

    lookup(
      { dev = "t3.micro", prod = "t3.large" },
      "stage",
      "t3.small"
    )

Result: "t3.small" because the stage key is absent.

* Without a default, lookup() returns an error if the key doesn't exist.

* Use case: Set a default instance type or optional configuration value. Note that a default can hide a typo in a required environment name,
* so validate required inputs rather than silently defaulting them.

# merge() — combine maps

* merge() combines two or more maps. If the same key exists in multiple maps, the rightmost map wins.

      merge(
        { environment = "dev", instance_type = "t3.micro" },
        { environment = "prod" }
      )


Result:

    {
      environment   = "prod"
      instance_type = "t3.micro"
    }

AWS DevOps use case: Combine common application tags with environment-specific tags.

    locals {
      common_tags = {
        Application = "Empower"
        ManagedBy   = "Terraform"
      }
    
      environment_tags = {
        Environment = "prod"
        Owner       = "platform"
      }
    
      final_tags = merge(
        local.common_tags,
        local.environment_tags
      )
    }


# zipmap() — build a map from two lists

* zipmap() pairs elements from a list of keys with corresponding elements from a list of values.

      zipmap(
        ["dev", "stage", "prod"],
        ["t3.micro", "t3.small", "t3.large"]
      )


Result:

    {
      dev   = "t3.micro"
      stage = "t3.small"
      prod  = "t3.large"
    }

* Both lists should have the same number of elements.

* Use case: Convert environment and instance-type lists into a map for further processing.

# tomap() — convert a value into a map

    tomap({
      dev  = "t3.micro"
      prod = "t3.large"
    })


* Result: a map with the two environment keys and their string values.

* Terraform may infer a compatible common value type when converting mixed values. Use explicit type constraints when you need predictable input structures.

* Use case: Convert a compatible map-like value into a map for a module input or local value.

# try() — handle expressions that might fail

* try() evaluates its arguments in order and returns the first one that doesn't produce an error.

      try(
        var.app_config["instance_type"],
        "t3.micro"
      )


* If the key is absent, it returns "t3.micro".

* A more realistic nested-object example:

      locals {
        volume_size = try(
          var.environment_config.root_volume.size,
          20
        )
      }


* If that attribute access fails, Terraform uses 20.

* Important: try() catches errors in expressions; it doesn't catch every possible problem in your configuration or make invalid resource arguments valid.
* Use it for genuinely optional or variable-shaped data, not to hide errors in required configuration.

# Practical task — Terraform Console

Run:

    terraform init
    terraform console


Try each expression and predict the output first.

    Function	Expression
    
    keys()	  keys({dev = "t3.micro", prod = "t3.large"})
    values()	values({dev = "t3.micro", prod = "t3.large"})
    lookup()	lookup({dev = "t3.micro"}, "prod", "t3.small")
    merge()	  merge({env = "dev"}, {env = "prod", owner = "devops"})
    zipmap()	zipmap(["dev", "prod"], ["small", "large"])
    tomap()	  tomap({dev = "t3.micro", prod = "t3.large"})
    try()	    try({app = "empower"}.missing, "default")

Exit with exit.

Note: If your Terraform console version rejects direct attribute syntax on a map literal in the last example, 
assign the map to a local or variable and use try() on that value instead.


Real-world AWS DevOps use case
Suppose your pipeline deploys Empower across three environments. Each environment has its own instance type, volume size, and tags, while all environments share common tags.

Create a main.tf file:


        variable "environment_config" {
          type = map(object({
            instance_type = string
            volume_size   = number
            tags          = map(string)
          }))
        
          default = {
            dev = {
              instance_type = "t3.micro"
              volume_size   = 20
              tags = {
                Environment = "dev"
              }
            }
        
            stage = {
              instance_type = "t3.small"
              volume_size   = 30
              tags = {
                Environment = "stage"
              }
            }
        
            prod = {
              instance_type = "t3.large"
              volume_size   = 50
              tags = {
                Environment = "prod"
              }
            }
          }
        }
        
        locals {
          common_tags = {
            Application = "Empower"
            ManagedBy   = "Terraform"
          }
        
          environment_names = keys(var.environment_config)
        
          instance_types = values({
            for env, config in var.environment_config :
            env => config.instance_type
          })
        
          production_config = lookup(
            var.environment_config,
            "prod",
            {
              instance_type = "t3.micro"
              volume_size   = 20
              tags          = {}
            }
          )
        
          production_tags = merge(
            local.common_tags,
            local.production_config.tags
          )
        
          environment_type_map = zipmap(
            keys(var.environment_config),
            local.instance_types
          )
        
          optional_volume_size = try(
            var.environment_config["prod"].volume_size,
            20
          )
        }
        
        output "environment_names" {
          value = local.environment_names
        }
        
        output "instance_types" {
          value = local.instance_types
        }
        
        output "production_config" {
          value = local.production_config
        }
        
        output "production_tags" {
          value = local.production_tags
        }
        
        output "environment_type_map" {
          value = local.environment_type_map
        }
        
        output "optional_volume_size" {
          value = local.optional_volume_size
        }



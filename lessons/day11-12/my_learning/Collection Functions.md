Today's learning objectives:
===========================

By the end of Day 3, you'll be able to:

    - Convert between collection types.
    - Combine lists and remove duplicate values.
    - Check whether a value exists and find its position.
    - Select elements and extract list ranges.
    - Sort, reverse, flatten, and divide collections.

Functions: tolist(), toset(), concat(), distinct(), contains(), index(), element(), slice(), sort(), reverse(), flatten(), chunklist().


Theory — Lists, sets, and collections

* Before learning the functions, understand the three collection types we'll use most.

List

* An ordered collection. Duplicate values are allowed.

      ["dev", "stage", "prod", "dev"]

Set

* A collection of unique values. It doesn't guarantee a meaningful order.

       toset(["dev", "stage", "prod", "dev"])

Result: a set containing dev, stage, and prod once each.

Tuple

* An ordered sequence whose elements can have different types.

    ["empower", 8081, true]

Remember: a function that converts a collection's type doesn't necessarily change the underlying information in the way you might expect.
In particular, converting a list to a set removes duplicates and loses meaningful ordering.

Collection functions — Part A:
------------------------------

#tolist() and toset()

* tolist() converts a compatible value into a list.

      tolist(["dev", "stage", "prod"])

* toset() converts a compatible value into a set, removing duplicates.

      toset(["dev", "prod", "dev"])


AWS DevOps example: You receive a list of environment names from a pipeline and need a unique collection of environments.

#concat() and distinct()

concat() joins multiple lists into one list.

    concat(["dev", "stage"], ["prod"])

Result:

    ["dev", "stage", "prod"]

distinct() removes duplicate values while preserving the first occurrence of each value.

    distinct(["dev", "prod", "dev", "stage"])

Result:

    ["dev", "prod", "stage"]

* AWS DevOps example: Combine ports or environment lists from multiple configuration sources and remove duplicate entries.

Collection functions — Part B
-----------------------------

#contains() and index()

contains() — check whether a value exists

    contains(["dev", "stage", "prod"], "prod")
    # true

Returns true if the value exists, otherwise false.

index() — find a value's position

    index(["dev", "stage", "prod"], "prod")
    # 2

* Positions are zero-based: dev is at index 0, stage at 1, and prod at 2. It returns an error if the value isn't found.

* AWS DevOps example: Check whether prod is an allowed deployment environment before using it in your configuration.

#element() and slice()

element() — select an element by index

    element(["web-a", "web-b", "web-c"], 1)
    # "web-b"

* Note: element() wraps around when the index exceeds the list length. For normal index access, list[index] is usually clearer.

# slice() — extract a range

slice(["dev", "stage", "prod", "uat"], 1, 3)

["stage", "prod"]

* The start index is included, but the end index is excluded.
  
* AWS DevOps example: Select a subset of configured environments for a deployment batch.

# sort() and reverse()

sort(["prod", "dev", "stage"])
["dev", "prod", "stage"]

reverse(["dev", "stage", "prod"])
 ["prod", "stage", "dev"]

* - sort() returns values in ascending order.
* - reverse() reverses the existing order; it does not sort the values first.
* For strings, sorting is lexicographical, not necessarily based on numeric meaning.

# flatten() and chunklist()

* These two functions are particularly useful when working with nested configuration data.

flatten() — flatten nested lists

    flatten([
      ["dev", "stage"],
      ["prod", "uat"]
    ])


Result:
  
    ["dev", "stage", "prod", "uat"]

chunklist() — split a list into fixed-size groups

    chunklist(
      ["dev", "stage", "prod", "uat"],
      2
    )


Result:
    
    [
      ["dev", "stage"],
      ["prod", "uat"]
    ]




* chunklist() can produce a smaller final group if the number of elements isn't divisible by the chunk size.

* AWS DevOps example: Flatten nested environment configurations or divide a list of servers into smaller deployment batches.

Practical task — Terraform Console
-----------------------------------

You don't need AWS credentials to practise these expressions.
Run:
terraform init
terraform console

    Function	      Expression to test
    tolist()	-->   tolist(["dev", "prod"])
    toset()	        toset(["dev", "prod", "dev"])
    concat()	      concat(["dev"], ["stage", "prod"])
    distinct()	    distinct(["a", "b", "a", "c"])
    contains()	    contains(["dev", "prod"], "prod")
    index()	        index(["dev", "stage", "prod"], "stage")
    element()      	element(["web-a", "web-b"], 1)
    slice()	        slice(["a", "b", "c", "d"], 1, 3)
    sort()	        sort(["prod", "dev", "stage"])
    reverse()      	reverse(["dev", "stage", "prod"])
    flatten()	      flatten([["a", "b"], ["c"]])
    chunklist()	    chunklist(["a", "b", "c", "d"], 2)


Real-world AWS DevOps task
--------------------------

* Imagine your pipeline provides the environments and deployment batches for the Empower application.
Create a main.tf file:

      variable "environments" {
        type    = list(string)
        default = ["prod", "dev", "stage", "dev"]
      }
      
      variable "deployment_batches" {
        type = list(list(string))
      
        default = [
          ["web-01", "web-02"],
          ["web-03", "web-04"]
        ]
      }
      
      locals {
        unique_environments = distinct(var.environments)
      
        sorted_environments = sort(
          local.unique_environments
        )
      
        environment_count = length(
          local.unique_environments
        )
      
        production_enabled = contains(
          local.unique_environments,
          "prod"
        )
      
        all_servers = flatten(
          var.deployment_batches
        )
      
        first_batch = slice(
          local.all_servers,
          0,
          2
        )
      }
      
      output "unique_environments" {
        value = local.unique_environments
      }
      
      output "sorted_environments" {
        value = local.sorted_environments
      }
      
      output "environment_count" {
        value = local.environment_count
      }
      
      output "production_enabled" {
        value = local.production_enabled
      }
      
      output "all_servers" {
        value = local.all_servers
      }
      
      output "first_batch" {
        value = local.first_batch
      }

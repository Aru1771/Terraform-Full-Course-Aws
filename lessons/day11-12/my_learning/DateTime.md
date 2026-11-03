Date/Time and Advanced Combinations
===================================

Theory — Date and Time Functions
--------------------------------

# Function 1: timestamp()

Purpose: Returns the current UTC date and time.

    timestamp()

Example output: 2026-11-03T01:52:00Z

The actual output changes as time passes.
AWS DevOps use case: Record when a deployment configuration was evaluated.

    locals {
      deployment_time = timestamp()
    }
    
    output "deployment_time" {
      value = local.deployment_time
    }

Important:
timestamp() is an impure function. 
Its value changes during Terraform evaluation. 
Using it directly in resource arguments can cause Terraform to detect changes repeatedly.


# Function 2: formatdate()

Purpose: Converts a timestamp into a specific date/time format.
Syntax:

    formatdate(format, timestamp)
Example: formatdate("YYYY-MM-DD", "2026-11-03T10:30:00Z")

Output: 2026-11-03

Another example: formatdate("DD/MM/YYYY", "2026-11-03T10:30:00Z")
Output: 03/11/2026

More examples:

formatdate("YYYY", "2026-11-03T10:30:00Z")
# 2026

formatdate("MMM DD, YYYY", "2026-11-03T10:30:00Z")
# Nov 03, 2026
AWS DevOps use case: Format deployment dates for reports, logs, and resource metadata.


# Function 3: timeadd()
Purpose: Adds a duration to a timestamp.

Syntax: timeadd(timestamp, duration)

Example: timeadd("2026-11-03T10:00:00Z", "2h")
Output:
2026-11-03T12:00:00Z

Other examples:
timeadd("2026-11-03T10:00:00Z", "30m")
# 2026-11-03T10:30:00Z

timeadd("2026-11-03T10:00:00Z", "24h")
# 2026-11-04T10:00:00Z

timeadd("2026-11-03T10:00:00Z", "168h")
# 7 days later

# Function 4: plantimestamp()
Purpose: Returns the timestamp when the current Terraform plan was created.

    plantimestamp()

Unlike timestamp(), this function is stable during the plan/apply cycle for a given plan.
AWS DevOps use case: Record when a Terraform plan was generated.
Duration units:

    - s = seconds
    - m = minutes
    - h = hours
AWS DevOps use case: Calculate a future timestamp for a scheduled maintenance window or deployment record.

<img width="952" height="448" alt="image" src="https://github.com/user-attachments/assets/daa1d16b-ab4d-49c8-a61d-7fb7794787db" />

<img width="1152" height="317" alt="image" src="https://github.com/user-attachments/assets/e2aabc23-369a-4aad-ac1a-645a0068b4c4" />

Create main.tf:

    locals {
      application_name = "Empower"
    
      deployment_time = timestamp()
    
      deployment_date = formatdate(
        "DD/MM/YYYY",
        local.deployment_time
      )
    
      maintenance_time = timeadd(
        local.deployment_time,
        "24h"
      )
    
      plan_created_time = plantimestamp()
    }
    
    output "application_name" {
      value = local.application_name
    }
    
    output "deployment_time" {
      value = local.deployment_time
    }
    
    output "deployment_date" {
      value = local.deployment_date
    }
    
    output "maintenance_time" {
      value = local.maintenance_time
    }
    output "plan_created_time" {
      value = local.plan_created_time
    }



Observe the timestamp values.

    Remember: timestamp() can change between evaluations, while plantimestamp() is tied to the current plan.


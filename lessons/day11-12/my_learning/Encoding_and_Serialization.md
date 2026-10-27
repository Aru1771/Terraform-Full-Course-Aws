Encoding and Serialization
==========================

* Today we'll learn how Terraform converts data between Terraform values, JSON, and YAML. 
* These skills are useful for AWS IAM policies, Kubernetes manifests, application configuration, and automation.

# Function 1: jsonencode()

* Converts a Terraform value into a JSON-formatted string.

      jsonencode({
        application = "Empower"
        port        = 8081
        monitoring  = true
      })


Output:

    {"application":"Empower","monitoring":true,"port":8081}


* The JSON object contains a string, a number, and a Boolean. Terraform handles the correct JSON data types automatically.

* AWS DevOps use case: Generate JSON for an IAM policy or other AWS configuration.

        Example:
        locals {
          policy_json = jsonencode({
            Version = "2012-10-17"
            Statement = [{
              Effect   = "Allow"
              Action   = ["s3:ListBucket"]
              Resource = "*"
            }]
          })
        }


* This produces valid JSON text that can be passed to an AWS resource argument expecting a JSON policy document. In production, restrict Resource to the required bucket or resources rather than using "*" unnecessarily.

# Function 2: jsondecode()

* Converts a JSON string into a Terraform value, such as an object, list, number, or Boolean.

      jsondecode("{\"application\":\"Empower\",\"port\":8081}")


Result conceptually:

      {
        application = "Empower"
        port        = 8081
      }


* AWS DevOps use case: Parse JSON returned by an API or stored in a configuration file.

Remember the difference:

    - jsonencode() — Terraform value → JSON string.
    - jsondecode() — JSON string → Terraform value.

# Function 3: yamlencode()

* Converts a Terraform value into a YAML-formatted string.

      yamlencode({
        application = "Empower"
        replicas    = 3
        enabled     = true
      })


Output:

      application: Empower
      enabled: true
      replicas: 3


* AWS DevOps use case: Generate YAML configuration for Kubernetes resources, Helm-related inputs, or application configuration.

* For example, you can generate a Kubernetes Deployment manifest from Terraform variables and provide the rendered YAML to a tool that applies the manifest.

# Function 4: yamldecode()

* Converts a YAML string into a Terraform value.

      yamldecode(<<-EOT
      application: Empower
      replicas: 3
      enabled: true
      EOT
      )


Result conceptually:

    {
      application = "Empower"
      replicas    = 3
      enabled     = true
    }


The <<-EOT ... EOT syntax is a Terraform heredoc, which lets you write a multiline string.

Remember:

# Function	Conversion

      yamlencode()	Terraform value → YAML string
      yamldecode()	YAML string → Terraform value

* JSON and YAML can represent similar data, but they have different syntax. Use the format expected by the tool or resource you're integrating with.

Function 5: base64encode()

    Converts a string into Base64-encoded text.
    base64encode("Empower")


Output:

    RW1wb3dlcg==


* This is an encoding operation, not encryption. Anyone with the encoded value can decode it.

* AWS DevOps use case: Encode text for an API or resource argument that explicitly requires Base64. For files, remember the filebase64() function you learned on Day 7.

# Function 6: base64decode()

* Converts Base64-encoded text back into the original string.

      base64decode("RW1wb3dlcg==")


Output:

    Empower


Memory trick: encode means convert into the requested format; decode means convert back into a usable value.


      Function	      Meaning
      jsonencode()	  Terraform value → JSON
      jsondecode()	  JSON → Terraform value
      yamlencode()	  Terraform value → YAML
      yamldecode()   	YAML → Terraform value
      base64encode()	Text → Base64
      base64decode()	Base64 → original text

  

File and Template Functions
===========================

Theory

# Function 1: file()

* Reads the contents of an existing file and returns it as a string.

* Suppose you have a file named app.conf:

      APP_NAME=Empower
      APP_PORT=8081
      ENVIRONMENT=dev


Terraform expression:

    file("${path.module}/app.conf")


* This returns the file's contents as a string.

* AWS DevOps use case: Read an existing configuration file that Terraform needs to use as input.

* Important: file() reads a file that already exists when Terraform evaluates the configuration.
  It cannot read a file that will only be created later during the same apply operation.


# Function 2: filebase64()

* Reads a file and returns its contents encoded as Base64.

      filebase64("${path.module}/userdata.sh")


* AWS DevOps use case: Read an existing script as Base64 text when a resource or API requires encoded file content.
* Note that some AWS resource arguments accept plain text and handle encoding themselves, so use this only when the target argument specifically requires Base64.

# Function 3: templatefile()

* Reads a template file, substitutes variables, and returns the rendered text.

* Suppose userdata.tftpl contains:

      #!/bin/bash
      APP_NAME="${app_name}"
      APP_PORT="${app_port}"
      echo "Deploying ${app_name} on port ${app_port}"


Terraform:

    locals {
      rendered_user_data = templatefile("${path.module}/userdata.tftpl", {
        app_name = "Empower"
        app_port = 8081
      })
    }


The rendered text becomes:

    #!/bin/bash
    APP_NAME="Empower"
    APP_PORT="8081"

* echo "Deploying Empower on port 8081"


* AWS DevOps use case: Generate an EC2 user-data script using environment-specific values without duplicating the whole script for dev, staging, and production.

* Template variables are supplied through the second argument as a map. Every variable referenced in the template must be supplied in that map.

# Function 4: fileexists()

* Correction to the note above: fileexists() is a built-in Terraform function. It returns true if the given path exists and is a regular file; otherwise, it returns false.
fileexists("${path.module}/userdata.sh")

Possible output:

    true

* This is useful when a deployment script or configuration file is optional.


# Function 5: fileset()

* Returns a set of file paths matching a pattern in a directory.

Suppose your project contains:

    scripts/
    ├── deploy.sh
    ├── cleanup.sh
    └── patch.sh


Terraform:

    fileset("${path.module}/scripts", "*.sh")


Output (a set; ordering is not guaranteed by the function):

    [
      "cleanup.sh",
      "deploy.sh",
      "patch.sh",
    ]


AWS DevOps use case: Discover a collection of existing scripts or configuration files that need to be processed.

Remember that fileset() returns matching file paths relative to the specified directory, not their contents.

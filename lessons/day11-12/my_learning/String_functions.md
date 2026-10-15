String functions
================


Change letter case:
-------------------

upper() — convert to uppercase

    upper("empower")
    # "EMPOWER"




lower() — convert to lowercase

    lower("PRODUCTION")
    # "production"
    



title() — capitalize words

    title("aws devops engineer")
    # "Aws Devops Engineer"


Remove unwanted characters or spaces
-------------------------------------

trimspace() — remove whitespace at the beginning and end

    trimspace("  production  ")
    # "production"

trim() — remove specified characters from both ends

    trim("!!empower!!", "!")
    # "empower"


trimprefix() and trimsuffix()

    trimprefix("app-empower", "app-")
    # "empower"

    trimsuffix("empower-prod", "-prod")
    # "empower"

replace() — replace matching text

    replace("empower_dev", "_", "-")
    # "empower-dev"

Real-world use: clean values supplied through .tfvars files or CI/CD pipeline variables before using them in naming logic.


Split and join strings
-----------------------

These two functions are especially useful for configuration processing.

split()

String → list

    split(",", "dev,stage,prod")


Result:

    ["dev", "stage", "prod"]

For example, a pipeline may pass dev,stage,prod as one string. split() can turn it into a list for further processing.


join()

List → string

    join("-", ["empower", "prod"])


Result:

    "empower-prod"

Extract and format text
------------------------

substr() — extract part of a string

    substr("empower", 0, 3)
    # "emp"


The starting position is zero-based.


format() — build a formatted string

    format("%s-%s", "empower", "prod")
    # "empower-prod"


%s is a string placeholder.


formatlist() — format every element in a list

    formatlist("app-%s", ["dev", "prod"])
    # ["app-dev", "app-prod"]




startswith() and endswith() — check string boundaries

    startswith("empower-prod", "empower")
    # true

    endswith("empower-prod", "prod")
    # true


Both return Boolean values (true or false).


One additional function to know is strrev("abc"), which returns "cba". It reverses the characters in a string.

    
    strrev("abc"), 
    returns "cba". 

It reverses the characters in a string.

All string functions:
---------------------

    upper("empower_dev")
    lower("PRODUCTION")
    title("aws devops engineer")
    trimspace("  dev  ")
    replace("empower_dev", "_", "-")
    split(",", "dev,stage,prod")
    join("-", ["empower", "prod"])
    substr("empower", 0, 3)
    format("%s-%s", "empower", "prod")
    startswith("empower-prod", "empower")


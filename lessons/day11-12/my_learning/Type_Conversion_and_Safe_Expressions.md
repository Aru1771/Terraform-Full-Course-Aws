Type Conversion and Safe Expressions
====================================

# Function 1: tostring()

* Converts a value into a string.

      tostring(8081)

Output: "8081"

* AWS DevOps use case: Convert a numeric application port into text when a configuration or template expects a string.

# Function 2: tonumber()

* Converts a numeric string into a number.

      tonumber("8081")

Output: 8081

* Use case: Read a numeric setting provided as text and use it in a calculation.

For example:

    locals {
      app_port = tonumber("8081")
      next_port = local.app_port + 1
    }

* The result of local.next_port is 8082.

# Function 3: tobool()

* Converts a valid Boolean string into a Boolean value.

      tobool("true")

Output: true

And:

    tobool("false")

Output: false

* Use case: Convert a Boolean setting stored as text into a proper Boolean for conditional logic.

# Function 4: try()

* You learned this on Day 4. Let's review it in today's safe-expression context.

* try() evaluates expressions in order and returns the first one that doesn't produce an error.

      try(tonumber("abc"), 8081)

Output: 8081

* Why? "abc" cannot be converted into a number, so tonumber("abc") produces an error. try() catches it and returns 8081.

* Important: try() catches errors in expressions; it doesn't validate that a value is sensible for your application.

# Function 5: can()

* can() checks whether an expression can be evaluated without an error. It returns true or false.

Example:

    can(tonumber("8081"))
Output: true

But:

    can(tonumber("abc"))
Output: false

* Difference between try() and can():

      Function  	Returns	Purpose
      try()	      The first successful expression's value	Use a fallback when an expression errors
      can()	      true or false	Check whether an expression succeeds

* For example, can() is useful inside Terraform variable validation to check whether a supplied value is valid for a particular operation.

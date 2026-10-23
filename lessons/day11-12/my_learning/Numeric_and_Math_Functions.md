Theory: Numeric and math functions
==================================

# Function 1: max()

* Returns the largest number from the supplied numbers.

      max(10, 25, 15, 8)

Output: 25

* AWS DevOps use case: Choose the largest calculated capacity requirement.

# Function 2: min()

* Returns the smallest number.

      min(10, 25, 15, 8)

Output: 8


* AWS DevOps use case: Find the smallest value among several capacity or threshold settings.

# Function 3: abs()

* Returns the absolute value of a number, removing its negative sign if present.

      abs(-25)

Output: 25

* AWS DevOps use case: Calculate the magnitude of a difference between two numeric values, regardless of its sign.

# Function 4: ceil()

* Rounds a number up to the next whole number.

      ceil(4.2)

Output: 5


Another example:

    ceil(4.0)

Output: 4

* AWS DevOps use case: If a calculation says you need 4.2 units of capacity, round up to 5 whole units.

# Function 5: floor()

* Rounds a number down to the nearest whole number.

      floor(4.9)

Output: 4

* AWS DevOps use case: Calculate how many complete units fit within a capacity limit.

# Function 6: parseint()

* Converts a string representing an integer in a specified number base into a number.

      parseint("1010", 2)

Output: 10

* The 2 means the input string is binary (base 2). For decimal strings, use base 10:

      parseint("250", 10)

Output: 250


* This is useful when numeric configuration arrives as text and you need to interpret it as an integer.

# Function 7: pow()

* Raises a number to a power.

      pow(2, 3)

Output: 8

Because 2^3 = 2 x 2 x 2 = 8.

* AWS DevOps use case: Calculate exponential growth in a capacity or backoff model.

# Function 8: log()

* Calculates the logarithm of a number using a specified base.

      log(100, 10)

Output: 2

Because \(10^2 = 100\).


# Function 9: signum()

* Returns the sign of a number:

      signum(-25) # -1
      signum(0)   # 0
      signum(25)  # 1


* This can help classify whether a numeric difference is negative, zero, or positive.

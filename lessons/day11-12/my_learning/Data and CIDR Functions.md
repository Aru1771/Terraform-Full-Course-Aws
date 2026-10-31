AWS Infrastructure Data and CIDR Functions
==========================================

* These functions are especially useful when designing VPCs, public/private subnets, and network infrastructure in AWS.

What is CIDR?

* CIDR stands for Classless Inter-Domain Routing. It represents an IP address range.

For example:

    10.0.0.0/16

- 10.0.0.0 — network address
- /16 — network prefix length
- This range contains 65,536 IP addresses.

* Imagine you're creating an AWS VPC:

<img width="702" height="367" alt="image" src="https://github.com/user-attachments/assets/a61855a4-6439-4779-a6ab-3a1c86e5903e" />

* Important Terraform CIDR functions

Function 1: cidrsubnet()
------------------------
Purpose: Calculate a smaller subnet inside a larger network.

Syntax:

    cidrsubnet(prefix, newbits, netnum)

- prefix — original network CIDR.
- newbits — number of additional network bits.
- netnum — subnet number to calculate.

Example: cidrsubnet("10.0.0.0/16", 8, 0)
Output: 10.0.0.0/24

Another example: cidrsubnet("10.0.0.0/16", 8, 1)
Output: 10.0.1.0/24

And: cidrsubnet("10.0.0.0/16", 8, 2)
Output: 10.0.2.0/24

Remember: newbits = 8 changes /16 into /24. netnum selects which subnet.

Function 2: cidrhost()
----------------------

* Purpose: Calculate a specific IP address inside a CIDR range.

Syntax:

    cidrhost(prefix, hostnum)
Example: cidrhost("10.0.1.0/24", 5)
Output: 10.0.1.5

More examples: cidrhost("10.0.1.0/24", 10)
# 10.0.1.10

cidrhost("10.0.1.0/24", 20)
# 10.0.1.20

* AWS use case: Calculate a specific IP address for a network appliance or another resource that needs a predictable address.

* Note: AWS reserves five IPv4 addresses in every subnet, so not every calculated address is assignable to an EC2 instance.

Function 3: cidrnetmask()
-------------------------
Purpose: Convert an IPv4 CIDR prefix into its subnet mask.

Example: cidrnetmask("10.0.1.0/24")

Output: 255.255.255.0


Another example: cidrnetmask("10.0.0.0/16")


Output: 255.255.0.0

* AWS use case: Useful when working with network configurations that require a traditional subnet mask instead of CIDR notation.

Function 4: cidrsubnets()
-------------------------

* Purpose: Automatically calculate multiple consecutive subnets from a larger CIDR range.

Syntax: cidrsubnets(prefix, newbits...)
Example: cidrsubnets("10.0.0.0/16", 8, 8, 8, 8)


Output:

    [
      "10.0.0.0/24",
      "10.0.1.0/24",
      "10.0.2.0/24",
      "10.0.3.0/24"
    ]


* Here, Terraform creates four consecutive /24 subnets.

Important production point: Treat the allocation order as stable. Changing or inserting earlier newbits arguments can shift subsequent subnet allocations. For production networks, avoid changing existing allocations casually.
Purpose: Automatically calculate multiple consecutive subnets from a larger CIDR range.

<img width="653" height="317" alt="image" src="https://github.com/user-attachments/assets/237acb0e-177d-46e5-8cd7-83679ef7db54" />

4. Practical task — AWS VPC subnet planning

* Let's create a small Terraform Console exercise for the Empower application.

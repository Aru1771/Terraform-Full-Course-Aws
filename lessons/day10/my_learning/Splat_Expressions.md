Terraform Splat Expressions
------------------------------


1. Theory

A splat expression is used to extract a particular attribute from multiple objects in Terraform.

In simple terms:

    "From all these objects, give me this particular attribute."

The common syntax is:

    resource.resource_name[*].attribute

For example, suppose Terraform creates multiple EC2 instances:

    resource "aws_instance" "web" {
      count = 3
    
      ami           = "ami-xxxxxxxx"
      instance_type = "t2.micro"
    }

Terraform creates:

    aws_instance.web[0]
    aws_instance.web[1]
    aws_instance.web[2]

Each instance has attributes such as:

    id
    public_ip
    private_ip
    availability_zone

If you want the public IP of all three instances, instead of accessing them individually:

    aws_instance.web[0].public_ip
    aws_instance.web[1].public_ip
    aws_instance.web[2].public_ip

you can use:

    aws_instance.web[*].public_ip

Result:

    [
      "3.10.10.11",
      "3.10.10.12",
      "3.10.10.13"
    ]

That's the basic idea of a splat expression.

Basic Syntax
-------------

The general syntax is:

COLLECTION[*].ATTRIBUTE

The important part is:

    [*]

which means:

    For every element in this collection

Splat Expression vs Indexing
------------------------------

Getting one element

    aws_subnet.private[0].id

Means:

    Give me the ID of the first subnet.

Result:

    subnet-111

Getting every element

    aws_subnet.private[*].id

Means:

    Give me the IDs of all subnets.

Result:

    [
      "subnet-111",
      "subnet-222",
      "subnet-333"
    ]

So:

    [0]

    means one specific element
while:

    [*]

    means all elements.



important Limitation
---------------------

This is where your previous for_each knowledge becomes important.
A normal splat expression works naturally with list-like collections, such as resources created with count.

For example:

    aws_instance.web[*].id

works with a resource created using:

    count = 3

But if you have:

    resource "aws_instance" "web" {
      for_each = var.instances
    }

then the resource is represented as a map.
In that situation, you generally use:

    [for instance in aws_instance.web : instance.id]

It means:

    Iterate through every instance in aws_instance.prod and return its id.

rather than relying on the normal splat form.

This is an important distinction:

    count
      ↓
    list/tuple
      ↓
    splat is very natural
    
    for_each
      ↓
    map
      ↓
    for expression is commonly used


But there is an interesting Splat trick for for_each
-----------------------------------------------------

Terraform also supports splatting the values of a map by converting them to a list:

    values(aws_instance.prod)[*].id

Think about it:

    aws_instance.prod
           ↓
         values()
           ↓
    [instance1, instance2]
           ↓
         [*]
           ↓
         .id
           ↓
    ["i-123", "i-456"]


So you could write:

    output "instance_ids" {
      value = values(aws_instance.prod)[*].id
    }

    
We'll practice this distinction because it is very useful in real Terraform code.
Splat expressions work directly with lists, sets, and tuples. for_each creates a map of resources, 
so you normally cannot use the simple resource.*.attribute splat syntax directly on a for_each resource.

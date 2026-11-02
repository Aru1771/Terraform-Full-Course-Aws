<img width="575" height="305" alt="image" src="https://github.com/user-attachments/assets/47bca4a0-c086-47ea-9ebc-b1806ac0e03d" />Hashing and Identity Functions
================================

* Today we'll learn how Terraform generates hashes, checks file integrity, and creates unique identifiers.

* These concepts are useful for AWS DevOps engineers working with:

      - Deployment packages and application artifacts.
      - File change detection.
      - Configuration validation.
      - Resource naming and identity.
      - Secure password hashing concepts.

Theory — What is hashing?
-------------------------

* Imagine you have an application deployment file:

      empower-app.zip

* You want to check whether the file has changed before deploying it.

* A hash function converts input data into a fixed-length string called a hash value.

<img width="575" height="305" alt="image" src="https://github.com/user-attachments/assets/a2593809-3d7c-4d37-9912-9cef16f10be2" />

Important points:

    - The same input produces the same hash.
    - A small input change usually produces a very different hash.
    - Hashing is one-way; you cannot normally recover the original input.
    - Hashing is not encryption.

Terraform hashing functions
---------------------------

# Function 1: md5()

Purpose: Generates an MD5 hash of a string.

Syntax:

    md5("Empower")


Output: a 32-character hexadecimal hash.

Example:

    md5("dev")
    md5("prod")


Both produce different hash values.

Real-world use: Legacy checksum and change-detection workflows. 
MD5 is not suitable for security-sensitive integrity or cryptographic protection because it has known collision weaknesses.


# Function 2: sha1()


Purpose: Generates a SHA-1 hash of a string.

Example:

    sha1("Empower")


Output: a 40-character hexadecimal hash.

Real-world use: Compatibility with older systems. 
SHA-1 is also considered cryptographically weak and should not be selected for new security-sensitive designs.

# Function 3: sha256()

Purpose: Generates a SHA-256 hash of a string.

Example:

    sha256("Empower")


Output: a 64-character hexadecimal hash.

Real-world use: Generate a strong checksum from a string, such as a configuration value or deployment metadata.

For new integrity checks, SHA-256 is generally preferred over MD5 and SHA-1.

# Function 4: filemd5()

Purpose: Calculates the MD5 hash of a file's contents.

Syntax:

    filemd5("application.zip")


For example:

    filemd5("empower-app.zip")


If the file content changes, its hash will usually change too.

AWS DevOps use case: Detect changes in deployment artifacts.

# Function 4: filemd5()

Purpose: Calculates the MD5 hash of a file's contents.

Syntax:

    filemd5("application.zip")


For example:

      filemd5("empower-app.zip")


If the file content changes, its hash will usually change too.

AWS DevOps use case: Detect changes in deployment artifacts.

# Function 5: filesha256()

Purpose: Calculates the SHA-256 hash of a file's contents.

Example:

    filesha256("application.zip")


This is particularly useful for validating deployment packages and configuration files.

<img width="805" height="398" alt="image" src="https://github.com/user-attachments/assets/6fdf6804-1340-4747-826f-02c08c79b79c" />

# Function 6: uuid()

Purpose: Generates a UUID (Universally Unique Identifier).

Example:

    uuid()


Example output:

    550e8400-e29b-41d4-a716-446655440000


Your actual output will be different.

AWS DevOps use case: Generate a unique identifier for a workflow or temporary operation.
Production caution
uuid() is an impure function: its result can change when Terraform evaluates it again. 
Avoid using it directly for persistent AWS resource names or identifiers. 
For a persistent generated UUID, consider the random_uuid resource from the Random provider.

# Function 7: bcrypt()

Purpose: Generates a bcrypt password hash.

Example:

    bcrypt("MyPassword123")


It returns a bcrypt hash rather than the original password.

Real-world use case: Password hashing and compatibility with systems that store bcrypt password hashes.

Important:

    - Bcrypt is designed for password hashing.
    - It uses a salt, so hashing the same password twice can produce different outputs.
    - Do not compare two bcrypt outputs directly to check whether passwords match; use a proper bcrypt verification mechanism.
    - Never put real passwords in Terraform code or state just to demonstrate this function.

<img width="935" height="357" alt="image" src="https://github.com/user-attachments/assets/48c2f25e-a8e9-4949-9e33-7486c13a91bd" />

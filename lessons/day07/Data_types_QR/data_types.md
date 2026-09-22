🏗️ Terraform Complex Data Types — Quick Reference
===================================================

| Data Type  | Stores                      | Same / Different Types | Ordered?                    | Duplicates?                    | Access Method     | Example                   |
| ---------- | --------------------------- | ---------------------- | --------------------------  | ----------------------------   | ----------------- | ------------------------- |
| **List**   | Collection of values        | Same type              | ✅ Yes                      | ✅ Allowed                    | Index `[0]`       | `["dev", "prod"]`         |
| **Map**    | Key-value pairs             | Values same type       | ❌ No positional order      | Keys ❌ / Values ✅           | Key `["dev"]`     | `{dev = "t3.micro"}`      |
| **Set**    | Collection of unique values | Same type              | ❌ No index                 | ❌ Not allowed                 | No index          | `["dev", "prod"]`         |
| **Object** | Named attributes            | Can be different types | Attributes, not positional   | Attribute names unique         | Attribute `.name` | `{name="app", port=8080}` |
| **Tuple**  | Fixed collection of values  | Can be different types | ✅ Yes                      | Depends on defined positions    | Index `[0]`       | `["app", 8080, true]`     |


🔥 Most Important Differences
------------------------------

1️⃣ List vs Set
---------------

|                  | List                   | Set                      |
| ---------------- | ---------------------- | ------------------------ |
| Order            | ✅ Maintains order      | ❌ No positional ordering |
| Duplicate values | ✅ Allowed              | ❌ Removed                |
| Index            | ✅ `[0]`                | ❌ No                     |
| Example          | `["dev","prod","dev"]` | `["dev","prod"]`         |


Remember:

    List = Order matters
    Set = Uniqueness matters


2️⃣ List vs Tuple
------------------

|               | List           | Tuple                         |
| ------------- | -------------- | ----------------------------- |
| Element types | Same type      | Can be different              |
| Size          | Can vary       | Fixed structure               |
| Index         | ✅              | ✅                             |
| Example       | `list(string)` | `tuple([string,number,bool])` |


Remember:

    List = same type
    Tuple = position-specific types


3️⃣ Map vs Object
-----------------

|                 | Map                        | Object                              |
| --------------- | -------------------------- | ----------------------------------- |
| Structure       | Key-value pairs            | Named attributes                    |
| Types           | Values generally same type | Attributes can have different types |
| Keys/attributes | Dynamic keys               | Predefined attributes               |
| Access          | `["key"]`                  | `.attribute`                        |
| Example         | `map(string)`              | `object({...})`                     |


Remember:

    Map = dynamic keys
    Object = predefined attributes

🧠 One-Line Memory Trick
-------------------------

       LIST   → Ordered + duplicates
      MAP    → Key + Value
      SET    → Unique values
      OBJECT → Named attributes + different types
      TUPLE  → Ordered + different types\

⭐ Interview Cheat Sheet
---------------------------

      Need ordered values?
              ↓
            LIST
      
      Need key-value pairs?
              ↓
            MAP
      
      Need unique values?
              ↓
            SET
      
      Need named attributes with different types?
              ↓
           OBJECT
      
      Need ordered values with different types?
              ↓
           TUPLE

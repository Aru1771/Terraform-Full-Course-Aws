🏗️ Terraform Complex Data Types — Quick Reference
===================================================

| Data Type  | Stores                      | Same / Different Types | Ordered?                    | Duplicates?                    | Access Method     | Example                   |
| ---------- | --------------------------- | ---------------------- | --------------------------  | ----------------------------   | ----------------- | ------------------------- |
| **List**   | Collection of values        | Same type              | ✅ Yes                      | ✅ Allowed                    | Index `[0]`       | `["dev", "prod"]`         |
| **Map**    | Key-value pairs             | Values same type       | ❌ No positional order      | Keys ❌ / Values ✅           | Key `["dev"]`     | `{dev = "t3.micro"}`      |
| **Set**    | Collection of unique values | Same type              | ❌ No index                 | ❌ Not allowed                 | No index          | `["dev", "prod"]`         |
| **Object** | Named attributes            | Can be different types | Attributes, not positional   | Attribute names unique         | Attribute `.name` | `{name="app", port=8080}` |
| **Tuple**  | Fixed collection of values  | Can be different types | ✅ Yes                      | Depends on defined positions    | Index `[0]`       | `["app", 8080, true]`     |

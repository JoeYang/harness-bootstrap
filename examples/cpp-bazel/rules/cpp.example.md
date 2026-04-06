---
paths: ["**/*.cpp", "**/*.h", "**/*.cc", "**/*.hpp"]
---

- RAII for all resource management -- no manual new/delete
- `unique_ptr`/`shared_ptr` when heap allocation is needed
- `const` by default: params, member functions, locals
- C++20 minimum; use concepts and ranges where they simplify code
- `#pragma once` over include guards
- `string_view` over `const string&` for read-only params
- Avoid virtual dispatch on hot paths -- use CRTP or compile-time polymorphism
- No exceptions on hot paths -- use error codes or `std::expected`

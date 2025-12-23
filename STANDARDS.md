# Project Technical Specification

## Overview

This document serves as the **primary technical specification** for the project and is intended for **experienced Laravel developers**.

The project is built using the following core technologies:

- **Laravel 12**
- **Laravel Octane**
- **Laravel Sail**
- **Redis**
- **MySQL**

All architectural, implementation, and operational decisions should align with best practices for the tools listed above.

> **Important:**  
> If a topic, behavior, or implementation detail is **not explicitly covered in this documentation**, developers **must strictly follow the official Laravel documentation** and its recommended conventions.
case
---

## Technology Stack

| Component        | Purpose                              |
|------------------|--------------------------------------|
| Laravel 12       | Core application framework           |
| Laravel Octane   | High-performance application runtime |
| Laravel Sail     | Local development environment        |
| Redis            | Caching, queues, and ephemeral data  |
| MySQL            | Relational database                  |

---

## Documentation Structure

This specification is split into multiple focused chapters. Each chapter is authoritative within its scope.

- [Database Specifications](./sections/database.md)
- [Code Structuring Specifications](./sections/code-structure.md)
- [Dates & Time Handling](./sections/dates.md)

Each linked document **must be read and followed** when working within its respective area.

---

## General Principles

- Prefer **clarity over cleverness**
- Follow **Laravel conventions first**, project conventions second
- Avoid premature abstraction
- Code must be production-ready, testable, and maintainable
- Performance considerations are mandatory due to Octane usage

---

## Scope & Authority

This documentation:

- Defines **project-specific rules**
- Overrides personal preferences
- Supplements Laravel documentation
- Evolves over time as the project grows

Any deviation from these specifications **must be explicitly discussed and approved**.

---

## Next Steps

Developers should begin by reviewing:

1. Database Specifications
2. Code Structuring Specifications
3. Dates & Time Handling

Additional chapters will be introduced as the project evolves.

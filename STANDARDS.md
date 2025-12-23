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
- [Testing Specifications](./sections/testing.md)

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

## Code Quality Tools

### Laravel Pint

**Laravel Pint** is used for automated code formatting and style enforcement.

- Enforces PSR-12 coding standards
- Runs automatically on code changes
- Ensures consistent formatting across the codebase
- Fixes formatting issues automatically

All code must pass Pint validation. Run Pint before committing:

```bash
./vendor/bin/pint
```

### Larastan

**Larastan** is used for static analysis to catch type errors and potential bugs before runtime.

- Detects type mismatches and undefined methods
- Identifies potential null pointer issues
- Enforces strict type checking
- Integrates with Laravel framework knowledge

All code must pass Larastan analysis at level 5. Run Larastan regularly during development:

```bash
./vendor/bin/phpstan analyse
```

### Quality Requirements

- Code must pass both Pint and Larastan before being merged
- Pre-commit hooks should run these tools automatically
- No code should be committed with linting or static analysis failures
- Type safety is non-negotiable

---

## Next Steps

Developers should begin by reviewing:

1. Database Specifications
2. Code Structuring Specifications
3. Dates & Time Handling
4. Testing Specifications

Additional chapters will be introduced as the project evolves.

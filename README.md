# GRS Standards

Technical standards for the GRS project.

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Guiding Principles](#guiding-principles)
- [Authority & Scope](#authority--scope)
- [Code Quality Tools](#code-quality-tools)
- [Technology Stack](#technology-stack)
- [Contributing](#contributing)
- [Questions](#questions)

## Overview

Authoritative project standards for GRS, covering:

- **Database design** — Table naming, foreign keys, indexes, and constraints
- **Code structure** — Project organization, naming conventions, and architectural patterns
- **Dates & time handling** — Timezone management and temporal data practices

All GRS developers must follow these specifications.

## Quick Start

1. Skim the principles and quality tools below
2. Then consult the relevant section when changing that area:
   - [Database](./sections/database.md)
   - [Code structure](./sections/code-structure.md)
   - [Dates & time](./sections/dates.md)
   - [Testing](./sections/testing.md)

## Guiding Principles

- Prefer **clarity over cleverness**
- Follow **Laravel conventions first**, project conventions second
- Avoid premature abstraction
- Code must be production-ready, testable, and maintainable
- Always consider Octane performance characteristics

## Authority & Scope

These specifications:
- Define **project-specific rules** that override personal preferences
- Override personal style in favor of consistency
- Supplement Laravel docs rather than restating them
- Evolve as the project grows

Any deviation from these specifications **must be explicitly discussed and approved**.

## Code Quality Tools

### Laravel Pint

Automated code formatting and style enforcement:

- Enforces PSR-12 coding standards
- Runs automatically on code changes
- Ensures consistent formatting across the codebase
- Fixes formatting issues automatically

All code must pass Pint validation:

```bash
./vendor/bin/pint
```

### Larastan

Static analysis for type safety and framework-aware checks:

- Detects type mismatches and undefined methods
- Identifies potential null pointer issues
- Enforces strict type checking
- Integrates with Laravel framework knowledge

All code must pass Larastan analysis at level 5:

```bash
./vendor/bin/phpstan analyse
```

### Quality Requirements

- Code must pass both Pint and Larastan before merge
- Pre-commit hooks should run these tools automatically
- No code should be committed with linting or static analysis failures
- Type safety is non-negotiable

## Technology Stack

- **Laravel 12** — Core framework
- **Laravel Octane** — High-performance runtime
- **Laravel Sail** — Local development environment
- **Redis** — Caching and queues
- **MySQL** — Relational database

## Contributing

When updating these specifications:

1. Make changes to the relevant markdown file in `sections/`
2. Test examples and code snippets thoroughly
3. Create a clear commit message explaining the change
4. Ensure consistency across all documentation

## Questions?

If a topic is not covered in this documentation, refer to the [official Laravel documentation](https://laravel.com/docs) and follow its recommended conventions.

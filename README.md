# GRS Standards

Technical standards for GRS.

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

Authoritative standards covering:

- **Database design** — Table naming, foreign keys, indexes, and constraints
- **Code structure** — Project organization, naming conventions, and architectural patterns
- **Dates & time handling** — Timezone management and temporal data practices
 - **Error handling** — Exceptions, logging, and failure behavior

All GRS developers must follow these specifications.

## Quick Start

1. Skim the principles and tooling below
2. Consult the relevant section when changing that area:
   - [Database](./sections/database.md)
   - [Code structure](./sections/code-structure.md)
   - [Dates & time](./sections/dates.md)
   - [Testing](./sections/testing.md)
   - [Error handling](./sections/error-handling.md)

## Guiding Principles

- Prefer **clarity over cleverness**
- Follow **Laravel conventions first**, project conventions second
- Avoid premature abstraction
- Code must be production-ready, testable, and maintainable
- Always consider Octane performance characteristics

## Authority & Scope

These specifications:
- Define **project-specific rules** over personal preferences
- Prioritize consistency over individual style
- Supplement Laravel docs rather than restating them
- Evolve as the project grows

Any deviation from these specifications **must be explicitly discussed and approved**.

## Code Quality Tools

### Laravel Pint

- Enforces PSR-12 and project formatting
- Should run automatically on changes (e.g. via hooks)
- All code must pass Pint before merge:

```bash
./vendor/bin/pint
```

### Larastan

- Static analysis for type safety and framework-aware checks
- All code must pass level 5 analysis:

```bash
./vendor/bin/phpstan analyse
```

### Quality Requirements

- Code must pass both Pint and Larastan before merge
- Prefer pre-commit hooks to run these tools
- Never commit with linting or analysis failures
- Type safety is non-negotiable

## Technology Stack

- **PHP 8.4** — Runtime
- **Laravel 12** — Core framework
- **Laravel Octane** — High-performance runtime
- **Laravel Sail** — Local development environment
- **Redis** — Caching and queues
- **MySQL** — Relational database

## Contributing

When updating these specifications:

1. Change the relevant file in `sections/`
2. Test examples and code snippets
3. Write a clear commit message
4. Ensure consistency across all documentation

## Questions?

If a topic is not covered in this documentation, refer to the [official Laravel documentation](https://laravel.com/docs) and follow its recommended conventions.

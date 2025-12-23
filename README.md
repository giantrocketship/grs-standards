# GRS Standards

Technical standards for the GRS project.

## Overview

Authoritative project standards for GRS, covering:

- **Database design** — Table naming, foreign keys, indexes, and constraints
- **Code structure** — Project organization, naming conventions, and architectural patterns
- **Dates & time handling** — Timezone management and temporal data practices

All GRS developers must follow these specifications.

## Quick Start

1. Read [STANDARDS.md](./STANDARDS.md)
2. Then consult the relevant section when changing that area:
   - [Database](./sections/database.md)
   - [Code structure](./sections/code-structure.md)
   - [Dates & time](./sections/dates.md)
   - [Testing](./sections/testing.md)

## Key Principles

- **Prefer clarity over cleverness**
- **Follow Laravel conventions first**; these rules layer on top
- **Avoid premature abstraction** — build only what's needed now
- **Production-ready always** — testable, maintainable, and performant

## Technology Stack

- **Laravel 12** — Core framework
- **Laravel Octane** — High-performance runtime
- **Laravel Sail** — Local development environment
- **Redis** — Caching and queues
- **MySQL** — Relational database

## Authority & Scope

These specifications:
- Define **project-specific rules** that override personal preferences
- **Supplement** (but do not repeat) Laravel documentation
- Evolve as the project grows

**Any deviation from these specifications must be explicitly discussed and approved.**

## Contributing

When updating these specifications:

1. Make changes to the relevant markdown file in `sections/`
2. Test examples and code snippets thoroughly
3. Update the main [STANDARDS.md](./STANDARDS.md) if needed
4. Create a clear commit message explaining the change
5. Ensure consistency across all documentation

## Questions?

If a topic is not covered in this documentation, refer to the [official Laravel documentation](https://laravel.com/docs) and follow its recommended conventions.

# GRS Standards

Technical specifications and standards for the GRS project.

## Overview

This repository contains the authoritative technical documentation for the GRS application. It defines standards for:

- **Database design** — Table naming, foreign keys, indexes, and constraints
- **Code structure** — Project organization, naming conventions, and architectural patterns
- **Dates & time handling** — Timezone management and temporal data practices

All developers working on the GRS project must read and follow these specifications.

## Quick Start

1. **Read the main specification**: [STANDARDS.md](./STANDARDS.md)
2. **Review the relevant section**:
   - [Database Specifications](./sections/database.md) — Before writing migrations
   - [Code Structuring Specifications](./sections/code-structure.md) — Before creating new modules or classes
   - [Dates & Time Handling](./sections/dates.md) — When working with dates and timezones

## Key Principles

- **Prefer clarity over cleverness** — Code must be easy to understand
- **Follow Laravel conventions first** — Project conventions supplement, not replace, Laravel's best practices
- **Avoid premature abstraction** — Build only what's needed now
- **Production-ready always** — Code must be testable, maintainable, and performant

## Technology Stack

- **Laravel 12** — Core framework
- **Laravel Octane** — High-performance runtime
- **Laravel Sail** — Local development environment
- **Redis** — Caching and queues
- **MySQL** — Relational database

## Authority & Scope

These specifications:
- Define **project-specific rules** that override personal preferences
- **Supplement** the official Laravel documentation
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
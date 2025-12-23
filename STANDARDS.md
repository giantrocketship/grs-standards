# Core Principles & Standards

Guiding principles and quality standards for GRS. For navigation, see [README.md](./README.md).

---

## Guiding Principles

- Prefer **clarity over cleverness**
- Follow **Laravel conventions first**, project conventions second
- Avoid premature abstraction
- Code must be production-ready, testable, and maintainable
- Always consider Octane performance characteristics

---

## Scope & Authority

This documentation:

- Defines **project-specific rules**
- Overrides personal preferences
- Supplements Laravel docs rather than restating them
- Evolves as the project grows

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

## Detailed Specifications

For specific standards in each area, see:

- [Database Specifications](./sections/database.md)
- [Code Structuring Specifications](./sections/code-structure.md)
- [Dates & Time Handling](./sections/dates.md)
- [Testing Specifications](./sections/testing.md)

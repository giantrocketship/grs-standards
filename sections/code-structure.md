# Code Structuring Specifications

## Overview

Directory structure, file organization, and naming conventions specific to GRS.

---

## Core Principle: Use Artisan Make Commands

**Never create classes by hand.** Use `php artisan make:*` so:

- Paths and namespaces follow Laravel
- Base classes/traits are correct
- Names are consistent across the project

---

## Directory Structure Overview

Only the high-level layout is standardized:

```
app/
├── Casts/
├── Contracts/
├── DTOs/
├── Enums/
├── Exceptions/
├── Http/
│   ├── Controllers/
│   ├── Requests/
│   └── Resources/
├── Jobs/
├── Models/
├── Policies/
└── Services/
```

---

## Module-Based Organization

Group by module under each directory.

Example pattern (Triage):

```
app/
├── DTOs/Triage/TriageData.php
├── Enums/Triage/TriageStatus.php
├── Exceptions/Triage/InvalidTriageStateException.php
├── Http/
│   ├── Controllers/TriageController.php
│   ├── Requests/Triage/StoreTriageRequest.php
│   └── Resources/Triage/TriageResource.php
├── Jobs/Triage/ProcessTriageJob.php
├── Models/Triage/Triage.php
├── Policies/Triage/TriagePolicy.php
└── Services/Triage/
    ├── TriageService.php
    ├── AssignmentService.php
    ├── Contracts/TriageRepositoryContract.php
    └── Traits/HasTriageStatus.php
```

---

## File Naming & Suffixes

### Mandatory Suffixes

Always add appropriate suffixes to class files for clarity:

| File Type | Suffix           | Example                           |
|-----------|------------------|-----------------------------------|
| Controller | `Controller`     | `TriageController.php`            |
| Model | None             | `Triage.php`                      |
| Request | `Request`        | `StoreTriageRequest.php`          |
| Resource | `Resource`       | `TriageResource.php`              |
| Job | `Job`            | `ProcessTriageJob.php`            |
| Cast | `Cast`           | `StatusCast.php`                  |
| Exception | `Exception`      | `InvalidTriageStateException.php` |
| Policy | `Policy`         | `TriagePolicy.php`                |
| Service | `Service`        | `TriageService.php`               |
| Trait | `Trait`          | `HasTriageStatus.php`             |
| Contract/Interface | `Contract`       | `TriageHandlerContract.php`       |
| Enum | `Enum`           | `TriageStatus.php`                |
| DTO | `Data`, `Result` | `TriageData.php`                  |
| Action | `Action`         | `ProcessTriageAction.php`         |

### Exclusions (No Suffix Required)

- **Models** — `Triage.php`, not `TriageModel.php`
- **Enums** — `TriageStatus.php` is fine
- **DTOs** — `TriageData.php` or `TriageResult.php`
- **Actions** — Verb-style names like `ProcessTriage.php`

---

## Detailed Directory Guidelines

### Models (`app/Models/`)

**Guidelines:**
- Use module subdirectories
- Keep models focused on relationships, casts, scopes
- Move business logic to services

### Enums (`app/Enums/`)

```php
// app/Enums/Triage/TriageStatus.php
namespace App\Enums\Triage;

enum TriageStatus: string
{
    case PENDING = 'pending';
    case IN_PROGRESS = 'in_progress';
    case COMPLETED = 'completed';
}
```

**Guidelines:**
- Organize by module
- Use descriptive case names
- Keep enums simple; move complex logic to services

### Casts (`app/Casts/`)

**Guidelines:**
- Use the `Cast` suffix
- Organize by module
- Register via model `$casts`

### DTOs (`app/DTOs/`)

DTOs are always Spatie Laravel Data objects.

**Guidelines:**
- **Always use Spatie Laravel Data**; do not hand-roll DTOs
- Organize by module: `app/DTOs/Triage/`, `app/DTOs/Calendar/`, etc.
- Suffix is optional: `TriageData.php`, `CreateTriageData.php`
- Use attributes for validation
- Use `from()` to create DTOs from requests/models
- Use DTOs at API and service boundaries; no business logic inside

### Jobs (`app/Jobs/`)

**Guidelines:**
- Use the `Job` suffix
- Organize by module
- Keep jobs focused and idempotent

### Exceptions (`app/Exceptions/`)

**Guidelines:**
- Use the `Exception` suffix
- Organize by module
- Provide meaningful messages and context

### Policies (`app/Policies/`)

**Guidelines:**
- Use the `Policy` suffix
- Organize by module
- Always check `account_id` for tenant isolation
- Register in `AuthServiceProvider`

### Services (`app/Services/`)

**Guidelines:**
- Use the `Service` suffix
- Organize by module
- Inject dependencies via constructor
- Single responsibility per service
- Use DTOs for data transfer
- Dispatch jobs for async work

### Contracts/Interfaces (`app/Contracts/` or `app/Services/Module/Contracts/`)

**Guidelines:**
- Use `Interface` suffix: `TriageRepositoryInterface.php`
- Core/global contracts in `app/Contracts/`
- Module-specific contracts in `app/Services/Module/Contracts/`
- Bind interfaces to implementations in service providers

### Traits (`app/Traits/` or `app/Services/Module/Traits/`)

**Guidelines:**
- Use the `Trait` suffix
- Global traits in `app/Traits/`
- Module traits in `app/Services/Module/Traits/`
- Single, focused concern per trait

### Form Requests (`app/Http/Requests/`)

**Guidelines:**
- Use the `Request` suffix
- Organize by module
- Put authorization in `authorize()`
- Use enums with `Rule::enum()`

### Resources (`app/Http/Resources/`)

**Guidelines:**
- Use the `Resource` suffix
- Organize by module
- Use ISO 8601 for dates
- Transform to a consistent API shape

### Controllers (`app/Http/Controllers/`)

**Guidelines:**
- Use the `Controller` suffix
- Organize by module
- Keep controllers thin; delegate to services
- Use type hints and dependency injection
- Use form requests for validation

---

## Error Handling & Failure Modes

### Fail Loudly, Never Silently

Code must fail **loudly and immediately** when something goes wrong.

**Rules:**
- Do not swallow exceptions or return "safe" but invalid values
- Do not add defaults for required data
- Prefer validation and typed constructors over nullable state

---

## Octane-Specific Guidelines

### Avoid Singletons in Service Providers

**Rules:**
- Do not register stateful services as singletons
- Prefer per-request bindings and constructor injection

### Don't Store Request Data on Long-Lived Services

**Rule:** No request-specific state on services that may be reused between requests; pass that data into methods instead.

---

## Summary Checklist

Before committing code:

- [ ] Laravel conventional files created using `php artisan make:*` commands
- [ ] Appropriate suffix added to class names (except models, enums, DTOs)
- [ ] Files organized into module subdirectories where applicable
- [ ] Controllers delegate to services for business logic
- [ ] Controllers return API resources (transformed to json by Laravel automatically)
- [ ] Services use DTOs for data transfer
- [ ] Custom casts in `app/Casts/`
- [ ] Exceptions organized by module in `app/Exceptions/`
- [ ] Policies in `app/Policies/` with account_id checks
- [ ] Jobs in `app/Jobs/` with module organization
- [ ] Contracts defined for key abstractions
- [ ] Code fails loudly — no silent failures or misleading defaults
- [ ] Required values are never defaulted to values that hide failures
- [ ] No singletons used in service providers
- [ ] No request-specific data stored in class properties

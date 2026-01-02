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

General rules:
- Keep module names consistent across layers (DTOs, Enums, Http, Jobs, Models, Policies, Services, etc.)
- Prefer one module folder depth (e.g. `Triage/`), avoid deeper trees unless clearly needed

Example pattern (Triage):

```
app/
├── DTOs/SharedUserData.php              # shared across services
├── Enums/GlobalStatus.php               # shared across services
├── Exceptions/ExternalServiceException.php
├── Http/
│   ├── Controllers/TriageController.php
│   ├── Requests/Triage/StoreTriageRequest.php
│   └── Resources/Triage/TriageResource.php
├── Jobs/NotifyStakeholdersJob.php       # shared across services
├── Models/Triage/Triage.php
├── Policies/SystemWidePolicy.php
└── Services/Triage/
    ├── DTOs/TriageData.php                     # service-specific
    ├── Enums/TriageStatus.php                  # service-specific
    ├── Exceptions/InvalidTriageStateException.php
    ├── Jobs/ProcessTriageJob.php
    ├── Policies/TriagePolicy.php
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

- Use module subdirectories
- Focus models on relationships, casts, scopes
- Move business logic to services

### Enums (`app/Services/<Module>/Enums/` and `app/Enums/`)

```php
// app/Services/Triage/Enums/TriageStatus.php
namespace App\Services\Triage\Enums;

enum TriageStatus: string
{
    case PENDING = 'pending';
    case IN_PROGRESS = 'in_progress';
    case COMPLETED = 'completed';
}
```

**Guidelines:**
- Service-specific enums live under `app/Services/<Module>/Enums/`
- Only enums intentionally shared across multiple services belong in `app/Enums/`
- Organize shared enums by module/domain under `app/Enums/...` if needed
- Use descriptive case names
- Keep enums simple; move complex logic to services

### Casts (`app/Services/<Module>/Casts/` and `app/Casts/`)

- Use the `Cast` suffix
- Service-specific casts live under `app/Services/<Module>/Casts/`
- Only casts intentionally shared across multiple services belong in `app/Casts/`
- Organize shared casts by module/domain under `app/Casts/...` if needed
- Register via model `$casts`

### DTOs (`app/Services/<Module>/DTOs/` and `app/DTOs/`)

DTOs are always Spatie Laravel Data objects.

- **Always use Spatie Laravel Data**; do not hand-roll DTOs
- Service-specific DTOs live under `app/Services/<Module>/DTOs/`
- Only DTOs intentionally shared across multiple services belong in `app/DTOs/`
- Organize shared DTOs by module/domain under `app/DTOs/...` if needed
- Suffix is optional: `TriageData.php`, `CreateTriageData.php`
- Use attributes for validation
- Use `from()` to create DTOs from requests/models
- Use DTOs at API and service boundaries; no business logic inside

### Jobs (`app/Services/<Module>/Jobs/` and `app/Jobs/`)

- Use the `Job` suffix
- Service-specific jobs live under `app/Services/<Module>/Jobs/`
- Only jobs intentionally shared across multiple services belong in `app/Jobs/`
- Organize shared jobs by module/domain under `app/Jobs/...` if needed
- Keep jobs focused and idempotent

### Exceptions (`app/Services/<Module>/Exceptions/` and `app/Exceptions/`)

- Use the `Exception` suffix
- Service-specific exceptions live under `app/Services/<Module>/Exceptions/`
- Only exceptions intentionally shared across multiple services belong in `app/Exceptions/`
- Organize shared exceptions by module/domain under `app/Exceptions/...` if needed
- Provide meaningful messages and context

### Policies (`app/Services/<Module>/Policies/` and `app/Policies/`)

- Use the `Policy` suffix
- Service-specific policies live under `app/Services/<Module>/Policies/`
- Only policies intentionally shared across multiple services belong in `app/Policies/`
- Organize shared policies by module/domain under `app/Policies/...` if needed
- Always check `account_id` for tenant isolation
- Register in `AuthServiceProvider`

### Services (`app/Services/`)

- Use the `Service` suffix
- Organize by module
- Inject dependencies via constructor
- Single responsibility per service
- Use DTOs for data transfer
- Dispatch jobs for async work

### Contracts/Interfaces (`app/Services/<Module>/Contracts/` and `app/Contracts/`)

- Use `Interface` suffix: `TriageRepositoryInterface.php`
- Service-specific contracts live under `app/Services/<Module>/Contracts/`
- Only contracts intentionally shared across multiple services go in `app/Contracts/`
- Bind interfaces to implementations in service providers

### Traits (`app/Services/<Module>/Traits/` and `app/Traits/`)

- Use the `Trait` suffix
- Service-specific traits live under `app/Services/<Module>/Traits/`
- Only traits intentionally shared across multiple services go in `app/Traits/`
- Single, focused concern per trait

### Form Requests (`app/Http/Requests/`)

- Use the `Request` suffix
- Organize by module
- Put authorization in `authorize()`
- Use enums with `Rule::enum()`

### Resources (`app/Http/Resources/`)

- Use the `Resource` suffix
- Organize by module
- Use ISO 8601 for dates
- Transform to a consistent API shape

### Controllers (`app/Http/Controllers/`)

- Use the `Controller` suffix
- Organize by module
- Keep controllers thin; delegate to services
- Use type hints and dependency injection
- Use form requests for validation

---

## Error Handling & Failure Modes

See [Error Handling Specifications](./error-handling.md) for full rules.

In this context:
- Do not swallow exceptions or return "safe" but invalid values
- Do not add defaults for required data just to avoid errors
- Prefer validation, typed constructors, and DTOs over nullable state

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

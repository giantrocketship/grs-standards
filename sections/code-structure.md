# Code Structuring Specifications

## Overview

This document defines the directory structure, file organization, and naming conventions for the application. Following these conventions ensures consistency, maintainability, and predictability across the codebase.

---

## Core Principle: Use Artisan Make Commands

**Never create files by hand.** Always use Laravel's `php artisan make:*` commands to scaffold new classes. This ensures:

- Correct file structure and namespacing
- Proper base classes and traits
- Consistency across the project
- Automatic service provider registration (where applicable)

### Available Make Commands:

Discover all available make commands and their options:

```bash
php artisan make --help
```

---

## Directory Structure Overview

```
app/
├── Casts/              # Custom model casts
├── Contracts/          # Core interfaces and contracts
├── DTO/               # Data Transfer Objects
├── Enums/              # Application enums
├── Exceptions/         # Custom exceptions
├── Http/
│   ├── Controllers/
│   ├── Requests/       # Form requests
│   └── Resources/      # API resources
├── Jobs/               # Queued jobs
├── Models/             # Eloquent models
├── Policies/           # Authorization policies
└──Services/           # Business logic services
```

---

## Module-Based Organization

For features or modules, organize related files into nested folders under their respective directories.

### Example: Triage Module

```
app/
├── Casts/
│   └── Triage/
│       └── StatusCast.php
├── DTO/
│   └── Triage/
│       ├── TriageData.php
│       └── AssignmentResult.php
├── Enums/
│   └── Triage/
│       ├── TriageStatus.php
│       └── PriorityEnum.php
├── Exceptions/
│   └── Triage/
│       └── InvalidTriageStateException.php
├── Http/
│   ├── Controllers/
│   │   └── TriageController.php
│   ├── Requests/
│   │   └── Triage/
│   │       ├── StoreTriageRequest.php
│   │       └── UpdateTriageRequest.php
│   └── Resources/
│       └── Triage/
│           └── TriageResource.php
├── Jobs/
│   └── Triage/
│       ├── ProcessTriageJob.php
│       └── SendTriageNotificationJob.php
├── Models/
│   └── Triage/
│       ├── Triage.php
│       └── TriageAssignment.php
├── Policies/
│   └── Triage/
│       └── TriagePolicy.php
└── Services/
    └── Triage/
        ├── TriageService.php
        ├── AssignmentService.php
        ├── Contracts/
        │   └── TriageRepositoryContract.php
        └── Traits/
            └── HasTriageStatus.php
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

- **Models** — Use simple name: `Triage.php`, not `TriageModel.php`
- **Enums** — Can be simple name: `TriageStatus.php`
- **DTOs** — Use simple name or suffixes: `TriageData.php` or `TriageResult.php`
- **Actions** — Use action verb if readable: `ProcessTriage.php`

---

## Detailed Directory Guidelines

### Models (`app/Models/`)

Store Eloquent models here, organized by module.

**Guidelines:**
- Use module subdirectories for feature-specific models
- Keep model code focused on relationships and scopes
- Move complex business logic to services

### Enums (`app/Enums/`)

Store application-level enums (never database enums).

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

Custom model attribute casts.

**Guidelines:**
- Use the `Cast` suffix
- Organize by module
- Register in model via `$casts` property

### DTOs (`app/DTOs/`)

Data Transfer Objects using **Spatie Laravel Data** for type-safe, validated data structures.

**Never build custom DTO classes.** Always use Spatie Laravel Data for consistency, validation, transformation, and casting.

#### Documentation & Examples

For comprehensive documentation on using Spatie Laravel Data, including basic definitions, validation, enums, casting, creating from requests/models, collections, transformations, and nested DTOs, refer to the official documentation:

**[Spatie Laravel Data Documentation](https://spatie.be/docs/laravel-data/v4/introduction)**

Key features include:
- Type-safe property declarations
- Built-in validation with attributes
- Automatic enum and cast handling
- Creating DTOs from requests, models, and arrays
- Collections and transformations
- Nested and complex data structures

**Guidelines:**
- **Always use Spatie Laravel Data** — never build custom DTO classes
- Organize by module: `app/DTOs/Triage/`, `app/DTOs/Calendar/`, etc.
- Suffix is optional: `TriageData.php` or `CreateTriageData.php`
- Use validation attributes for request validation
- Use `from()` to create DTOs from requests/models
- Leverage automatic casting with enums and custom casts
- Use for API requests/responses and service layer
- Keep DTOs focused on data structure, not business logic

### Jobs (`app/Jobs/`)

Queued jobs for async processing.

```bash
php artisan make:job Triage/ProcessTriageJob
```

```php
// app/Jobs/Triage/ProcessTriageJob.php
namespace App\Jobs\Triage;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessTriageJob implements ShouldQueue
{
    use Queueable;

    public function __construct(private int $triageId) {}

    public function handle(): void
    {
        // Job logic
    }
}
```

**Guidelines:**
- Use the `Job` suffix
- Organize by module
- Keep jobs focused and idempotent

### Exceptions (`app/Exceptions/`)

Custom exception classes.

**Guidelines:**
- Use the `Exception` suffix
- Organize by module
- Provide meaningful error messages and context

### Policies (`app/Policies/`)

Authorization policies for models.

**Guidelines:**
- Use the `Policy` suffix
- Organize by module
- Always check `account_id` for tenant isolation
- Register in `AuthServiceProvider`

### Services (`app/Services/`)

Business logic layer for complex operations.

**Guidelines:**
- Use the `Service` suffix
- Organize by module
- Inject dependencies via constructor
- Keep services focused on a single responsibility
- Use DTOs for data transfers
- Dispatch jobs for async work

### Contracts/Interfaces (`app/Contracts/` or `app/Services/Module/Contracts/`)

Define contracts for abstraction and testing.

**Guidelines:**
- Use `Interface` suffix for clarity: `TriageRepositoryInterface.php`
- Core/global contracts in `app/Contracts/`
- Module-specific contracts in `app/Services/Module/Contracts/`
- Bind interfaces to implementations in service providers

### Traits (`app/Traits/` or `app/Services/Module/Traits/`)

Reusable functionality across classes.

```php
// app/Services/Triage/Traits/HasTriageStatus.php
namespace App\Services\Triage\Traits;

use App\Enums\Triage\TriageStatus;

trait HasTriageStatus
{
    public function isPending(): bool
    {
        return $this->status === TriageStatus::PENDING;
    }

    public function isCompleted(): bool
    {
        return $this->status === TriageStatus::COMPLETED;
    }
}
```

**Guidelines:**
- Use the `Trait` suffix
- Global traits in `app/Traits/`
- Module-specific traits in `app/Services/Module/Traits/`
- Keep traits focused on a single concern

### Form Requests (`app/Http/Requests/`)

Validation and authorization for requests.

```bash
php artisan make:request Triage/StoreTriageRequest
```

**Guidelines:**
- Use the `Request` suffix
- Organize by module
- Include authorization logic in `authorize()` method
- Use enums with `Rule::enum()` for validation

### Resources (`app/Http/Resources/`)

API response transformation.

```bash
php artisan make:resource Triage/TriageResource
```

**Guidelines:**
- Use the `Resource` suffix
- Organize by module
- Always use ISO 8601 for dates
- Transform data for consistent API responses

### Controllers (`app/Http/Controllers/`)

HTTP request handling.

```bash
php artisan make:controller Triage/TriageController --resource
```

**Guidelines:**
- Use the `Controller` suffix
- Organize by module
- Keep controllers thin; delegate to services
- Use type hints and dependency injection
- Use form requests for validation

---

## Error Handling & Failure Modes

### Fail Loudly, Never Silently

Code must fail **loudly and immediately** when something goes wrong. Silent failures lead to subtle bugs and data corruption.

**Rule:** Never suppress errors or default values that could mask problems.

**Bad Examples:**
```php
// ❌ Silent failure: Swallows exception
try {
    $user = User::findOrFail($id);
} catch (ModelNotFoundException $e) {
    $user = null; // Silently returns null
}

// ❌ Silent default: Could hide bugs
$status = $data['status'] ?? 'pending'; // What if status is required?

// ❌ No validation: Fails silently later
$this->userId = $input['user_id']; // What if it's missing or invalid?
```

**Good Examples:**
```php
// ✅ Fail loudly: Throws exception immediately
$user = User::findOrFail($id); // Throws ModelNotFoundException

// ✅ Explicit validation: Requires intent
throw_if(! isset($data['status']), InvalidArgumentException::class);

// ✅ Type validation: Catches mismatches early
public function setUserId(int $id): void
{
    // Type hint forces correct input
    $this->userId = $id;
}
```

### Never Default Values That Hide Failures

**Rule:** Never provide default values for required data if they could allow code to continue in an invalid state.

**Bad Examples:**
```php
// ❌ Missing account_id defaults to 0 (invalid)
$accountId = $request->input('account_id', 0);

// ❌ Empty string defaults that cause bugs later
$email = $request->input('email', '');

// ❌ Silent null defaults for required fields
$name = $data['name'] ?? '';
```

**Good Examples:**
```php
// ✅ Fail if required field is missing
$accountId = $request->validate(['account_id' => 'required|integer']);

// ✅ Use exceptions for required data
$email = $request->input('email')
    ?? throw new InvalidArgumentException('Email is required');

// ✅ Type checking prevents silent failures
public function __construct(private int $accountId, private string $email)
{
    // Constructor guarantees valid data
}
```

---

## Octane-Specific Guidelines

### Avoid Singletons in Service Providers

Due to Octane's request isolation model, never use singletons that persist state across requests.

```php
// ❌ WRONG: Singleton persists state across requests
app()->singleton(CacheService::class, function () {
    return new CacheService();
});

// ✅ CORRECT: Bind without singleton
app()->bind(CacheService::class, function () {
    return new CacheService();
});

// ✅ CORRECT: Use dependency injection instead
public function __construct(private CacheService $cache) {}
```

### Don't Store Data in Class Properties

Never store request-specific data in class properties that persist.

```php
// ❌ WRONG: Data persists across requests in Octane
class UserService
{
    private $userId; // Wrong!

    public function setUserId($id)
    {
        $this->userId = $id;
    }
}

// ✅ CORRECT: Pass data as parameters
class UserService
{
    public function getUserData($userId)
    {
        // Logic
    }
}

// ✅ CORRECT: Use constructor injection for dependencies
class UserService
{
    public function __construct(private UserRepository $repository) {}
}
```

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

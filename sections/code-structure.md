# Code Structuring Specifications

## Overview

Directory structure, file organization, and naming conventions specific to GRS.

---

## Core Principle: Use Artisan Make Commands

Use `php artisan make:*` to create files and follow Laravel conventions and expected structure. Ensure files land in the correct folder, or move them to the correct folder after default creation.

- Paths and namespaces follow Laravel
- Base classes/traits are correct
- Names are consistent across the project

---

## Directory Structure Overview

High-level layout is standardized:

```
app/
├── Actions/
├── Attributes/
├── Casts/
├── Console/Commands/
├── DTOs/
├── Enums/
├── Exceptions/
├── Filament/
├── Http/
│   ├── Controllers/
│   ├── Requests/
│   └── Resources/
├── Jobs/
├── Models/
│   ├── [FeatureName]/
│   │   ├── FeatureModelA.php
│   │   └── FeatureModelB.php
│   ├── Account.php
│   └── User.php
├── Observers/
├── Policies/
├── Providers/
└── Services/
```

---

## Module-Based Organization

While developing a feature, you must follow the model developing logic: create a folder named after the feature inside `app/Services/`, and all files dedicated to the feature must live within that folder.

General rules:
- Keep module names consistent across layers (DTOs, Enums, Exceptions, Jobs, Services, etc.)
- Prefer one module folder depth (e.g. `Triage/`), avoid deeper trees unless clearly needed

Example pattern:

```
app/
├── DTOs/SharedUserData.php                 # shared across services
├── Enums/GlobalStatus.php                  # shared across services
├── Exceptions/ExternalServiceException.php
├── Http/                                   # shared across services
│   ├── Controllers/SharedController.php
│   ├── Requests/SharedRequest.php
│   └── Resources/SharedResource.php
│ 
├── Jobs/NotifyStakeholdersJob.php          # shared across services
├── Models/
│   ├──[FeatureName]/                       # service-specific
│   │   └── ModelA.php
│   └── User.php                            # shared across services
│ 
├── Policies/        
│   ├──[FeatureName]/                       # service-specific
│   │   └── PolicyForModelA.php
│   └── UserPolicy.php                      # shared across services
│ 
├── Providers/FeatureNameProvider.php       # service-specific
│
└── Services/[FeatureName]/                 # service-specific files
    ├── Console/Commands/CommandName.php 
    ├── Contracts/SomeInterface.php
    ├── DTOs/SomeData.php                     
    ├── Enums/FeatureNameStatus.php          
    ├── Exceptions/InvalidFeatureNameStateException.php
    ├── Http/
    │   ├── Controllers/FeatureController.php
    │   ├── Requests/FeatureRequest.php
    │   └── Resources/FeatureResource.php
    ├── Jobs/ProcessFeatureNameJob.php
    ├── Modules/                            # helper or additional services
    │   ├── LockService.php
    │   └── AssignmentService.php
    ├── Plugins/
    │   └── FeatureNamePlugin.php
    ├── Helpers/
    │   └── FeatureNameHelper.php
    ├── Utils/
    │   └── FeatureNameFormatter.php
    ├── Traits/HasFeatureNameStatus.php
    └── FeatureNameService.php              # main service class
```

---

Note: Commands must live in `app/Services/[FeatureName]/Console/Commands/`.

---

```
config/
├── feature_name.php
│
database/
├── factories/
│   └── [FeatureName]/
│       └── FeatureNameFactory.php
├── migrations/
│   └── 2024_01_01_000000_create_feature_name_table.php
└── seeders/
    └── FeatureNameSeeder.php
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
| Enum | None or `Enum`   | `TriageStatus.php`                |
| DTO | `Data`, `Result` | `TriageData.php`                  |
| Action | `Action`         | `ProcessTriageAction.php`         |

Note: Custom folders (e.g., `Plugins/`) must use matching suffixes like `Plugin`.

### Exclusions (No Suffix Required)

- **Models** — `Triage.php`, not `TriageModel.php`
- **Enums** — `TriageStatus.php` is fine
- **DTOs** — `TriageData.php` or `TriageResult.php`
- **Actions** — Verb-style names like `ProcessTriage.php`

---

## Detailed Directory Guidelines

### Models (`app/Models/`)

**Directory Structure:**
- All models live in `app/Models/`
- Feature-specific models live in dedicated subdirectories: `app/Models/FeatureName/`
- Example: `app/Models/Triage/Triage.php`, `app/Models/UnifiedCalendar/WorkSchedule.php`

**Requirements:**
- Every model **must have a `$fillable` array** listing all mass-assignable columns (except `id`)
- Every model **must have a corresponding factory** in `database/factories/FeatureName/ModelNameFactory.php`
- Every model **must cast columns** appropriately:
  - Boolean columns: `'is_active' => 'boolean'`
  - Enum columns: `'status' => StatusEnum::class`
  - Date/datetime columns: `'created_at' => 'datetime'`, `'scheduled_on' => 'date'`
  - JSON columns: `'config' => 'array'`
- Focus models on relationships, casts, and scopes
- Move business logic to services

**Example:**
```php
// app/Models/UnifiedCalendar/WorkSchedule.php
class WorkSchedule extends Model
{
    use HasFactory;

    protected $table = 'ucal_work_schedules'; # only if not using default

    protected $fillable = [
        'account_id',
        'user_id',
        'schedule_layer_id',
        'weekday',
        'starts_time',
        'ends_time',
        'title',
        'notes',
        'created_at',
        'updated_at',
    ];

    protected $casts = [
        'weekday' => 'integer',
        'starts_time' => 'string',
        'ends_time' => 'string',
    ];

    // relationships, scopes, etc.
}
```

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

## Database Access Patterns

**Golden Rule: Use Eloquent Models, not `DB::table()`**

The `DB` facade should **only** be used in:
- Migration files (`up()` and `down()` methods)
- Seeder files (when inserting/updating in bulk)
- `DB::transaction()` for transaction management (allowed everywhere)

### ❌ Forbidden Patterns

**Never use `DB::table()` for queries:**

```php
// WRONG: Direct table access in services/controllers
$users = DB::table('users')
    ->where('account_id', $accountId)
    ->get();

$defaults = DB::table('ucal_account_layer_defaults')
    ->where('account_id', $accountId)
    ->get();

// WRONG: Direct insert/update outside migrations
DB::table('users')->insert(['name' => 'John']);
DB::table('orders')->where('id', $id)->update(['status' => 'shipped']);
```

### ✅ Correct Patterns

**Use Eloquent Models for all queries:**

```php
// CORRECT: Use models
$users = User::where('account_id', $accountId)->get();

$defaults = AccountLayerDefault::where('account_id', $accountId)->get();

$user = User::create(['name' => 'John']);

$order = Order::find($id);
$order->update(['status' => 'shipped']);
```

**Use transactions with `DB::transaction()`:**

```php
// CORRECT: Use DB::transaction() for transaction management
DB::transaction(function () {
    $order = Order::create([/* ... */]);
    $order->items()->create([/* ... */]);
    $order->calculateTotals();
});
```

### Why This Matters

- **Type safety**: Models provide IDE autocomplete and static analysis
- **Relationships**: Models enable lazy/eager loading and relationship access
- **Casts**: Models automatically cast columns (boolean, date, enum, etc.)
- **Scopes**: Models support local/global scopes for query reusability
- **Events**: Models support lifecycle hooks (`created`, `updating`, etc.)
- **Consistency**: Laravel convention is models, not raw queries

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

Note: Models that belong to a feature must live in `app/Models/[FeatureName]/`. Never place a `Models/` folder inside `app/Services/[FeatureName]/`.

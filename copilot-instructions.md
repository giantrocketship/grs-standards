# GRS Copilot Instructions

> Auto-generated from grs-standards. Do not edit by hand; update the source markdown instead.

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
   - [Dev/Docker setup](./sections/dev-setup.md)
   - [Dates & time](./sections/dates.md)
   - [Testing](./sections/testing.md)
   - [Error handling](./sections/error-handling.md)

## Guiding Principles

- Prefer **clarity over cleverness**
- Follow **Laravel conventions first**, project conventions second
- Avoid premature abstraction
- Code must be production-ready, testable, and maintainable
- Always consider Octane performance characteristics
 - Place **service-specific classes** (DTOs, contracts, actions, jobs, etc.)
    under `app/Services/<ServiceName>/...`; only **shared, cross-service classes**
    belong in roots like `app/DTOs`, `app/Contracts`, etc.

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

---

# Code Structuring Specifications

## Overview

Directory structure, file organization, and naming conventions specific to GRS.

---

## Core Principle: Use Artisan Make Commands

Use `php artisan make:*` to create files and follow Laravel conventions and expected structure. Ensure files land in the correct folder, or move them to the correct folder after default creation.

- Paths and namespaces follow Laravel
- Base classes/traits are correct
- Names are consistent across the project
- Never place two classes in a single file
- Prefer passing model instances instead of scalar IDs (e.g., $model->id) into methods to avoid extra database queries

---

## Directory Structure Overview

> `<Module>` stands for `[FeatureName]` in all examples below

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
│   ├── <Module>/
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
- Naming convention example (UnifiedCalendar): use `UnifiedCalendar/` for module folders, `UnifiedCalendarService.php` for the main service, table names prefixed with `unified_calendar_`, and models either prefixed with `UnifiedCalendar` or set `protected $table` explicitly

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
│   ├── <Module>/                       # service-specific
│   │   └── ModelA.php
│   └── User.php                            # shared across services
│ 
├── Policies/        
│   ├── <Module>/                       # service-specific
│   │   └── PolicyForModelA.php
│   └── UserPolicy.php                      # shared across services
│ 
├── Providers/FeatureNameProvider.php       # service-specific
│
└── Services/<Module>/                 # service-specific files
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

Note: Commands must live in `app/Services/<Module>/Console/Commands/`.

---

```
config/
├── feature_name.php
│
database/
├── factories/
│   └── <Module>/
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
| Action | None             | `ProcessTriage.php`               |
| Attribute | `Attribute`    | `HasTriageAttribute.php`          |
| Cast | `Cast`           | `StatusCast.php`                  |
| Command | `Command`       | `SyncTriageCommand.php`           |
| Contract/Interface | `Contract`       | `TriageHandlerContract.php`       |
| Controller | `Controller`     | `TriageController.php`            |
| DTO | `Data`, `Result` | `TriageData.php`                  |
| Enum | None or `Enum`   | `TriageStatus.php`                |
| Exception | `Exception`      | `InvalidTriageStateException.php` |
| Factory | `Factory`       | `TriageFactory.php`               |
| Job | `Job`            | `ProcessTriageJob.php`            |
| Model | None             | `Triage.php`                      |
| Observer | `Observer`      | `TriageObserver.php`              |
| Policy | `Policy`         | `TriagePolicy.php`                |
| Provider | `ServiceProvider` | `TriageServiceProvider.php`    |
| Request | `Request`        | `StoreTriageRequest.php`          |
| Resource | `Resource`       | `TriageResource.php`              |
| Seeder | `Seeder`        | `TriageSeeder.php`                |
| Service | `Service`        | `TriageService.php`               |
| Trait | None             | `HasTriageStatus.php`             |

Note: Custom folders (e.g., `Plugins/`) must use matching suffixes like `Plugin`.

### Exclusions (No Suffix Required)

- **Models** — `Triage.php`, not `TriageModel.php`
- **Enums** — `TriageStatus.php` is fine
- **DTOs** — `TriageData.php` or `TriageResult.php`
- **Actions** — Verb-style names like `ProcessTriage.php`
- **Traits** — `HasTriageStatus.php`, not `HasTriageStatusTrait.php`

---

## Detailed Directory Guidelines

### Models (`app/Models/<Module>`)

**Directory Structure:**

Models that belong to a feature must live in `app/Models/<Module>/`. Never place a `Models/` folder inside `app/Services/<Module>/`.

> Example: `app/Models/Triage/Triage.php`, `app/Models/UnifiedCalendar/WorkSchedule.php`

**Requirements:**
- Every model **must have a `$fillable` array** listing all mass-assignable columns (except `id`)
- Every model **must have a corresponding factory** in `database/factories/FeatureName/ModelNameFactory.php`
- Every model **must cast columns** appropriately:
  - Boolean columns: `'is_active' => 'boolean'`
  - Enum columns: `'status' => StatusEnum::class`
  - Date/datetime columns: `'created_at' => 'datetime'`, `'scheduled_on' => 'date'`
  - JSON columns: `'config' => 'array'`
- Focus models on relationships, casts, and scopes

> Adding functions to models that perform logic is forbidden; put that code in a dedicated service file

**Example:**
```php
// app/Models/UnifiedCalendar/WorkSchedule.php
class WorkSchedule extends Model
{
    use HasFactory;

    protected $table = 'ucal_work_schedules'; # only if model name is different from table name

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

### Console Commands (`app/Services/<Module>/Console/Commands/`)

- Commands live under `app/Services/<Module>/Console/Commands/`
- Use the `Command` suffix
- Command signatures use lowercase, kebab-case segments separated by `:` and follow the pattern `<domain>:<feature>:<action>` (use the shortest form that fits, e.g., two segments when a feature domain is sufficient)
- Options must be kebab-case, explicit, and include descriptions and defaults where applicable
- Example signature (multi-option):
```php
protected $signature = 'helpdesk:discovery:changes
    {--integration-id= : The integration ID}
    {--tag= : Find integration by tag instead of ID}
    {--entity= : Filter by entity type (ticket, company)}
    {--type= : Filter by change type (field_added, field_deprecated, etc.)}
    {--since=7 days ago : Show changes since date (default: 7 days ago)}
    {--limit=50 : Maximum changes to show}';
```
- Example signature (single option):
```php
protected $signature = 'ticket-assign:explain {--decision= : Decision ID}';
```

### Contracts/Interfaces (`app/Services/<Module>/Contracts/`)

- Use `Contract` suffix: `TriageRepositoryContract.php`
- Service-specific contracts live under `app/Services/<Module>/Contracts/`
- Bind interfaces to implementations in service providers
- Avoid using singletons. Use them only if confident no issues or memory leaks caused to Laravel Octane.


### DTOs (`app/Services/<Module>/DTOs/`)

DTOs are always Spatie Laravel Data objects.

- **Always use Spatie Laravel Data**; do not hand-roll DTOs
- Refer to the Spatie documentation for conventions and usage rules
- Service-specific DTOs live under `app/Services/<Module>/DTOs/`
- Only DTOs intentionally shared across multiple services can be stored in `app/DTOs/`
- Suffix is optional: `TriageData.php`, `CreateTriageData.php`
- Use attributes for validation
- DTO fields must use camelCase
- Use `from()` to create DTOs from requests/models
- Use DTOs at API and service boundaries; no business logic inside

**Example:**
```php
class SongData extends Data
{
    public function __construct(
        public string $title,
        public string $artist,
        #[MapInputName('release_date')]
        public string $releaseDate,
        public DateTime $date,
        public Format $format,
    ) {
    }
}
```

### Enums (`app/Services/<Module>/Enums/`)

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
- Only enums intentionally shared across multiple services can be stored in `app/Enums/`
- Enum names must be uppercase
- Use descriptive case names
- Enums can have helper methods, but move complex logic to services

### Exceptions (`app/Services/<Module>/Exceptions/`)

- Use the `Exception` suffix
- Service-specific exceptions live under `app/Services/<Module>/Exceptions/`
- Only exceptions intentionally shared across multiple services belong in `app/Exceptions/`
- Organize shared exceptions by module/domain under `app/Exceptions/...` if needed
- Provide meaningful messages and context

### Jobs (`app/Services/<Module>/Jobs/`)

- Use the `Job` suffix
- Service-specific jobs live under `app/Services/<Module>/Jobs/`
- Keep jobs focused and idempotent

### Services (`app/Services/<Module>/`)

- Use the `Service` suffix
- Organize by module name
- Core service must live in `app/Services/<Module>/FeatureNameService.php`
- Helper/auxiliary services live in `app/Services/<Module>/Modules/`
- Inject dependencies via constructor
- Single responsibility per service
- Use DTOs for data transfer
- Dispatch jobs for async work
- Never store state/data on service classes (Laravel Octane reuses instances)

### Traits (`app/Services/<Module>/Traits/`)

- Trait names follow Laravel conventions (e.g., `HasSlug`, `Sluggable`);
- Service-specific traits live under `app/Services/<Module>/Traits/`
- Only traits intentionally shared across multiple services go in `app/Traits/`
- Model traits live in `app/Models/Traits/<Module>/MyTrait.php`
- Single, focused concern per trait

---

### Database Factories (`database/factories/`)

- Use the `Factory` suffix
- Feature-specific factories live under `database/factories/<Module>/`
- Use `fake()` instead of `$this->faker`
- Prefer factory states for variants (e.g., integration-specific configurations)
- Keep factories deterministic and focused on defaults
```php
public function definition(): array
{
    return [
        'uuid' => Uuid::uuid4()->toString(),
        'name' => fake()->company(),
        'psa_type' => fake()->randomElement(PsaType::cases()),
    ];
}

public function connectWise(): static
{
    return $this->state(fn () => [
        'psa_type' => PsaType::ConnectWise,
        'psa_creds' => [
            'company_id' => config('services.connectwise.test.company_id'),
        ],
    ]);
}
```

### Database Migrations (`database/migrations/`)

- Prefer a single migration file per feature
- Table names must be prefixed with the feature name in `snake_case`
- Use Laravel schema conventions (`->nullable()`, `->index()`, `->default()`)
- Use `->timestamp()` instead of `->datetime()`
- If the feature has logical separation or too many tables, split into multiple migrations
- While the feature is unmerged, edit the initial migration file
- After the feature is merged to `main`, use new migrations file for tables changes

### Database Seeders (`database/seeders/`)

- Use the `Seeder` suffix
- Feature-specific seeders live under `database/seeders/<Module>Seeder.php`
- Prefer a single seeder file per feature
- If seeding is large or logically distinct, split into multiple files prefixed with `FeatureName`
- Seeders should be idempotent

---

Less common

### Actions (`app/Services/<Module>/Actions/` and `app/Actions/`)

- Name actions as verbs without the `Action` suffix (Laravel-style single-action classes), e.g. `ApprovePayment.php`, `SyncCalendar.php`
- Service-specific actions live under `app/Services/<Module>/Actions/`; shared actions can be in `app/Actions/`
- Use actions to encapsulate a single, reusable workflow invoked by controllers, jobs, or commands

### Casts (`app/Services/<Module>/Casts/`)

- Use the `Cast` suffix
- Service-specific casts live under `app/Services/<Module>/Casts/`
- Only casts intentionally shared across multiple services can be stored in `app/Casts/`
- Organize shared casts by module/domain under `app/Casts/...` if needed
- Register via model `$casts`

### HTTP (`app/Services/<Module>/Http/`)

- Organize module-specific controllers, requests, and resources under `app/Services/<Module>/Http/`
- Shared HTTP files belong in `app/Http/`
- Keep module HTTP layers aligned with service boundaries

### Form Requests (`app/Services/<Module>/Http/Requests/`)

- Use the `Request` suffix
- Organize by module
- Use policies and gates, never put authorization in `authorize()`
- Use enums with `Rule::enum()`

### Resources (`app/Services/<Module>/Http/Resources/`)

- Use the `Resource` suffix
- Organize by module
- Use snake_case for JSON keys
- Use ISO 8601 for dates
- Transform to a consistent API shape

### Controllers (`app/Services/<Module>/Http/Controllers/`)

- Use the `Controller` suffix
- Organize by module
- Keep controllers thin; delegate to services
- Use type hints and dependency injection
- Use form requests for validation
- Always return a `Response` object (or `response()->json()` if needed)

### Observers (`app/Observers/<Module>/`)

- Observers mirror model nesting: if a model lives in `app/Models/<Module>/`, the observer must live in `app/Observers/<Module>/`
- Use the `Observer` suffix
- Keep observers focused and side-effect-aware
- Use Attribute on the model file to register the observer
```php
use Illuminate\Database\Eloquent\Attributes\ObservedBy;

#[ObservedBy([WorkScheduleObserver::class])]
class WorkSchedule extends Model
{
}
```

### Policies (`app/Policies/<Module>/`)

- Policies mirror model nesting: if a model lives in `app/Models/<Module>/`, the policy must live in `app/Policies/<Module>/`
- Use the `Policy` suffix
- Policies should be auto-discovered; avoid manual registration unless needed

### Providers (`app/Providers/`)

- Module providers are responsible for feature bindings
- Prefer explicit bindings over magic
- Do not store request-specific data in providers

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

- [ ] Files created via `php artisan make:*` and placed in the correct folder
- [ ] Module folders used consistently across `app/Services/<Module>/`, `app/Models/<Module>/`, policies, and observers
- [ ] Class suffixes follow the table; exceptions only for models, enums, DTOs, actions, and traits
- [ ] Models have `$fillable`, correct casts, and a factory; no business logic in models
- [ ] DTOs use Spatie Laravel Data with camelCase fields and `from()` at boundaries
- [ ] Controllers stay thin, use form requests, delegate to services, and return responses/resources
- [ ] Services are stateless and DI-driven; no request data stored on properties
- [ ] Providers avoid singletons unless explicitly safe for Octane
- [ ] Policies and observers mirror model nesting; policies include `account_id` checks
- [ ] Eloquent used for queries; `DB::table()` only in migrations/seeders; `DB::transaction()` for transactions
- [ ] Errors fail loudly with no swallowed exceptions or misleading defaults
- [ ] Factories use `fake()` with deterministic defaults; seeders are idempotent

---

# Database Specifications

## Overview

Database design rules for all GRS migrations and schemas.

---

## Table Naming & Organization

### Module-Based Prefixes

Tables must use a module prefix:

- Calendar: `calendar_credentials`, `calendar_sync_states`, `calendar_sync_logs`
- Email: `email_templates`, `email_schedules`
- CRM: `crm_contacts`, `crm_deals`

---

## Account as Tenant

1. **All tables include `account_id`** — even when denormalized
2. **Use `account_id`, never `tenant_id`**
3. `account_id` always has a foreign key to `accounts`

### Example:

```php
Schema::create('calendar_sync_logs', function (Blueprint $table) {
    $table->id();
    $table->foreignId('account_id')->constrained('accounts')->cascadeOnDelete();
    $table->foreignId('calendar_sync_state_id')->nullable()->constrained('calendar_sync_states')->nullOnDelete();
    $table->string('status');
    $table->timestamps();
});
```

---

## Foreign Keys

### Default Behavior

**Use `->nullOnDelete()` for all foreign keys except `account_id`.**

- Most foreign keys should be `nullable` and use `->nullOnDelete()` to preserve records when related data is deleted
- Only `account_id` uses `->cascadeOnDelete()` for tenant isolation (when an account is deleted, all related data is purged)

### Examples:

**Account foreign key (always cascades):**
```php
$table->foreignId('account_id')->constrained()->cascadeOnDelete();
```

**Other foreign keys (nullable with nullOnDelete):**
```php
$table->foreignId('user_id')->nullable()->constrained()->nullOnDelete();
$table->foreignId('calendar_id')->nullable()->constrained('calendars')->nullOnDelete();
$table->foreignId('supervisor_id')->nullable()->constrained('users')->nullOnDelete();
```

### Rules

- Use `->constrained()` without a table name when the foreign key matches the convention (`{table}_id` → `{table}`)
- Explicitly specify the table name if it doesn't follow convention: `->constrained('custom_table_name')`
- **`account_id` always uses `->cascadeOnDelete()`** — no exceptions
- **All other foreign keys use `->nullable()->constrained()->nullOnDelete()`** by default
- **Never use legacy syntax**: `$table->foreign('column')->references('id')->on('table')`
- **Never use custom constraint names** — let Laravel generate them

---

## Indexes

Add indexes to:
- Foreign keys
- Frequent WHERE/JOIN/ORDER BY columns
- Search/filter fields
- Compound query keys

Preferred syntax:

```php
$table->string('email')->unique();
$table->string('name')->index();
$table->foreignId('account_id')->constrained()->cascadeOnDelete();
$table->string('status')->index();
```

Rules:

- Use `->index()` on individual columns
- Use `->unique()` where required
- Use `->nullable()->index()` for optional indexed columns
- Let Laravel generate index names by default (preferred)
- If you must name an index manually, follow Laravel's pattern: `{table}_{column1}_{column2}_{type}` where `{type}` is `index`, `unique`, or `foreign`
- Do not use legacy index syntax

---

## Enums

**Never use database enums** (no MySQL `ENUM`, no Laravel `enum()` columns).

Use strings/ints validated against application-level enums:

**Application enum (app/Enums/SyncStatus.php):**
```php
enum SyncStatus: string
{
    case PENDING = 'pending';
    case SYNCING = 'syncing';
    case COMPLETED = 'completed';
    case FAILED = 'failed';
}
```

**Migration:**
```php
$table->string('status'); // Not enum() — use string
```

**Validation:**
```php
$validated = $request->validate([
    'status' => ['required', Rule::enum(SyncStatus::class)],
]);
```

---

## Denormalization

Rules:

- You may add `account_id` anywhere it simplifies queries
- Document denormalization decisions (migration comment or commit)
- Only denormalize for clear performance/operational benefit
- Keep foreign key constraints even on denormalized fields

### Example:

A `calendar_events` table might include `account_id` even if it's accessed only through `calendar_sync_states` → `account_id`, because querying events directly by account is common:

```php
Schema::create('calendar_events', function (Blueprint $table) {
    $table->id();
    $table->foreignId('account_id')->constrained()->cascadeOnDelete(); // Denormalized for performance
    $table->foreignId('calendar_id')->nullable()->constrained('calendars')->nullOnDelete();
    $table->string('external_id')->unique();
    $table->string('title');
    $table->timestamps();
});
```

---

## Column Naming

Conventions:

- `snake_case` only
- Foreign keys: `{table}_id` (singular table name)
- Booleans: `is_*` (e.g., `is_active`, `is_deleted`)
- Timestamps: `created_at`, `updated_at`
- Soft deletes: `deleted_at`

### Date & Time Columns

For naming patterns (`*_at`, `*_on`, `*_time`, `*_epoch`) and storage rules, see [Dates & Time Handling](./dates.md).

Bad practices (avoid):

- Compound foreign key names: `calendar_user_id_start_datetime`
- CamelCase: `createdAt`, `userId`
- Abbreviations: `usr_id`, `acct_id`
- Custom naming schemes

Good example:

```php
$table->id();
$table->foreignId('account_id')->constrained()->cascadeOnDelete();
$table->foreignId('calendar_id')->nullable()->constrained('calendars')->nullOnDelete();
$table->string('name');
$table->boolean('is_active')->default(true);
$table->timestamp('synced_at')->nullable();
$table->timestamps();
```

---

## Column Clarity & Aggregation

Aggregation suffixes:

- `_count` — counts (`events_synced_count`)
- `_sum` — sums (`revenue_sum`)
- `_avg` — averages (`rating_avg`)
- `_min` — minimums (`price_min`)
- `_max` — maximums (`price_max`)

Good examples:
```php
$table->integer('events_synced_count')->default(0);
$table->unsignedInteger('user_count')->default(0);
```

Bad examples:
```php
$table->integer('events_synced'); // Unclear: count? JSON? Event IDs?
$table->integer('revenue');        // Could be sum, average, or single value
```

---

## External IDs

Use `ext_id` for any identifier from an external system:

Example:
```php
$table->string('ext_id')->nullable();  // External ID from third-party system
```

---

## Public vs Internal ID Exposure

### Rule: Never Expose Database IDs Publicly

Internal code uses `id`. Public-facing interfaces use `uuid` instead of `id`:
- REST APIs
- Admin portals
- User-facing applications
- Third-party integrations

### Implementation

Add a `uuid` column to any table that will be referenced publicly:

```php
Schema::create('calendar_events', function (Blueprint $table) {
    $table->id(); // Internal use only
    $table->uuid('uuid')->unique(); // Public-facing identifier
    $table->foreignId('account_id')->constrained()->cascadeOnDelete();
    $table->string('title');
    $table->timestamps();
});
```

### Usage Patterns

**Internal (Service Layer):**
```php
// Use id for internal lookups
$event = CalendarEvent::find($id); // or findOrFail()
```

**Public APIs:**
```php
// Route accepts UUID
Route::get('/api/events/{uuid}', EventController::class);

// Controller uses UUID to look up record
public function show(string $uuid)
{
    $event = CalendarEvent::where('uuid', $uuid)->firstOrFail();
    return EventResource::make($event);
}
```

**API Resources:**
```php
class EventResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->uuid, // Return UUID, not database id
            'title' => $this->title,
            'created_at' => $this->created_at,
        ];
    }
}
```

---

## Soft Deletes

When soft deletes are required, use Laravel's `SoftDeletes` trait:

```php
use Illuminate\Database\Eloquent\SoftDeletes;

class Calendar extends Model
{
    use SoftDeletes;
}
```

In migrations:

```php
$table->softDeletes();
```

---

## Timestamps

Always include timestamps on tables that represent entities with state:

```php
$table->timestamps(); // creates created_at and updated_at
```

Omit timestamps only for purely transactional or log tables where history is tracked differently.

---

## Summary Checklist

Before committing any migration:

- [ ] Module-based table prefix is applied
- [ ] `account_id` is present and properly constrained
- [ ] Foreign keys use modern fluent syntax (`->constrained()`)
- [ ] Foreign keys have appropriate cascade behavior
- [ ] Indexes are applied with `->index()` and use Laravel's default naming (no custom names unless required)
- [ ] No database enums are used
- [ ] Column names follow `snake_case` convention
- [ ] Aggregation columns use clear suffixes (`_count`, `_sum`, `_avg`, etc.)
- [ ] External IDs use `ext_id` naming
- [ ] Public-facing tables have `uuid` column (never expose `id`)
- [ ] No custom constraint or index names are specified
- [ ] Denormalization is documented if present

---

# Dates & Time Handling

## Overview

GRS date/time rules for storage and multi-timezone behavior.

---

## Core Principles

1. **Always store UTC only** in the database (no local time or offset storage)
2. **Use Carbon/CarbonImmutable**, not native `DateTime`
3. **Convert to user timezone only at the edges** (input/output)
4. **Use CarbonImmutable** for event-like, immutable data

---

## Column Naming Conventions

Use these patterns across all tables:

| Pattern | Type | Example | Usage |
|---------|------|---------|-------|
| `*_at` | DateTime | `created_at`, `synced_at`, `expires_at` | Datetime stored as UTC |
| `*_on` | Date | `birth_date_on`, `scheduled_on`, `starts_on` | Date only (no time) |
| `*_time` | Time | `start_time`, `end_time` | Time only (no date) |
| `*_epoch` | Integer | `created_epoch`, `timestamp_epoch` | Unix timestamp (seconds) |

### Examples

```php
// Datetime columns (stored as UTC)
$table->timestamp('created_at');
$table->timestamp('synced_at')->nullable();
$table->timestamp('expires_at')->nullable();

// Date-only columns
$table->date('birth_date_on');
$table->date('scheduled_on');

// Time-only columns
$table->time('shift_start_time');
$table->time('shift_end_time');

// Epoch/Unix timestamp columns (rarely needed)
$table->unsignedBigInteger('created_epoch')->nullable();
```

---

## Using Carbon

- Import `Illuminate\Support\Carbon` (and `CarbonImmutable` where needed)
- Prefer `Carbon` for regular business logic; `CarbonImmutable` for domain events/value objects

---

## Database Storage Rules

### Always Store UTC

Every datetime column in the database is UTC, never local.

```php
// ✅ CORRECT: Always store as UTC
$model->expires_at = Carbon::now('UTC')->addDays(30);
//or
$model->expires_at = Carbon::now()->addDays(30);

// ❌ WRONG: Never store in user's timezone
$model->expires_at = Carbon::now('Europe/London')->addDays(30);
```

### Casting to Carbon

Ensure all datetime columns are cast to Carbon in models:

```php
class Calendar extends Model
{
    protected $casts = [
        'created_at' => 'datetime',
        'updated_at' => 'datetime',
        'synced_at' => 'datetime',
        'expires_at' => 'datetime',
        'scheduled_on' => 'date',
        'shift_start_time' => 'time',
    ];
}
```

**Explicit casting for immutables:**

```php
class Event extends Model
{
    protected $casts = [
        'occurred_at' => 'immutable_datetime',
        'scheduled_on' => 'immutable_date',
    ];
}
```

---

## Displaying Dates to Users

### Convert on Output Only

Do not store user timezone in datetime columns; convert only when displaying:

```php
// In a model or transformer
public function getDisplayableExpireDate(): string
{
    return $this->expires_at
        ->setTimezone(auth()->user()->timezone)
        ->format('Y-m-d H:i:s');
}

// In a controller
return [
    'expires_at' => $model->expires_at
        ->setTimezone($user->timezone)
        ->toIso8601String(),
];
```

### User Timezone Storage

Store user timezone preference as a string (e.g. `America/New_York`):

```php
class User extends Model
{
    protected $casts = [
        'timezone' => 'string', // e.g., 'America/New_York', 'Europe/London'
    ];
}
```

Valid timezone strings are from PHP's supported timezones.

---

## Handling User Input

### Parse Input as User Timezone, Store as UTC

```php
// User submits time in their timezone
public function store(Request $request)
{
    $userTimezone = auth()->user()->timezone;

    // Parse in user's timezone, convert to UTC for storage
    $scheduledAt = Carbon::createFromFormat(
        'Y-m-d H:i:s',
        $request->scheduled_at,
        $userTimezone
    )->setTimezone('UTC');

    Calendar::create([
        'scheduled_at' => $scheduledAt,
    ]);
}
```

---

## Querying & Filtering

### Always Query in UTC

```php
// ✅ CORRECT: Query with UTC
$events = Calendar::where('scheduled_at', '>=', Carbon::now('UTC'))
    ->get();

// ✅ CORRECT: Query with date range in UTC
$startOfDay = Carbon::parse('2025-12-23 00:00:00', 'UTC');
$endOfDay = Carbon::parse('2025-12-23 23:59:59', 'UTC');

$events = Calendar::whereBetween('scheduled_at', [$startOfDay, $endOfDay])
    ->get();
```

### Relative Queries

```php
// Events synced in the last hour (UTC)
$recentEvents = Calendar::where('synced_at', '>=', Carbon::now()->subHour())
    ->get();

// Events created today (in UTC)
$todayStart = Carbon::now()->startOfDay(); // or now()->startOfDay()
$todayEnd = Carbon::now()->endOfDay(); // or now()->endOfDay()

$todayEvents = Calendar::whereBetween('created_at', [$todayStart, $todayEnd])
    ->get();
```

---

## Timezone Pitfalls to Avoid

### ❌ Never Mix Timezones in Comparisons

```php
// WRONG: Mixing timezones
if ($model->expires_at < Carbon::now('America/New_York')) {
    // This comparison is invalid — one is UTC, one is EST
}

// CORRECT: Both in UTC
if ($model->expires_at < Carbon::now('UTC')) {
    // Both are in UTC, comparison is valid
}
```

### ❌ Don't Use PHP's date() Function

```php
// WRONG: Uses server timezone
$date = date('Y-m-d H:i:s'); // Server timezone dependent

// CORRECT: Use Carbon
$date = Carbon::now('UTC')->format('Y-m-d H:i:s');
```

---

## Diff & Duration

### Calculating Time Differences

```php
// Get difference as CarbonInterval
$interval = $model->expires_at->diff(Carbon::now());

// Get human-readable format
echo $interval->forHumans(); // "in 5 days", "3 hours ago"

// Get total seconds
$seconds = $model->expires_at->diffInSeconds(Carbon::now());

// Get specific unit
$days = $model->expires_at->diffInDays(Carbon::now());
$hours = $model->expires_at->diffInHours(Carbon::now());
```

---

## Database Migrations

### DateTime Columns

```php
// Automatically adds created_at and updated_at as datetime
$table->timestamps();

// Explicit datetime column
$table->timestamp('expires_at')->nullable();

```

### Date & Time Columns

```php
// Date only (no timezone considerations)
$table->date('birth_date_on');

// Time only (no timezone considerations)
$table->time('shift_start_time');

// Nullable variants
$table->date('scheduled_on')->nullable();
$table->time('end_time')->nullable();
```

---

## Soft Deletes with Timestamps

`deleted_at` follows the same UTC rules:

```php
use Illuminate\Database\Eloquent\SoftDeletes;

class Calendar extends Model
{
    use SoftDeletes;
}
```

Queries automatically exclude soft-deleted records:

```php
// Excludes soft-deleted records
$events = Calendar::all();

// Includes soft-deleted records
$events = Calendar::withTrashed()->get();

// Only soft-deleted records
$events = Calendar::onlyTrashed()->get();
```

---

## Testing with Dates

### Freeze Time in Tests

```php
use Illuminate\Support\Facades\Date;

test('event_expires', funciton () {
    // Freeze time at a specific point
    Date::setTestNow('2025-12-23 12:00:00');

    $event = Calendar::create([
        'expires_at' => Carbon::now()->addDays(7),
    ]);

    // Assertions
    $this->assertTrue($event->isActive());

    // Move time forward
    Date::setTestNow('2025-12-31 12:00:00');

    $this->assertFalse($event->isActive());
});
```

---

## Summary Checklist

Before committing code that handles dates:

- [ ] All datetime columns use `*_at` naming convention
- [ ] All dates stored in database are UTC
- [ ] Models cast datetime columns to Carbon
- [ ] User timezone conversion happens only for display/input
- [ ] Queries use UTC timestamps with `Carbon::now('UTC')`
- [ ] No usage of PHP's `date()` or native `DateTime` in business logic
- [ ] No timezone information stored in database (except user preferences)
- [ ] Immutable Carbon used for domain events if applicable

---

# Testing Specifications

## Overview

Testing standards for GRS using **Pest**.

---


## Test Types

### Unit Tests

- **No database access** — never use `RefreshDatabase` or `DatabaseTransactions`
- No HTTP requests
- Single method/function focus
- Use `Model::factory()->make()` to create model instances **without persisting**
- Use mocks/stubs for all dependencies
- Should be fast (no I/O operations)
- Location: `tests/Unit/`

```php
// ✅ Unit test - no database
test('calculates order total correctly', function () {
    $items = collect([
        OrderItem::factory()->make(['price' => 100, 'quantity' => 2]),
        OrderItem::factory()->make(['price' => 50, 'quantity' => 1]),
    ]);

    $calculator = new OrderCalculator();

    expect($calculator->total($items))->toBe(250);
});
```

### Feature Tests

- **Access database** using `RefreshDatabase` or `DatabaseTransactions`
- Test full HTTP request/response cycle (controllers, middleware, validation)
- Use `Model::factory()->create()` to persist data
- Location: `tests/Feature/`

```php
// ✅ Feature test - with database
test('user can view their profile', function () {
    $user = User::factory()->create();

    actingAs($user)
        ->get('/profile')
        ->assertOk()
        ->assertSee($user->name);
});
```

### Integration Tests

- **Access database** using `RefreshDatabase` or `DatabaseTransactions`
- Test multiple classes working together (without HTTP layer)
- Test service classes, repositories, jobs, events
- Verify database state changes
- Location: `tests/Integration/`

```php
// ✅ Integration test - services with database
test('order service creates order with items', function () {
    $user = User::factory()->create();
    $products = Product::factory()->count(3)->create();

    $service = new OrderService();
    $order = $service->createOrder($user, $products);

    expect($order)->toBeInstanceOf(Order::class);
    expect($order->items)->toHaveCount(3);
    $this->assertDatabaseHas('orders', ['user_id' => $user->id]);
});
```

### Summary Table

| Test Type   | Database | Factory Method | What it tests                        |
|-------------|----------|----------------|--------------------------------------|
| Unit        | No       | `make()`       | Single class/method in isolation     |
| Integration | Yes      | `create()`     | Multiple classes working together    |
| Feature     | Yes      | `create()`     | Full HTTP request/response cycle     |

---

## Test Functions

### Preferred Syntax

- Prefer `test()` for defining cases
- `it()` is allowed but `test()` is the default style

---

## External HTTP Calls

When a test makes real HTTP calls to external services (not the app), label it with the `external` group:

```php
test('fetches data from external API', function () {
    // HTTP call to external service
})->group('external');
```

This allows external tests to be skipped or run separately.

---

## Test Datasets

- Reusable datasets live in `tests/Datasets/*Dataset.php`
- Name datasets descriptively (e.g. `valid articles`, `invalid articles`)
- Refer to Pest documentation for rules on datasets, helpers, and other test utilities

```php
// tests/Datasets/ArticleDataset.php
<?php

dataset('valid articles', [
    [
        'title' => 'First Article',
        'content' => 'Article content',
        'published' => true,
    ],
    [
        'title' => 'Second Article',
        'content' => 'More content',
        'published' => false,
    ],
]);

dataset('invalid articles', [
    [
        'title' => '',
        'content' => 'Missing title',
    ],
    [
        'title' => 'Article',
        'content' => '',
    ],
]);
```

---

## Controller Testing

### HTTP Test Functions

```php
test('user can create a post', function () {
    $response = actingAs($this->user)
        ->postJson(action([PostController::class, 'store']), [
            'title' => 'Test Post',
            'content' => 'Test content',
        ])
        ->assertCreated();
});

test('user can view index', function () {
    $response = actingAs($this->user)
        ->getJson(action([PostController::class, 'index']))
        ->assertOk();
});

test('unauthenticated user is forbidden', function () {
    getJson(action([PostController::class, 'index']))
        ->assertUnauthorized();
});
```

### Action Testing

```php
test('user controller returns users', function () {
    $response = $this->get(action([UserController::class, 'index']));
    expect($response->status())->toBe(200);
});

test('user controller stores user', function () {
    $response = postJson(action([UserController::class, 'store']), [
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ]);

    expect($response->status())->toBe(201);
});
```

### Forbidden Patterns

**Do not use `route()`** in tests:

```php
// ❌ Don't do this
$response = $this->get(route('users.index'));
```

**Do not use old PHPUnit `$this->assert*()` methods** — use response assertion methods or Pest's `expect()` instead:

```php
// ❌ Don't do this
$this->assertEquals($response->status(), 200);
$this->assertTrue($user->exists());

// ✅ Do this with response methods
actingAs($this->user)
    ->getJson(action([UserController::class, 'index']))
    ->assertOk();

// ✅ Or use expect() for other assertions
expect($user->exists())->toBeTrue();
```

---

## Mocking Best Practices

When mocking dependencies, follow this pattern:

```php
use Mockery;

test('handler processes ticket with mocked services', function () {
    // Create mock for external service
    $getLiveTicket = Mockery::mock(GetLiveTicket::class);
    $getLiveTicket->shouldReceive('handle')
        ->andReturn([
            'id' => 123,
            'title' => 'Test Ticket',
        ]);

    // Create mock for analysis service
    $triageResult = new TriageResult(
        classifications: [
            'category' => [
                'id' => 2,
                'name' => 'Feature',
                'confidence' => 0.85,
            ],
        ],
        confidence: 0.85,
    );

    $analysisService = Mockery::mock(AnalysisService::class);
    $analysisService->shouldReceive('analyze')
        ->andReturn($triageResult);

    // Create handler and test
    $handler = new TriageHandler($getLiveTicket, $analysisService);
    $result = $handler->process();

    expect($result)->toBeInstanceOf(TriageResult::class);
    expect($result->confidence)->toBe(0.85);
});
```

Key points:
- Use `Mockery::mock()`
- Use `shouldReceive()` / `andReturn()`
- Assert the behavior that matters

---

## HTTP Request Mocking

When testing code that makes HTTP requests to external services, use `Http::fake()`.

```php
use Illuminate\Support\Facades\Http;

test('fetches user data from external API', function () {
    Http::fake([
        'api.example.com/users/*' => Http::response([
            'id' => 1,
            'name' => 'John Doe',
            'email' => 'john@example.com',
        ]),
    ]);

    $result = app(UserService::class)->fetchUser(123);

    expect($result['name'])->toBe('John Doe');
});
```

### Asserting Requests Were Sent

```php
test('sends correct request to external API', function () {
    Http::fake([
        'api.example.com/sync' => Http::response(['status' => 'success']),
    ]);

    app(SyncService::class)->syncData(['user_id' => 456]);

    Http::assertSent(fn ($request) =>
        $request->url() === 'https://api.example.com/sync' &&
        $request->method() === 'POST'
    );
});
```

```php
test('handles multiple external API calls', function () {
    Http::fake([
        'api.service-a.com/data' => Http::response(['data' => 'from A']),
        'api.service-b.com/config' => Http::response(['config' => 'from B']),
    ]);

    $service = app(AggregatorService::class);
    $result = $service->aggregate();

    expect($result['a'])->toBe('from A');
    expect($result['b'])->toBe('from B');

    Http::assertSent(fn ($request) =>
        str_contains($request->url(), 'api.service-a.com')
    );
});
```

```php
test('makes correct number of API calls', function () {
    Http::fake();

    app(BatchProcessor::class)->process(10);

    Http::assertSentCount(10);
});
```

```php
test('handles API errors gracefully', function () {
    Http::fake([
        'api.example.com/risky' => Http::response(
            ['error' => 'Service unavailable'],
            status: 503
        ),
    ]);

    $result = app(RiskyService::class)->attempt();

    expect($result['success'])->toBeFalse();
});
```

Key points:
- Use `Http::fake()` to prevent real network requests
- Use wildcards (`*`) for dynamic endpoints
- Use `Http::assertSent()` and `Http::assertSentCount()`

---

## Test Organization

```
tests/
├── Unit/
│   ├── Services/
│   ├── Models/
│   └── Utils/
├── Feature/
│   ├── Articles/
│   ├── Users/
│   └── ...
├── Integration/
│   ├── Workflows/
│   └── ...
├── Helpers/
│   ├── FeatureOneHelpers.php
│   ├── FeatureTwoHelpers.php
│   └── ...
└── Datasets/
    ├── FeatureOneDataset.php
    ├── FeatureTwoDataset.php
    └── ...
```

---

## Database Handling in Tests

Feature and Integration tests that access the database should use Laravel's database testing utilities:

```php
test('creates article in database', function () {
    postJson('/articles', [
        'title' => 'Test Article',
        'content' => 'Test content',
    ]);

    expect(Article::query()->count())->toBe(1);
});
```

Database is automatically refreshed between tests.

---

## Seed Data & Test Data

### Tests Must NEVER Rely on Seed Data

**Seed data is strictly for demos and walkthroughs — never for testing.**

Tests must be self-contained and create all required data using factories. Relying on seeders makes tests:

- Brittle (break when seeders change)
- Non-deterministic (different results in different environments)
- Hard to understand (hidden dependencies on external data)

```php
// ❌ WRONG: relying on seeded data
$account = Account::find(1);
$user = User::where('email', 'admin@example.com')->first();

// ❌ WRONG: calling seeders in tests
$this->seed(AccountSeeder::class);

// ✅ CORRECT: create all data in the test
$account = Account::factory()->create();
$user = User::factory()->admin()->create();
```

### Hardcoded IDs

Never hardcode IDs. Always reference created models directly.

```php
// ❌ WRONG: hardcoded ID
$response = getJson('/users/1');

// ✅ CORRECT: use the created model's ID
$user = User::factory()->create();
$response = getJson("/users/{$user->id}");
```

### Factory States for Specific Scenarios

Use factory states to create models with specific attributes instead of relying on seeded data:

```php
// In UserFactory.php
public function admin(): static
{
    return $this->state(fn (array $attributes) => [
        'role' => 'admin',
    ]);
}

// In test
$admin = User::factory()->admin()->create();
```

---

## Common Patterns

### Testing JSON API Responses

```php
test('api returns correct user data', function () {
    $user = User::factory()->create();

    $response = getJson("/users/{$user->id}");

    expect($response->json())
        ->toHaveKey('data.id', $user->id)
        ->toHaveKey('data.email', $user->email);
});
```

### Testing with Factory Data

```php
test('processes multiple users', function () {
    $users = User::factory()->count(5)->create();

    $response = getJson('/users');

    expect($response->json('data'))->toHaveCount(5);
});
```

### Testing Exception Handling

```php
test('throws exception for invalid input', function () {
    $this->expectException(InvalidArgumentException::class);

    new Article(['title' => '']);
});
```

---

## Summary

- Prefer `test()`
- Organize by type: Unit, Feature, Integration
- Use `expect()` over `$this->assert*()`
- Use `action()` and HTTP helpers for controllers
- Group external HTTP tests with `external`
- Keep datasets in `tests/Datasets/`
- Use Mockery for dependencies
- Use `Http::fake()` + `Http::assertSent*()` for external HTTP
- Keep tests focused and isolated

---

# Error Handling Specifications

## Overview

Error handling rules for GRS. Assume Laravel conventions and apply these project-specific standards.

---

## Core Principles

- **Fail loudly, never quietly**
- **When in doubt, throw an exception**
- **Never hide or auto-fix an error**
- **Do not use default values to "avoid" errors**
- **Use specific exception classes, not generic `Exception`**
- **Exception messages must include helpful context (IDs, account, etc.)**

These apply everywhere: controllers, services, jobs, listeners, console commands, and background workers.

---

## Fail Loudly

Code must fail **immediately and explicitly** when something is wrong.

- Do not swallow exceptions or return "safe" but invalid values just to keep code running
- Do not catch-and-ignore exceptions; only catch if you can handle them meaningfully
- Prefer Laravel's validation and typed inputs over nullable parameters

Bad:

```php
// Swallows the real error
try {
    $service->handle($payload);
} catch (Throwable $e) {
    Log::warning('failed to handle payload');
}
```

Good:

```php
// Let the exception bubble or wrap in a specific one
$service->handle($payload); // throws a domain-specific exception if invalid
```

---

## No Silent Defaults

Do **not** use default values to mask missing or invalid data.

- Enforce required values via validation or explicit checks
- Never default a required value just to avoid an error
- If required input is missing or invalid, throw (or let validation fail) instead of guessing

Bad:

```php
// Hides upstream bug by silently defaulting
$quantity = $payload['quantity'] ?? 0;
```

Good:

```php
if (! array_key_exists('quantity', $payload)) {
    throw new InvalidArgumentException('Missing required key: quantity');
}

$quantity = $payload['quantity'];
```

Laravel controllers should normally rely on form request validation rather than manual checks.

---

## Logging Is Not a Fix

Logging alone does **not** fix a bug.

- Do not replace proper error handling with `Log::warning()`, `Log::info()`, or similar
- Logs are for observability; exceptions are for control flow on error paths
- If something is broken, throw (or let Laravel throw) and log in addition if helpful

Bad:

```php
// Attempts to "handle" an error with only a log
if (! $user) {
    Log::warning('User not found');
    return null;
}
```

Good:

```php
if (! $user) {
    throw new ModelNotFoundException('User not found: ' . $username);
}
```

Use Laravel's built-in exceptions (`ModelNotFoundException`, `AuthorizationException`, `ValidationException`, etc.) or custom domain exceptions where appropriate.

---

## Use Specific Exception Classes

Always choose the **most specific** exception type:

- Validation issues → let Laravel form requests / `Validator` throw `ValidationException`
- Missing models → `ModelNotFoundException` or `firstOrFail()` / `findOrFail()`
- Authorization issues → `AuthorizationException` / `Gate::authorize()`
- Domain errors → custom exceptions like `InvalidTriageStateException`
- Invalid arguments → `InvalidArgumentException`

Avoid `throw new \Exception('error');` in production code.

Bad:

```php
throw new \Exception('error');
```

Better:

```php
throw new \InvalidArgumentException("Can't find id={$id} for account={$accountId}");
```

Or with a custom exception:

```php
throw new CannotFindResourceForAccount("resource_id={$id} account_id={$accountId}");
```

---

## Exception Messages With Context

Exception messages must include enough context to debug quickly.

- Include key identifiers: IDs, account IDs, external IDs, module names
- Briefly describe what was attempted and why it failed
- Avoid generic messages like `"error"` or `"something went wrong"`

Bad:

```php
throw new InvalidArgumentException('error');
```

Good:

```php
throw new InvalidArgumentException(
    "Cannot find triage id={$triageId} for account_id={$accountId}"
);
```

---

## Laravel Alignment

These rules sit on top of Laravel's normal patterns:

- Prefer form request validation and typed DTOs over manual `isset()`/`empty()` checks
- Use `abort_if` / `abort_unless` / `throw_if` / `throw_unless` helpers where they improve clarity
- Let Laravel's exception handler convert exceptions into HTTP responses; do not manually suppress them

If you are unsure whether to log or throw, **throw** (and optionally log) rather than silently continuing.

---

## Summary Checklist

Before committing error-handling code:

- [ ] No exceptions are swallowed or ignored
- [ ] No required values are defaulted just to "avoid" errors
- [ ] Logging is not used as a substitute for proper exceptions
- [ ] Exception classes are specific and appropriate
- [ ] Exception messages include relevant IDs/context
- [ ] Laravel validation and helpers are used where appropriate

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

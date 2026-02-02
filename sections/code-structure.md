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
    ├── Support/                       # helper or additional services
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
- Every model **must define a `$rules` array** for validation
- Every model **must use the `ValidatesOnSave` trait**
- Every model **must include a phpdoc block with `@property` tags** so phpstan can understand the model fields and relations
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

use App\Models\Traits\ValidatesOnSave;
use App\Models\User;
use Database\Factories\UnifiedCalendar\WorkScheduleFactory;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

/**
 * @property int $id
 * @property string $uuid
 * @property int|null $account_id
 * @property int $user_id
 * @property int|null $schedule_layer_id
 * @property int $weekday
 * @property string $starts_time
 * @property string $ends_time
 * @property string|null $title
 * @property string|null $notes
 * @property-read ScheduleLayer|null $layer
 * @property-read User $user
 */
class WorkSchedule extends Model
{
    /** @use HasFactory<WorkScheduleFactory> */
    use HasFactory, ValidatesOnSave;

    protected $table = 'universal_calendar_work_schedules';

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

    protected $rules = [
        'account_id' => 'required|integer|exists:accounts,id',
        'user_id' => 'nullable|integer|exists:users,id',
        'schedule_layer_id' => 'nullable|integer|exists:universal_calendar_schedule_layers,id',
        'weekday' => 'required|integer|between:1,7',
        'starts_time' => 'required|string|date_format:H:i:s',
        'ends_time' => 'required|string|date_format:H:i:s',
        'title' => 'nullable|string|max:255',
        'notes' => 'nullable|string|max:255',
    ];

    public function layer(): BelongsTo
    {
        return $this->belongsTo(ScheduleLayer::class, 'schedule_layer_id');
    }

    // other relationships, scopes, etc.
}
```

### Console Commands (`app/Services/<Module>/Console/Commands/`)

- Commands live under `app/Services/<Module>/Console/Commands/`
- Use the `Command` suffix
- Command signatures use lowercase, kebab-case segments separated by `:` and follow the pattern `<domain>:<feature>:<action>` (use the shortest form that fits, e.g., two segments when a feature domain is sufficient)
- Options must be kebab-case, explicit, and include descriptions and defaults where applicable
- Required parameters are positional arguments
- Optional parameters are named options (`--from=`, `--to=`, etc.)
- Commands that operate on users must require both `{account}` and `{user}` to avoid multi-account ambiguity
- Standardize parameter names to semantic, domain-level names (avoid database column naming)
  - Use `account` (not `account-id` / `account_id`)
  - Use `user` (not `user-id` / `user_id`)
  - Use `integration` (not `integration-id` / `integration_id`)
- Example signature (multi-option):
```php
protected $signature = 'helpdesk:discovery:changes
    {account : The account ID}
    {integration : The integration ID}
    {--tag= : Find integration by tag instead of ID}
    {--entity= : Filter by entity type (ticket, company)}
    {--type= : Filter by change type (field_added, field_deprecated, etc.)}
    {--since=7 days ago : Show changes since date (default: 7 days ago)}
    {--limit=50 : Maximum changes to show}';
```
- Example signature (single option):
```php
protected $signature = 'ticket-assign:explain {decision : Decision ID}';
```

#### CLI Interactivity

Commands must be **non-interactive by default**. Never prompt for confirmation with `$this->confirm()`, `$this->ask()`, or similar interactive prompts during normal execution.

If a command performs a destructive or dangerous operation, protect it with a `--confirm` option instead of an interactive prompt:

```php
// WRONG: Interactive confirmation
public function handle(): int
{
    if (! $this->confirm('Are you sure you want to delete this user?')) {
        return self::FAILURE;
    }
    // ...
}

// CORRECT: Non-interactive with --confirm flag
protected $signature = 'user:delete {user : The user ID} {--confirm : Actually perform the deletion}';

public function handle(): int
{
    $user = User::findOrFail($this->argument('user'));

    if (! $this->option('confirm')) {
        $this->info("This would delete user {$user->username} (id:{$user->id}). Use --confirm to perform the deletion.");
        return self::SUCCESS;
    }

    $user->delete();
    $this->info("User {$user->username} (id:{$user->id}) deleted.");
    return self::SUCCESS;
}
```

This pattern ensures commands are scriptable and can be safely used in pipelines, cron jobs, and automation without hanging on interactive prompts.

#### CLI Output

All command options must use **kebab-case** (e.g., `--id-only`, `--dry-run`), never camelCase (`--idOnly`) or snake_case (`--id_only`).

**Table output** must follow Laravel's built-in `$this->table()` pattern. When a command produces table output, always include these options:

| Option       | Description                                                                 |
|--------------|-----------------------------------------------------------------------------|
| `--json`     | Output as JSON instead of a table                                           |
| `--limit=`   | Limit the number of rows displayed                                          |
| `--id-only`  | Print only IDs with no header (one per line, suitable for piping to scripts)|

The `--id-only` option applies when the output rows represent database records. It prints raw IDs with no table formatting, so the output can drive other commands (e.g., `artisan user:list --id-only | xargs -I{} artisan user:sync {}`).

**Column ordering** for table output must follow this sequence:

1. `id` (database primary key) — always first
2. `created_at`
3. `account_id` (if applicable)
4. `user_id` (if applicable)
5. All other fields in logical order

Be selective with columns — only show fields relevant to the command's purpose. Do not dump every database column into the table.

**Example:**
```php
protected $signature = 'user:list
    {account : The account ID}
    {--json : Output as JSON}
    {--limit= : Limit the number of rows}
    {--id-only : Print only user IDs (no header)}';

public function handle(): int
{
    $query = User::where('account_id', $this->argument('account'));

    if ($limit = $this->option('limit')) {
        $query->limit((int) $limit);
    }

    $users = $query->get();

    if ($this->option('id-only')) {
        $users->each(fn (User $user) => $this->line($user->id));
        return self::SUCCESS;
    }

    if ($this->option('json')) {
        $this->line($users->map(fn (User $user) => [
            'id' => $user->id,
            'created_at' => $user->created_at->toIso8601String(),
            'account_id' => $user->account_id,
            'username' => $user->username,
            'email' => $user->email,
        ])->toJson(JSON_PRETTY_PRINT));
        return self::SUCCESS;
    }

    $this->table(
        ['ID', 'Created', 'Account', 'Username', 'Email'],
        $users->map(fn (User $user) => [
            $user->id,
            $user->created_at->toDateTimeString(),
            $user->account_id,
            $user->username,
            $user->email,
        ]),
    );

    return self::SUCCESS;
}
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
- Helper/auxiliary services live in `app/Services/<Module>/Support/`
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

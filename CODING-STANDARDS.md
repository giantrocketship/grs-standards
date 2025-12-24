# GRS Coding Standards

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
   - [Dates & time](./sections/dates.md)
   - [Testing](./sections/testing.md)
   - [Error handling](./sections/error-handling.md)

## Guiding Principles

- Prefer **clarity over cleverness**
- Follow **Laravel conventions first**, project conventions second
- Avoid premature abstraction
- Code must be production-ready, testable, and maintainable
- Always consider Octane performance characteristics

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

- Use module subdirectories
- Focus models on relationships, casts, scopes
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

- Use the `Cast` suffix
- Organize by module
- Register via model `$casts`

### DTOs (`app/DTOs/`)

DTOs are always Spatie Laravel Data objects.

- **Always use Spatie Laravel Data**; do not hand-roll DTOs
- Organize by module: `app/DTOs/Triage/`, `app/DTOs/Calendar/`, etc.
- Suffix is optional: `TriageData.php`, `CreateTriageData.php`
- Use attributes for validation
- Use `from()` to create DTOs from requests/models
- Use DTOs at API and service boundaries; no business logic inside

### Jobs (`app/Jobs/`)

- Use the `Job` suffix
- Organize by module
- Keep jobs focused and idempotent

### Exceptions (`app/Exceptions/`)

- Use the `Exception` suffix
- Organize by module
- Provide meaningful messages and context

### Policies (`app/Policies/`)

- Use the `Policy` suffix
- Organize by module
- Always check `account_id` for tenant isolation
- Register in `AuthServiceProvider`

### Services (`app/Services/`)

- Use the `Service` suffix
- Organize by module
- Inject dependencies via constructor
- Single responsibility per service
- Use DTOs for data transfer
- Dispatch jobs for async work

### Contracts/Interfaces (`app/Contracts/` or `app/Services/Module/Contracts/`)

- Use `Interface` suffix: `TriageRepositoryInterface.php`
- Core/global contracts in `app/Contracts/`
- Module-specific contracts in `app/Services/Module/Contracts/`
- Bind interfaces to implementations in service providers

### Traits (`app/Traits/` or `app/Services/Module/Traits/`)

- Use the `Trait` suffix
- Global traits in `app/Traits/`
- Module traits in `app/Services/Module/Traits/`
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
- Do not specify custom index names
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
- [ ] Indexes are applied with `->index()` (no custom names)
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

1. **Always store UTC** in the database
2. **Use Carbon/CarbonImmutable**, not native `DateTime`
3. **Convert to user timezone only at the edges** (input/output)
4. **Use CarbonImmutable** for event-like, immutable data

---

## Column Naming Conventions

Use these patterns across all tables:

| Pattern | Type | Example | Usage |
|---------|------|---------|-------|
| `*_at` | DateTime | `created_at`, `synced_at`, `expires_at` | Datetime stored as UTC |
| `*_on` | Date | `birth_date`, `scheduled_on`, `starts_on` | Date only (no time) |
| `*_time` | Time | `start_time`, `end_time` | Time only (no date) |
| `*_epoch` | Integer | `created_epoch`, `timestamp_epoch` | Unix timestamp (seconds) |

### Examples

```php
// Datetime columns (stored as UTC)
$table->dateTime('created_at');
$table->dateTime('synced_at')->nullable();
$table->dateTime('expires_at')->nullable();

// Date-only columns
$table->date('birth_date');
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
$todayStart = Carbon::now()->startOfDay();
$todayEnd = Carbon::now()->endOfDay();

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
$table->dateTime('expires_at')->nullable();

// Datetime with default (current UTC)
$table->dateTime('scheduled_at')->useCurrent();

// Datetime with timezone-aware default (DO NOT USE with Laravel)
// Laravel handles UTC conversion automatically
```

### Date & Time Columns

```php
// Date only (no timezone considerations)
$table->date('birth_date');

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

    protected $casts = [
        'deleted_at' => 'datetime',
    ];
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

public function test_event_expires()
{
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
}
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

- No DB
- No HTTP
- Single method/function focus
- Location: `tests/Unit/`

### Feature Tests

- HTTP-level features
- May hit DB
- Smaller scope than integration
- Location: `tests/Feature/`

### Integration Tests

- End-to-end workflows
- Cross-module behavior
- May involve external services
- Location: `tests/Integration/`

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
└── Datasets/
    ├── ArticleDataset.php
    ├── UserDataset.php
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

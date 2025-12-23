# Database Specifications

## Overview

This document defines the database design conventions and standards for the project. All database migrations, schemas, and relationships must follow these rules strictly.

---

## Table Naming & Organization

### Module-Based Prefixes

Each module has its own namespace and tables must use a module-specific prefix. This ensures clarity and prevents naming collisions.

**Example:**
- Calendar module: `calendar_credentials`, `calendar_sync_states`, `calendar_sync_logs`
- Email module: `email_templates`, `email_schedules`
- CRM module: `crm_contacts`, `crm_deals`

---

## Account as Tenant

The `account` model serves as the central tenant structure for the application.

### Rules:

1. **All tables must include `account_id`** — even if the relationship is indirect or denormalized
2. **Use `account_id`, never `tenant_id`** — this is consistent across the entire codebase
3. **Foreign key to accounts table is required** for data isolation and cascading behavior

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

### Strict Laravel Conventions

Always use the modern Laravel fluent syntax for foreign keys. Never use legacy `references()` and `on()` methods.

### Default Behavior:

**Default rule: Use `->nullOnDelete()` for all foreign keys except `account_id`**

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

### Rules:

- Use `->constrained()` without a table name when the foreign key matches the convention (`{table}_id` → `{table}`)
- Always specify the table name explicitly if it doesn't follow convention: `->constrained('custom_table_name')`
- **`account_id` always uses `->cascadeOnDelete()`** — no exceptions
- **All other foreign keys use `->nullable()->constrained()->nullOnDelete()`** — this is the default pattern
- **Never use legacy syntax**: `$table->foreign('column')->references('id')->on('table')`
- **Never use custom constraint names** — let Laravel generate them automatically

---

## Indexes

### When to Index

Add indexes to columns that are:
- Foreign keys (automatically indexed in most cases)
- Frequently used in WHERE, JOIN, or ORDER BY clauses
- Searchable or filterable fields
- Part of compound queries

### Syntax:

```php
$table->string('email')->unique();
$table->string('name')->index();
$table->foreignId('account_id')->constrained()->cascadeOnDelete();
$table->string('status')->index();
```

### Rules:

- Use `->index()` on individual columns that need indexing
- Use `->unique()` on columns that must be unique
- Use `->nullable()->index()` for optional indexed columns
- **Never specify custom index names** — let Laravel generate them automatically
- **Never use old syntax**: `$table->index(['column'], 'custom_index_name')`

---

## Enums

### Database Enums are Prohibited

**Never use database enums** (MySQL ENUM type or Laravel's `enum()` column type).

### Why:

- Database enums are inflexible and difficult to modify
- They create coupling between database and application logic
- Testing and migrations become problematic

### Solution:

Use string or integer columns with validation against application-level enums:

**Application Enum (app/Enums/SyncStatus.php):**
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

**Model Validation:**
```php
$validated = $request->validate([
    'status' => ['required', Rule::enum(SyncStatus::class)],
]);
```

---

## Denormalization

Denormalization is allowed when it serves a clear performance or practical purpose.

### Rules:

- You may add `account_id` to any table even if the relationship is indirect
- Document why denormalization was chosen (comment in migration or commit message)
- Only denormalize when it prevents expensive joins or provides significant performance benefit
- Maintain data integrity constraints (foreign keys) even for denormalized fields

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

### Conventions:

- Use `snake_case` for all column names
- Foreign keys should follow the pattern `{table}_id` (singular table name)
- Boolean columns should be prefixed with `is_` (e.g., `is_active`, `is_deleted`)
- Timestamps should be `created_at` and `updated_at`
- Soft-delete column should be `deleted_at`

### Date & Time Column Naming

For comprehensive rules on datetime columns, including naming patterns (`*_at`, `*_on`, `*_time`, `*_epoch`), storage requirements, and Carbon casting, see [Dates & Time Handling](./dates.md).

### Bad Practices (Avoid):

- ❌ Compound foreign key names: `calendar_user_id_start_datetime`
- ❌ Camel case: `createdAt`, `userId`
- ❌ Abbreviations: `usr_id`, `acct_id`
- ❌ Custom naming schemes

### Good Practices:

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

### Aggregation Suffixes

When a column holds aggregated data (counts, sums, averages, etc.), use clear suffixes to indicate the operation:

- `_count` — number of items (e.g., `events_synced_count`)
- `_sum` — total sum (e.g., `revenue_sum`)
- `_avg` — average value (e.g., `rating_avg`)
- `_min` — minimum value (e.g., `price_min`)
- `_max` — maximum value (e.g., `price_max`)

**Why:** Ambiguous column names like `events_synced` could be JSON data, a count, or actual event records. Always be explicit.

**Good Examples:**
```php
$table->integer('events_synced_count')->default(0);
$table->unsignedInteger('user_count')->default(0);
```

**Bad Examples:**
```php
$table->integer('events_synced'); // Unclear: count? JSON? Event IDs?
$table->integer('revenue');        // Could be sum, average, or single value
```

---

## External IDs

### External ID Naming

For columns that store identifiers from external systems or third-party services, always use `ext_id`:

**Rule:** Use `ext_id` for any external identifier, regardless of the source system.

**Why:** This makes it immediately clear that a column contains data from an external system, not a local database reference.

**Examples:**
```php
$table->string('ext_id')->nullable();  // External ID from third-party system
```

---

## Public vs Internal ID Exposure

### Rule: Never Expose Database IDs Publicly

**Internal Code & Services:** Use `id` (the auto-increment primary key) for internal service-to-service communication.

**Public-Facing Interfaces:** Always reference records by `uuid` instead of `id`. Never expose database IDs to:
- REST APIs
- Admin portals
- User-facing applications
- Third-party integrations

**Why:** Database IDs are predictable, sequential, and reveal information about system scale. UUIDs are non-sequential, non-guessable, and safe to expose.

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
# Database Specifications

## Overview

Database design conventions for GRS. All migrations, schemas, and relationships must follow these rules.

---

## Table Naming & Organization

### Module-Based Prefixes

Tables must use a module prefix:

- Calendar: `calendar_credentials`, `calendar_sync_states`, `calendar_sync_logs`
- Email: `email_templates`, `email_schedules`
- CRM: `crm_contacts`, `crm_deals`

---

## Account as Tenant

Rules:

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

### Date & Time Column Naming

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

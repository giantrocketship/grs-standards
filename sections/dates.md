# Dates & Time Handling

## Overview

Proper date and time handling is critical for multi-timezone applications. This specification ensures consistency, prevents bugs, and maintains data integrity across the entire system.

---

## Core Principles

1. **Always store UTC** — All datetimes in the database are stored as UTC
2. **Always use Carbon** — Use `Illuminate\Carbon\Carbon` or `Illuminate\Carbon\CarbonImmutable` instead of native PHP `DateTime`
3. **Timezone awareness** — Convert to user timezone only for display/input
4. **Immutables when necessary** — Use `CarbonImmutable` for event-based data

---

## Column Naming Conventions

Use these naming patterns consistently across all tables:

| Pattern | Type | Example | Usage |
|---------|------|---------|-------|
| `*_at` | DateTime | `created_at`, `synced_at`, `expires_at` | Datetime stored as UTC |
| `*_on` | Date | `birth_date`, `scheduled_on`, `starts_on` | Date only (no time) |
| `*_time` | Time | `start_time`, `end_time` | Time only (no date) |
| `*_epoch` | Integer | `created_epoch`, `timestamp_epoch` | Unix timestamp (seconds) |

### Examples:

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

### Always Import Carbon

```php
use Illuminate\Support\Carbon;
// or for immutable:
use Illuminate\Support\Carbon\CarbonImmutable;
```

### Mutable vs Immutable

**Use `Carbon` (mutable) for:**
- Business logic where state changes are expected
- Calculations and transformations
- Legacy code compatibility

**Use `CarbonImmutable` for:**
- Domain events
- Value objects
- Event sourcing scenarios
- Data that should not be modified

### Creating Carbon Instances

```php
// Parse from string
$carbon = Carbon::parse('2025-12-23 14:30:00');

// From timestamp
$carbon = Carbon::createFromTimestamp(1703334600);

// Now (current UTC time)
$now = Carbon::now('UTC');
// or simply
$now = Carbon::now();

// Specific date/time
$carbon = Carbon::create(2025, 12, 23, 14, 30, 0, 'UTC');
```

---

## Database Storage Rules

### Always Store UTC

Every datetime column in the database must contain UTC time, **never** local timezone.

```php
// ✅ CORRECT: Always store as UTC
$model->expires_at = Carbon::now('UTC')->addDays(30);
//or
$model->expires_at = Carbon::now()->addDays(30);

// ❌ WRONG: Never store in user's timezone
$model->expires_at = Carbon::now('Europe/London')->addDays(30);
```

### Casting to Carbon

Ensure all datetime columns are cast to Carbon in your models:

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

Never store user timezone in the database. Always convert when displaying:

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

Store user timezone preference as a string, not derived from the app:

```php
class User extends Model
{
    protected $casts = [
        'timezone' => 'string', // e.g., 'America/New_York', 'Europe/London'
    ];
}
```

Valid timezone strings are from PHP's supported timezones (e.g., `'UTC'`, `'America/New_York'`, `'Europe/London'`).

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

When using soft deletes, the `deleted_at` column follows the same UTC rules:

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
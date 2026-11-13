# Caching Specifications

## Overview

Caching rules for GRS. **Caching is a last resort**, used only when there is no better choice.

These rules cover application caching. Framework-level caches are a different thing and are fine.

---

## Core Principles

- **Caching is a last resort**
- **Fix the code, the query, or the schema first**
- **Do not add a cache speculatively, or "for performance", without a measured problem**
- **Invalidation and expiry are part of the design** — get them wrong and you ship stale data and bugs that are miserable to debug
- **An application cache needs explicit approval in the pull request**

---

## Fix the Real Problem First

Caching is often used to hide bad code and bad database design. A slow page or a slow command is usually one of these:

- A missing index
- An N+1 query
- An inefficient query
- Work that should not be on the hot path at all

Fix that. Add the index, eager-load the relationship, rewrite the query, do less work, or move the work to a job. Do not put a cache in front of it and call the problem solved.

Cache invalidation and expiry are a nightmare when they are done wrong. The cache serves stale data, the bug shows up far from the write that should have cleared it, and nobody can tell whether they are looking at a logic bug or a cache bug. That is why a cache is never the first tool.

---

## Do Not Add Application Caching Speculatively

Do not add any of the following unless the bar in [When a Cache Is Actually Required](#when-a-cache-is-actually-required) is met:

- `Cache::remember()`, `Cache::rememberForever()`, `Cache::put()` / `Cache::get()`, and the rest of the `Cache` facade used to store application data
- Query result caching (a query wrapped in `Cache::remember`, or a package that caches builder results)
- Memoized lookups that outlive the current call: static properties, long-lived service properties, Octane worker state
- Cache-aside layers (read from cache, fill on miss, write back on change), including caching a controller or HTTP response to avoid repeating work
- Model caching packages

A local variable that holds a value for the rest of one method is not a cache. Do not "optimize" a method that is not measured.

Laravel Octane reuses workers and service instances. A static property or a property on a long-lived service, used so a lookup "only hits the database once", is a cache. It will go stale or leak across requests. Do not add one. Services stay stateless; see [Code Structuring Specifications](./code-structure.md).

---

## Framework Caches Are Fine

These are framework machinery. Use them:

- Config, route, and view caching at deploy (`php artisan config:cache`, `route:cache`, `view:cache`)
- Queues, including unique jobs and overlap locks that the queue system owns
- Sessions
- Horizon

Redis (or whatever the cache store is) used as the queue or session driver is fine. The same store used to hide a slow query is application caching and follows the rules above.

`Cache::lock()` and the rate limiter are coordination, not a cache of application data. This rule does not restrict them.

---

## When a Cache Is Actually Required

Add an application cache only when there is no better choice, and only with all of the following in the pull request:

1. **Show the measurement.** A profiler trace, query log, or timing that shows the cost. "It might be slow" and "for performance" are not a measurement.
2. **Document why no better option exists.** Say what you ruled out: an index, a query change, eager loading, doing less work, a job. If one of those fixes it, do that instead.
3. **Define the invalidation and expiry strategy explicitly.** Name the key, what writes it, what deletes or overwrites it, and the TTL. "It expires eventually" is not a strategy. If you cannot say what clears the key when the underlying data changes, you cannot add the cache.
4. **Keep it in one obvious place.** One class or one method owns that key. Do not scatter `Cache::remember` through controllers, models, and jobs.
5. **Get explicit approval in the PR.** Call the cache out in the pull request description. A reviewer approves the cache itself, not only the feature around it.

Until that approval is in the PR, the cache does not land.

---

## Examples

Bad — a cache hiding a missing index:

```php
// The query is slow because (account_id, status) is not indexed.
// Add the index. Do not cache the result.
$open = Cache::remember("account:{$accountId}:open-tickets", 300, function () use ($accountId) {
    return Ticket::query()
        ->where('account_id', $accountId)
        ->where('status', 'open')
        ->get();
});
```

Good — fix the schema, then run the query:

```php
$table->index(['account_id', 'status']);
```

```php
$open = Ticket::query()
    ->where('account_id', $accountId)
    ->where('status', 'open')
    ->get();
```

Bad — a memoized lookup hiding an N+1:

```php
$tickets = Ticket::query()->where('account_id', $accountId)->get();

foreach ($tickets as $ticket) {
    $ticket->assignee_name = Cache::remember(
        "user:{$ticket->user_id}:name",
        3600,
        fn () => User::query()->find($ticket->user_id)?->name,
    );
}
```

Good — eager load:

```php
$tickets = Ticket::query()
    ->where('account_id', $accountId)
    ->with('assignee')
    ->get();
```

Bad — a forever cache with no invalidation, added "for performance":

```php
public function settings(Account $account): array
{
    return Cache::rememberForever(
        "account:{$account->id}:settings",
        fn () => $account->settings()->pluck('value', 'key')->all(),
    );
}
```

Nothing deletes that key when a setting changes. Callers will read stale settings, and the failure will not point back at this method. Do not merge this. If a measurement later shows the query itself is the problem and it cannot be fixed with an index or a simpler query, come back with the measurement, the reason, the invalidation plan, a single owner for the key, and explicit approval — not with `rememberForever` and a hope.

---

## Summary Checklist

Before adding any application cache:

- [ ] The cost is measured, and the measurement is in the PR
- [ ] An index, a query change, eager loading, less work, or a job was ruled out, and the PR says why
- [ ] The key, the writer, the invalidation, and the TTL are defined
- [ ] The cache lives in one obvious place
- [ ] The PR asks for explicit approval of the cache
- [ ] Config, route, view, queue, session, and Horizon caches were not mistaken for the thing this rule forbids

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

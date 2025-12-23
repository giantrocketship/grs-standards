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

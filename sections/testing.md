# Testing Specifications

## Overview

Testing standards for GRS using **Pest**.

---

## Required Testing Rules

- All tests are Pest tests; PHPUnit tests are forbidden
- Tests must not extend PHPUnit `TestCase`
- Tests must not use `RefreshDatabase` (handled by Pest)
- Tests must not use seeders
- All test data must be created via factories (no `Model::create()`, `$model->save()`, or `DB::` facade usage)

---

## Test Types

### Unit Tests

- **No database access** — never use `RefreshDatabase` or `DatabaseTransactions`
- **No HTTP requests**
- **No third-party service calls**
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

- **Access database** (handled by Pest)
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

- **Access database** (handled by Pest)
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

External HTTP calls are **forbidden** in unit tests. Feature and integration tests may call external services when required; label those tests with the `external` group:

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
- Use datasets or helper functions in `tests/Helpers` that rely on factories (seeders are forbidden)

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
│   ├── Models/
│   ├── Services/
│   └── ...
├── Integration/
│   ├── Services/
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

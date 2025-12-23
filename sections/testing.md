# Testing Specifications

## Overview

Testing is a critical part of our development process. This document defines the standards for writing tests in the GRS project using **Laravel Pest**.

All tests must be written following these specifications to ensure consistency, maintainability, and reliability across the codebase.

**Documentation Reference**: [Pest Documentation](https://pestphp.com/docs/getting-started)

---

## Test Types

### Unit Tests

**Purpose**: Test individual components in isolation without external dependencies.

**Characteristics**:
- Do not hit the database
- Do not make HTTP requests
- Focus on a single function or method
- Fast execution

**Location**: `tests/Unit/`

### Feature Tests

**Purpose**: Test application features at the HTTP level without testing the entire end-to-end flow.

**Characteristics**:
- Make HTTP requests to application endpoints
- May access the database
- Refresh database between tests
- Test a single feature or workflow
- Smaller scope than integration tests

**Location**: `tests/Feature/`

### Integration Tests

**Purpose**: Test end-to-end workflows involving multiple components, services, and systems.

**Characteristics**:
- Test complete user journeys
- Span multiple features or modules
- Refresh database between tests
- Larger scope than feature tests
- May involve external services

**Location**: `tests/Integration/`

---

## Test Functions

### Preferred Syntax

**Prefer `test()` for defining test cases**:

```php
test('user can create a new article', function () {
    // test code
});
```

You may use `it()` as an alternative, but `test()` is preferred across the codebase:

```php
// Acceptable, but test() is preferred
it('user can create a new article', function () {
    // test code
});
```

---

## External HTTP Calls

When a test makes real HTTP calls to external services (not your application), it **must** be labeled with the `external` group:

```php
test('fetches data from external API', function () {
    // HTTP call to external service
})->group('external');
```

This allows external tests to be skipped or run separately.

---

## Test Datasets

Reusable test data should be stored in dataset files following Pest conventions.

**Location**: `tests/Datasets/FeatureNameDataset.php`

**Example**:

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

**Usage**:

```php
test('creates article with valid data', function (array $data) {
    $response = postJson('/articles', $data);
    $response->assertSuccessful();
})->with('valid articles');
```

---

## Controller Testing

When testing controllers, follow these conventions:

### HTTP Test Functions

Use Pest's built-in HTTP functions for making requests with `action()`:

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

When testing controller methods directly, use `action()` with the controller class and method name:

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

**Do not use `route()` function** for testing:

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

**Key Points**:
- Use `Mockery::mock()` to create mocks
- Use `shouldReceive()` to define mock expectations
- Use `andReturn()` to specify return values
- Always verify the behavior you care about
- Clean up mocks in tearDown or use Mockery's automatic cleanup

---

## HTTP Request Mocking

When testing code that makes HTTP requests to external services, use Laravel's `Http::fake()` to mock the responses without making real network calls.

### Basic HTTP Mocking

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

Verify that your code made the expected HTTP requests:

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

### Mocking Multiple Endpoints

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

### Asserting Request Count

```php
test('makes correct number of API calls', function () {
    Http::fake();

    app(BatchProcessor::class)->process(10);

    Http::assertSentCount(10);
});
```

### Testing Error Responses

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

**Key Points**:
- Use `Http::fake()` to prevent real network requests
- Use wildcard patterns (`*`) for dynamic endpoints
- Assert requests with `Http::assertSent()` to verify correct API calls
- Use `Http::assertSentCount()` to verify the number of requests
- Return appropriate HTTP status codes to test error handling

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

Database is automatically refreshed between tests — no manual cleanup needed.

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

- Use `test()` for all test functions
- Organize tests by type: Unit, Feature, Integration
- Use `expect()` for all assertions
- Use `action()` and HTTP functions for controller testing
- Label external HTTP calls with `external` group
- Use datasets in `tests/Datasets/` for reusable test data
- Follow Mockery conventions for mocking dependencies
- Use `Http::fake()` to mock external HTTP requests
- Assert HTTP requests with `Http::assertSent()` and `Http::assertSentCount()`
- Keep tests focused and isolated
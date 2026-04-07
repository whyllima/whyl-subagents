---
name: endpoint-builder
description: Creates Laravel 13 API routes in routes/api/v1.php using Route::controller() grouping with per-route permission middleware. JWT/Sanctum support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Endpoint Builder

Creates versioned API routes in `routes/api/v1.php`.

## Before Starting

### 1. Check for ACL
```bash
ls app/Models/Module.php 2>/dev/null
```
**If ACL exists:** Add per-route permission middleware.
**If NO ACL:** Routes without middleware.

### 2. Check route file exists
```bash
ls routes/api/v1.php 2>/dev/null
```
If not exists, create it. Ensure `routes/api.php` has only infrastructure:
```php
// routes/api.php
Route::get('/health', fn () => response()->json(['status' => 'ok']));
```

Ensure `bootstrap/app.php` registers versioned routes:
```php
->withRouting(
    api: __DIR__.'/../routes/api.php',
    apiPrefix: 'api',
    then: function () {
        Route::middleware('api')->prefix('api/v1')->group(
            base_path('routes/api/v1.php')
        );
    },
)
```

## Route Pattern

### With ACL
```php
use App\Http\Controllers\{Domain}\{Entity}Controller;

Route::controller({Entity}Controller::class)->prefix('{entities}')->group(function () {
    Route::get('/', 'index')->middleware('permission:index-{entity}');
    Route::get('/{entity}', 'show')->middleware('permission:show-{entity}');
    Route::post('/', 'store')->middleware('permission:store-{entity}');
    Route::put('/{entity}', 'update')->middleware('permission:update-{entity}');
    Route::delete('/{entity}', 'destroy')->middleware('permission:delete-{entity}');
});
```

### Without ACL
```php
use App\Http\Controllers\{Domain}\{Entity}Controller;

Route::controller({Entity}Controller::class)->prefix('{entities}')->group(function () {
    Route::get('/', 'index');
    Route::get('/{entity}', 'show');
    Route::post('/', 'store');
    Route::put('/{entity}', 'update');
    Route::delete('/{entity}', 'destroy');
});
```

### Custom Actions
```php
Route::post('/{entity}/toggle', 'toggle')->middleware('permission:update-{entity}');
Route::get('/{entity}/export', 'export')->middleware('permission:export-{entity}');
Route::post('/{entity}/duplicate', 'duplicate')->middleware('permission:store-{entity}');
```

### Nested Resources
```php
Route::controller(FlowLogController::class)->group(function () {
    Route::get('/flows/{flowUuid}/logs', 'indexByFlow')->middleware('permission:show-flow');
    Route::get('/flows/{flowUuid}/logs/stats', 'stats')->middleware('permission:show-flow');
    Route::get('/flow-logs', 'index')->middleware('permission:index-flow');
    Route::get('/flow-logs/{uuid}', 'show')->middleware('permission:show-flow');
});
```

## Auth Wrapping

All protected routes inside JWT middleware group:
```php
// routes/api/v1.php

// Public routes
Route::controller(AuthController::class)->prefix('auth')->group(function () {
    Route::post('/', 'login');
    Route::post('/forgot-password', 'forgotPassword');
    Route::post('/reset-password', 'resetPassword');
});

// Protected routes
Route::middleware('jwt')->group(function () {
    Route::controller(AuthController::class)->prefix('auth')->group(function () {
        Route::get('/', 'me');
        Route::delete('/', 'logout');
        Route::put('/', 'refresh');
        Route::put('/profile', 'updateProfile');
    });

    // Feature routes here...
});
```

## Conventions

- **Always** add `use` import at top of file
- **Route parameter** uses model binding: `{entity}` resolves to UUID
- **Permission format:** `{action}-{module}` (e.g. `index-flow`, `store-user`)
- **Prefix** is plural kebab-case: `flows`, `flow-groups`, `platform-connections`
- **All** protected routes inside `Route::middleware('jwt')` group

## Workflow

1. Check for ACL (ls app/Models/Module.php)
2. Read existing `routes/api/v1.php` (or create if missing)
3. Add `use` import for the Controller
4. Append route group inside the JWT middleware group
5. Run `vendor/bin/pint --dirty`

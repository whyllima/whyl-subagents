---
name: endpoint-builder
description: Creates Laravel API routes using Route::controller() grouping. JWT/Sanctum support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Endpoint Builder

Creates: API Routes (assumes controller exists)

## Before Starting - Check Auth Type

```bash
grep -l "tymon/jwt-auth" composer.json 2>/dev/null
```

**If JWT exists:** Use `auth:api` or `jwt` middleware.
**If NO JWT:** Use `auth:sanctum` middleware.

## Import Pattern

**ALWAYS use `use` statement at top of file, NEVER inline namespace:**

```php
// ✅ CORRECT - import at top
use App\Http\Controllers\{Entity}Controller;

// ❌ WRONG - never use inline
Route::controller(\App\Http\Controllers\{Entity}Controller::class)...
```

## Route Pattern

```php
Route::controller({Entity}Controller::class)->prefix('{entities}')->group(function () {
    Route::get('/', 'index');
    // Custom GET routes BEFORE /{entity}
    Route::get('/with-relations', 'getWithRelations');
    Route::get('/{entity}', 'show');
    Route::post('/', 'store');
    Route::put('/{entity}', 'update');
    Route::delete('/{entity}', 'destroy');
});
```

### With Auth (JWT)

```php
use App\Http\Controllers\{Entity}Controller;

Route::middleware('jwt')->group(function () {
    Route::controller({Entity}Controller::class)->prefix('{entities}')->group(function () {
        Route::get('/', 'index');
        Route::get('/{entity}', 'show');
        Route::post('/', 'store');
        Route::put('/{entity}', 'update');
        Route::delete('/{entity}', 'destroy');
    });
});
```

### With Auth (Sanctum)

```php
use App\Http\Controllers\{Entity}Controller;

Route::middleware('auth:sanctum')->group(function () {
    Route::controller({Entity}Controller::class)->prefix('{entities}')->group(function () {
        Route::get('/', 'index');
        Route::get('/{entity}', 'show');
        Route::post('/', 'store');
        Route::put('/{entity}', 'update');
        Route::delete('/{entity}', 'destroy');
    });
});
```

## Rules

- **ALWAYS** add `use` import at top of routes file
- **NEVER** use inline namespace (`\App\Http\Controllers\...`)
- Prefix: plural (`entities`)
- Param: singular (`{entity}`)
- Custom routes BEFORE `/{entity}`
- Method names as strings
- **JWT project:** use `jwt` middleware
- **Sanctum project:** use `auth:sanctum` middleware

## Workflow

1. **Check auth type** (grep tymon/jwt-auth composer.json)
2. Verify controller exists
3. Read routes/api.php
4. **Add `use` import** at top of file (check if already exists)
5. Add routes (with correct auth middleware)

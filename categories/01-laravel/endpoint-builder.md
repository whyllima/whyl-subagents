---
name: endpoint-builder
description: Creates Laravel API routes using Route::controller() grouping.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Endpoint Builder

Creates: API Routes (assumes controller exists)

## Pattern

```php
use App\Http\Controllers\{Entity}Controller;

Route::controller({Entity}Controller::class)->prefix('{entities}')->group(function () {
    Route::get('/', 'index');
    // Custom GET routes BEFORE /{entity}
    Route::get('/with-relations', 'getWithRelations');
    Route::get('/{entity}', 'show');
    Route::post('/', 'store');
    Route::put('/{entity}', 'update');
    Route::delete('/{entity}', 'destroy');
});

// With auth
Route::middleware('auth:sanctum')->group(function () {
    Route::controller({Entity}Controller::class)->prefix('{entities}')->group(...);
});
```

## Rules

- Prefix: plural (`entities`)
- Param: singular (`{entity}`)
- Custom routes BEFORE `/{entity}`
- Method names as strings

## Workflow

1. Verify controller exists
2. Read routes/api.php
3. Add routes + import

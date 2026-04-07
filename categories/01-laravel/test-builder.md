---
name: test-builder
description: Creates PHPUnit 12 feature tests for Laravel 13 API endpoints with #[Seed] attribute, DatabaseTransactions, domain folders, JWT/Sanctum support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Test Builder

Creates feature tests for API endpoints. PHPUnit ^12 compatible.

## Before Starting

Check auth type:
```bash
grep -l "jwt" config/auth.php 2>/dev/null
grep -l "sanctum" config/auth.php 2>/dev/null
```

Check if Factory exists:
```bash
find database/factories -name "{Entity}Factory.php" 2>/dev/null
```

## PHPUnit Feature Test

File: `tests/Feature/{Domain}/{Entity}ControllerTest.php`

```php
namespace Tests\Feature\{Domain};

use App\Models\{Domain}\{Entity};
use App\Models\Auth\User;
use Illuminate\Foundation\Testing\Attributes\Seed;
use Illuminate\Foundation\Testing\DatabaseTransactions;
use Tests\TestCase;

#[Seed]
class {Entity}ControllerTest extends TestCase
{
    use DatabaseTransactions;

    private User $user;
    private string $token;
    private string $baseUrl = '/api/v1/{entities}';

    protected function setUp(): void
    {
        parent::setUp();
        $this->user = User::factory()->create();
        $this->token = auth()->login($this->user);
        // Sanctum: $this->token = $this->user->createToken('test')->plainTextToken;
    }

    private function authHeader(): array
    {
        return ['Authorization' => "Bearer {$this->token}"];
    }

    public function test_index_returns_paginated_list(): void
    {
        {Entity}::factory()->count(3)->create();

        $response = $this->getJson($this->baseUrl, $this->authHeader());

        $response->assertOk()
            ->assertJsonStructure(['data' => [['uuid', 'name', 'created_at']]]);
    }

    public function test_show_returns_single_entity(): void
    {
        $entity = {Entity}::factory()->create();

        $response = $this->getJson("{$this->baseUrl}/{$entity->uuid}", $this->authHeader());

        $response->assertOk()
            ->assertJsonFragment(['uuid' => $entity->uuid]);
    }

    public function test_store_creates_entity(): void
    {
        $data = ['name' => 'Test Entity'];

        $response = $this->postJson($this->baseUrl, $data, $this->authHeader());

        $response->assertCreated()
            ->assertJsonFragment(['name' => 'Test Entity']);

        $this->assertDatabaseHas('{entities}', ['name' => 'Test Entity']);
    }

    public function test_store_validates_required_fields(): void
    {
        $response = $this->postJson($this->baseUrl, [], $this->authHeader());

        $response->assertUnprocessable()
            ->assertJsonValidationErrors(['name']);
    }

    public function test_update_modifies_entity(): void
    {
        $entity = {Entity}::factory()->create();

        $response = $this->putJson(
            "{$this->baseUrl}/{$entity->uuid}",
            ['name' => 'Updated'],
            $this->authHeader()
        );

        $response->assertOk()
            ->assertJsonFragment(['name' => 'Updated']);
    }

    public function test_destroy_soft_deletes_entity(): void
    {
        $entity = {Entity}::factory()->create();

        $response = $this->deleteJson("{$this->baseUrl}/{$entity->uuid}", [], $this->authHeader());

        $response->assertOk();
        $this->assertSoftDeleted('{entities}', ['uuid' => $entity->uuid]);
    }

    public function test_unauthenticated_returns_401(): void
    {
        $response = $this->getJson($this->baseUrl);

        $response->assertUnauthorized();
    }
}
```

## Important

- **DatabaseTransactions** — does NOT truncate (safe for shared DBs)
- **Base URL** uses versioned path: `/api/v1/{entities}`
- **Test file** in domain folder: `tests/Feature/{Domain}/`
- **PHPUnit ^12** required
- **#[Seed]** attribute seeds database before test class

## Workflow

1. Determine domain name and auth type
2. Verify Factory exists (create with seeder-builder if not)
3. Create test in `tests/Feature/{Domain}/`
4. Run `php artisan test --filter={Entity}ControllerTest`

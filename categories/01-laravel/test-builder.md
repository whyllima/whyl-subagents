---
name: test-builder
description: Creates PHPUnit feature tests for Laravel API endpoints using DatabaseTransactions. JWT/Sanctum support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Test Builder

Creates: PHPUnit Feature Tests (uses DatabaseTransactions, NOT RefreshDatabase)

## Before Starting - Check Auth Type

```bash
grep -l "tymon/jwt-auth" composer.json 2>/dev/null
```

**If JWT exists:** Use JWT token authentication in tests.
**If NO JWT:** Use Sanctum `actingAs()`.

## Pattern (Sanctum)

```php
class {Entity}ControllerTest extends TestCase
{
    use DatabaseTransactions;

    private User $user;
    private string $baseUrl = '/api/v1/{entities}';

    protected function setUp(): void
    {
        parent::setUp();
        $this->user = User::factory()->create();
    }

    public function test_index_returns_all(): void
    {
        {Entity}::factory()->count(3)->create();
        $this->actingAs($this->user, 'sanctum')
            ->getJson($this->baseUrl)
            ->assertOk()
            ->assertJsonStructure(['data' => [['uuid', 'name']]]);
    }

    // ... other tests with actingAs($this->user, 'sanctum')

    public function test_unauthenticated_returns_401(): void
    {
        $this->getJson($this->baseUrl)->assertUnauthorized();
    }
}
```

## Pattern (JWT)

```php
use Tymon\JWTAuth\Facades\JWTAuth;

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
        $this->token = JWTAuth::fromUser($this->user);
    }

    public function test_index_returns_all(): void
    {
        {Entity}::factory()->count(3)->create();
        $this->withHeader('Authorization', "Bearer {$this->token}")
            ->getJson($this->baseUrl)
            ->assertOk()
            ->assertJsonStructure(['data' => [['uuid', 'name']]]);
    }

    public function test_show_returns_single(): void
    {
        $e = {Entity}::factory()->create();
        $this->withHeader('Authorization', "Bearer {$this->token}")
            ->getJson("{$this->baseUrl}/{$e->uuid}")
            ->assertOk()
            ->assertJsonFragment(['uuid' => $e->uuid]);
    }

    public function test_store_creates(): void
    {
        $this->withHeader('Authorization', "Bearer {$this->token}")
            ->postJson($this->baseUrl, ['name' => 'Test'])
            ->assertCreated();
        $this->assertDatabaseHas('{entities}', ['name' => 'Test']);
    }

    public function test_store_validates(): void
    {
        $this->withHeader('Authorization', "Bearer {$this->token}")
            ->postJson($this->baseUrl, [])
            ->assertUnprocessable()
            ->assertJsonValidationErrors(['name']);
    }

    public function test_update_modifies(): void
    {
        $e = {Entity}::factory()->create();
        $this->withHeader('Authorization', "Bearer {$this->token}")
            ->putJson("{$this->baseUrl}/{$e->uuid}", ['name' => 'Updated'])
            ->assertOk();
        $this->assertDatabaseHas('{entities}', ['uuid' => $e->uuid, 'name' => 'Updated']);
    }

    public function test_destroy_deletes(): void
    {
        $e = {Entity}::factory()->create();
        $this->withHeader('Authorization', "Bearer {$this->token}")
            ->deleteJson("{$this->baseUrl}/{$e->uuid}")
            ->assertOk();
        $this->assertSoftDeleted('{entities}', ['uuid' => $e->uuid]);
    }

    public function test_unauthenticated_returns_401(): void
    {
        $this->getJson($this->baseUrl)->assertUnauthorized();
    }
}
```

## Helper Method (JWT)

Add to TestCase or trait for cleaner tests:

```php
protected function authHeaders(): array
{
    return ['Authorization' => "Bearer {$this->token}"];
}

// Usage
$this->withHeaders($this->authHeaders())->getJson($this->baseUrl);
```

## Workflow

1. **Check auth type** (grep tymon/jwt-auth composer.json)
2. Read controller to understand methods
3. Create test file (JWT or Sanctum pattern)
4. Run `vendor/bin/pint --dirty`

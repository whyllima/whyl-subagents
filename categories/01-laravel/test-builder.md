---
name: test-builder
description: Creates PHPUnit feature tests for Laravel API endpoints using DatabaseTransactions.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Test Builder

Creates: PHPUnit Feature Tests (uses DatabaseTransactions, NOT RefreshDatabase)

## Pattern

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

    public function test_show_returns_single(): void
    {
        $e = {Entity}::factory()->create();
        $this->actingAs($this->user, 'sanctum')
            ->getJson("{$this->baseUrl}/{$e->uuid}")
            ->assertOk()
            ->assertJsonFragment(['uuid' => $e->uuid]);
    }

    public function test_store_creates(): void
    {
        $this->actingAs($this->user, 'sanctum')
            ->postJson($this->baseUrl, ['name' => 'Test'])
            ->assertCreated();
        $this->assertDatabaseHas('{entities}', ['name' => 'Test']);
    }

    public function test_store_validates(): void
    {
        $this->actingAs($this->user, 'sanctum')
            ->postJson($this->baseUrl, [])
            ->assertUnprocessable()
            ->assertJsonValidationErrors(['name']);
    }

    public function test_update_modifies(): void
    {
        $e = {Entity}::factory()->create();
        $this->actingAs($this->user, 'sanctum')
            ->putJson("{$this->baseUrl}/{$e->uuid}", ['name' => 'Updated'])
            ->assertOk();
        $this->assertDatabaseHas('{entities}', ['uuid' => $e->uuid, 'name' => 'Updated']);
    }

    public function test_destroy_deletes(): void
    {
        $e = {Entity}::factory()->create();
        $this->actingAs($this->user, 'sanctum')
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

## Workflow

1. Read controller to understand methods
2. Create test file
3. Run `vendor/bin/pint --dirty`

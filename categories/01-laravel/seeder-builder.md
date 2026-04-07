---
name: seeder-builder
description: Creates Laravel 13 Factories with #[UseModel] attribute and Seeders with Faker, domain-aware.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Seeder Builder

Creates Factory + Seeder for test/dev data.

## Factory (PHP Attributes — Laravel 13)

File: `database/factories/{Domain}/{Entity}Factory.php`

```php
namespace Database\Factories\{Domain};

use App\Models\{Domain}\{Entity};
use Illuminate\Database\Eloquent\Factories\Attributes\UseModel;
use Illuminate\Database\Eloquent\Factories\Factory;

#[UseModel({Entity}::class)]
class {Entity}Factory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => fake()->sentence(3),
            'slug' => fake()->unique()->slug(),
            'description' => fake()->paragraph(),
            'status' => fake()->randomElement(['active', 'inactive']),
        ];
    }

    public function active(): static
    {
        return $this->state(fn (array $attributes) => ['status' => 'active']);
    }

    public function inactive(): static
    {
        return $this->state(fn (array $attributes) => ['status' => 'inactive']);
    }
}
```

## Seeder

```php
namespace Database\Seeders;

use App\Models\{Domain}\{Entity};
use Illuminate\Database\Seeder;

class {Entity}Seeder extends Seeder
{
    public function run(): void
    {
        {Entity}::factory()->count(50)->create();
        {Entity}::factory()->active()->count(30)->create();
        {Entity}::factory()->inactive()->count(10)->create();
    }
}
```

## Factory Relationship Patterns

```php
// BelongsTo — reference existing or create
'category_uuid' => Category::factory(),

// Or reference existing records
'category_uuid' => Category::inRandomOrder()->first()?->uuid,
```

## Workflow

1. Determine domain name
2. Create Factory in `database/factories/{Domain}/`
3. Create Seeder in `database/seeders/`
4. Run `vendor/bin/pint --dirty`

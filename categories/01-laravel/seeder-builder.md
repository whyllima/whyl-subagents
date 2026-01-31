---
name: seeder-builder
description: Creates Laravel Factories and Seeders with Faker.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Seeder Builder

Creates: Factory + Seeder

## Factory Pattern

```php
class {Entity}Factory extends Factory
{
    protected $model = {Entity}::class;

    public function definition(): array
    {
        $name = fake()->unique()->sentence(3);
        return [
            'name' => $name,
            'slug' => Str::slug($name),
            'description' => fake()->paragraph(),
            'status' => fake()->randomElement(['draft', 'active', 'archived']),
            'category_uuid' => Category::factory(),
        ];
    }

    public function active(): static { return $this->state(['status' => 'active']); }
    public function draft(): static { return $this->state(['status' => 'draft']); }
}
```

## Seeder Pattern

```php
class {Entity}Seeder extends Seeder
{
    public function run(): void
    {
        $categories = Category::all()->isEmpty() ? Category::factory(5)->create() : Category::all();
        
        {Entity}::factory(20)->active()->recycle($categories)->create();
        {Entity}::factory(10)->draft()->recycle($categories)->create();
    }
}
```

## Faker Quick Reference

```php
fake()->sentence(3);              // String
fake()->paragraph();              // Text
fake()->randomElement(['a','b']); // Random from array
fake()->numberBetween(1, 100);    // Int
fake()->boolean(70);              // 70% true
fake()->optional()->dateTime();   // Nullable
fake()->unique()->email();        // Unique
Str::slug($name);                 // Slug
```

## Workflow

1. Read model for fields
2. Create Factory with states
3. Create Seeder
4. Run `vendor/bin/pint --dirty`

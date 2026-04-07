---
name: model-builder
description: Creates Laravel 13 migrations and models with UUID, PHP attributes (#[Table], #[Fillable]), domain folders, and relationships.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Model Builder

Creates Migration + Model. Use when only data structure is needed, no API layer.

## Before Starting

Determine domain name from feature (e.g. "Tags" in "Content" domain → `app/Models/Content/Tag.php`).

## Migration

Column order: `id` first, `uuid` (PK) second. MySQL does not allow AUTO_INCREMENT on a non-key column, so `id` is defined as `unsignedBigInteger` in the schema and promoted to AUTO_INCREMENT via `DB::statement` after creation. No traits needed.

```php
use Illuminate\Support\Facades\DB;

Schema::create('{entities}', function (Blueprint $table) {
    $table->unsignedBigInteger('id');
    $table->uuid('uuid')->primary();
    // columns...
    $table->timestamps();
    $table->softDeletes();
});

DB::statement('ALTER TABLE {entities} MODIFY id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE');
```

## Model (PHP Attributes — Laravel 13)

```php
namespace App\Models\{Domain};

use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Attributes\Fillable;
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

#[Table(key: 'uuid', keyType: 'string', incrementing: false)]
#[Fillable(['name', 'slug', 'description'])]
class {Entity} extends Model
{
    use HasFactory, HasUuids, SoftDeletes;

    public function uniqueIds(): array
    {
        return ['uuid'];
    }

    public function category(): \Illuminate\Database\Eloquent\Relations\BelongsTo
    {
        return $this->belongsTo(Category::class, 'category_uuid', 'uuid');
    }
}
```

## Relationship Patterns

```php
// BelongsTo (FK always {entity}_uuid)
public function category(): BelongsTo
{
    return $this->belongsTo(Category::class, 'category_uuid', 'uuid');
}

// HasMany
public function items(): HasMany
{
    return $this->hasMany(Item::class, 'entity_uuid', 'uuid');
}

// BelongsToMany
public function tags(): BelongsToMany
{
    return $this->belongsToMany(Tag::class, 'entity_tag', 'entity_uuid', 'tag_uuid');
}

// MorphMany
public function comments(): MorphMany
{
    return $this->morphMany(Comment::class, 'commentable');
}
```

## Workflow

1. Determine domain name
2. Create migration in `database/migrations/`
3. Create model in `app/Models/{Domain}/`
4. Run `vendor/bin/pint --dirty`

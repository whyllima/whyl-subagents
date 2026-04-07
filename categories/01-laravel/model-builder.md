---
name: model-builder
description: Creates Laravel 13 migrations and models with UUID PK, PHP attributes, domain folders, FKs, composite keys, pivot tables, indexes, and relationships.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Model Builder

Creates Migration + Model. Use when only data structure is needed, no API layer.

## Before Starting

Determine domain name from feature (e.g. "Tags" in "Content" domain → `app/Models/Content/Tag.php`).

## Migration Patterns

### Standard Table

Column order: `id` first (unsignedBigInteger), `uuid` (PK) second. `MODIFY` for AUTO_INCREMENT after creation.

```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('{entities}', function (Blueprint $table) {
            $table->unsignedBigInteger('id');
            $table->uuid('uuid')->primary();
            // FK columns...
            // Business columns...
            $table->timestamps();
            $table->softDeletes();

            // Indexes
        });

        DB::statement('ALTER TABLE {entities} MODIFY id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE');
    }

    public function down(): void
    {
        Schema::dropIfExists('{entities}');
    }
};
```

### Foreign Key — Standard (cascade)

FK suffix always `_uuid`. Use `foreignUuid()->constrained()` shorthand:

```php
$table->foreignUuid('category_uuid')->constrained('categories', 'uuid')->cascadeOnDelete();
```

Or verbose (when you need more control):

```php
$table->uuid('category_uuid');
$table->foreign('category_uuid')->references('uuid')->on('categories')->onDelete('cascade');
```

### Foreign Key — Nullable (set null on delete)

```php
$table->foreignUuid('parent_uuid')->nullable()->constrained('{entities}', 'uuid')->nullOnDelete();
```

Or verbose:

```php
$table->uuid('parent_uuid')->nullable();
$table->foreign('parent_uuid')->references('uuid')->on('{entities}')->onDelete('set null');
```

### Foreign Key — Restrict (prevent delete)

```php
$table->foreignUuid('plan_uuid')->constrained('plans', 'uuid')->restrictOnDelete();
```

### Composite Unique Key

```php
$table->unique(['organization_uuid', 'slug']);

// Named (required when 3+ columns or long names):
$table->unique(['platform_connection_uuid', 'entity_type', 'entity_uuid'], 'pr_connection_type_entity_unique');
```

### Indexes

```php
// Single column
$table->index('status');

// Composite
$table->index(['user_uuid', 'status']);

// Named composite (recommended for complex)
$table->index(['user_uuid', 'created_at'], 'idx_entities_user_created');
```

### Full Example — Table with FKs, Indexes, Composite Unique

```php
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('platform_metrics', function (Blueprint $table) {
            $table->unsignedBigInteger('id');
            $table->uuid('uuid')->primary();
            $table->foreignUuid('action_uuid')->nullable()->constrained('flow_actions', 'uuid')->nullOnDelete();
            $table->string('platform');
            $table->string('entity_type');
            $table->string('entity_uuid');
            $table->date('metric_date');
            $table->unsignedBigInteger('impressions')->default(0);
            $table->unsignedBigInteger('clicks')->default(0);
            $table->decimal('ctr', 8, 4)->default(0);
            $table->json('extra')->nullable();
            $table->timestamps();

            $table->unique(['platform', 'entity_type', 'entity_uuid', 'metric_date'], 'pm_platform_entity_date_unique');
            $table->index('metric_date');
            $table->index('action_uuid');
        });

        DB::statement('ALTER TABLE platform_metrics MODIFY id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE');
    }

    public function down(): void
    {
        Schema::dropIfExists('platform_metrics');
    }
};
```

### Pivot Table (no id/uuid, composite PK)

Pivot tables use composite primary key instead of id/uuid:

```php
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('entity_tag', function (Blueprint $table) {
            $table->foreignUuid('entity_uuid')->constrained('entities', 'uuid')->cascadeOnDelete();
            $table->foreignUuid('tag_uuid')->constrained('tags', 'uuid')->cascadeOnDelete();
            $table->primary(['entity_uuid', 'tag_uuid']);
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('entity_tag');
    }
};
```

### Pivot Table with Extra Columns

```php
Schema::create('role_user', function (Blueprint $table) {
    $table->foreignUuid('user_uuid')->constrained('users', 'uuid')->cascadeOnDelete();
    $table->foreignUuid('role_uuid')->constrained('roles', 'uuid')->cascadeOnDelete();
    $table->string('assigned_by')->nullable();
    $table->primary(['user_uuid', 'role_uuid']);
    $table->timestamps();
});
```

### High-Volume Table (BIGINT PK only, no UUID)

For tables with very high write volume (100k+ rows/day), use BIGINT PK only for performance:

```php
Schema::create('audit_logs', function (Blueprint $table) {
    $table->id(); // BIGINT auto-increment PK
    $table->foreignUuid('user_uuid')->nullable()->constrained('users', 'uuid')->nullOnDelete();
    $table->string('action');
    $table->string('entity_type');
    $table->uuid('entity_uuid')->nullable();
    $table->json('old_values')->nullable();
    $table->json('new_values')->nullable();
    $table->timestamp('created_at', 3); // millisecond precision
    $table->index(['entity_type', 'entity_uuid']);
    $table->index('user_uuid');
});
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
#[Fillable(['name', 'slug', 'description', 'category_uuid'])]
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

// BelongsToMany (pivot table)
public function tags(): BelongsToMany
{
    return $this->belongsToMany(Tag::class, 'entity_tag', 'entity_uuid', 'tag_uuid');
}

// BelongsToMany with pivot columns
public function roles(): BelongsToMany
{
    return $this->belongsToMany(Role::class, 'role_user', 'user_uuid', 'role_uuid')
        ->withPivot('assigned_by')
        ->withTimestamps();
}

// Self-referencing (parent/children)
public function parent(): BelongsTo
{
    return $this->belongsTo(static::class, 'parent_uuid', 'uuid');
}

public function children(): HasMany
{
    return $this->hasMany(static::class, 'parent_uuid', 'uuid');
}

// MorphMany
public function comments(): MorphMany
{
    return $this->morphMany(Comment::class, 'commentable');
}
```

## FK Naming Convention

| Type | Format | Example |
|------|--------|---------|
| Standard FK | `{entity}_uuid` | `category_uuid`, `user_uuid` |
| Self-reference | `parent_uuid` | `parent_uuid` |
| Polymorphic | `{relation}_type` + `{relation}_uuid` | `commentable_type`, `commentable_uuid` |

## Workflow

1. Determine domain name
2. Determine table type (standard / pivot / high-volume)
3. Create migration in `database/migrations/`
4. Create model in `app/Models/{Domain}/`
5. Run `vendor/bin/pint --dirty`

---
name: model-builder
description: Creates Laravel migrations and models with UUID, traits, and relationships.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Model Builder

Creates: Migration + Model (no API layer)

## Patterns

### Migration

MySQL does not allow AUTO_INCREMENT on a non-key column when another column is the PRIMARY KEY.
Use `DB::statement()` to add `id` as `BIGINT UNSIGNED AUTO_INCREMENT UNIQUE` after creating the table.

```php
use Illuminate\Support\Facades\DB;

Schema::create('{entities}', function (Blueprint $table) {
    $table->uuid('uuid')->primary();
    // DO NOT add 'id' here — added via DB::statement below
    $table->string('name');
    // columns...
    $table->foreignUuid('category_uuid')->nullable()->constrained('categories', 'uuid');
    $table->timestamps();
    $table->softDeletes();
    $table->index(['status', 'created_at']);
});

// Add auto-increment id as UNIQUE key (MySQL requires AUTO_INCREMENT to be a key)
DB::statement('ALTER TABLE {entities} ADD COLUMN id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE AFTER uuid');
```

### Pivot Table

Pivot tables do NOT need the `id` column.

```php
Schema::create('{entity_relation}', function (Blueprint $table) {
    $table->foreignUuid('entity_uuid')->constrained('entities', 'uuid')->cascadeOnDelete();
    $table->foreignUuid('relation_uuid')->constrained('relations', 'uuid')->cascadeOnDelete();
    $table->primary(['entity_uuid', 'relation_uuid']);
    $table->timestamps();
});
```

### Model

Do NOT use the `AutoIncrementId` trait — `id` is handled by MySQL AUTO_INCREMENT.

```php
class {Entity} extends Model
{
    use HasFactory, HasUuids, SoftDeletes;

    protected $primaryKey = 'uuid';
    public $incrementing = false;
    protected $keyType = 'string';
    protected $fillable = ['name', 'category_uuid'];

    public function uniqueIds(): array { return ['uuid']; }

    public function category() { return $this->belongsTo(Category::class, 'category_uuid', 'uuid'); }
    public function items() { return $this->hasMany(Item::class, 'entity_uuid', 'uuid'); }
    public function tags() { return $this->belongsToMany(Tag::class, 'entity_tag', 'entity_uuid', 'tag_uuid', 'uuid', 'uuid'); }
}
```

## Workflow

1. Create migration (uuid PK + DB::statement for auto-increment id)
2. Create model with traits and relationships (no AutoIncrementId)
3. Run `vendor/bin/pint --dirty`

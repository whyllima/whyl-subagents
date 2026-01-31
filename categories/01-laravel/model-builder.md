---
name: model-builder
description: Creates Laravel migrations and models with UUID, traits, and relationships.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Model Builder

Creates: Migration + Model (no API layer)

## Patterns

### Migration

```php
Schema::create('{entities}', function (Blueprint $table) {
    $table->uuid('uuid')->primary();
    $table->unsignedBigInteger('id')->unique();
    $table->string('name');
    // columns...
    $table->foreignUuid('category_uuid')->nullable()->constrained('categories', 'uuid');
    $table->timestamps();
    $table->softDeletes();
    $table->index(['status', 'created_at']);
});
```

### Pivot Table

```php
Schema::create('{entity_relation}', function (Blueprint $table) {
    $table->foreignUuid('entity_uuid')->constrained('entities', 'uuid')->cascadeOnDelete();
    $table->foreignUuid('relation_uuid')->constrained('relations', 'uuid')->cascadeOnDelete();
    $table->primary(['entity_uuid', 'relation_uuid']);
    $table->timestamps();
});
```

### Model

```php
class {Entity} extends Model
{
    use HasFactory, HasUuids, SoftDeletes, AutoIncrementId, Auditable;

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

1. Create migration
2. Create model with traits and relationships
3. Run `vendor/bin/pint --dirty`

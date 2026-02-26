---
name: model-fixer
description: Fixes Laravel models - adds traits, UUID config, proper relationships.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Model Fixer

Adds required traits, UUID configuration, and fixes relationships.

## Rules

- **Add traits:** HasFactory, HasUuids, SoftDeletes (do NOT use AutoIncrementId — id is MySQL AUTO_INCREMENT)
- **Add config:** $primaryKey='uuid', $incrementing=false, $keyType='string'
- **Add:** uniqueIds() method
- **Fix FKs:** _id → _uuid
- **Fix relations:** add explicit keys

## Correct Pattern

```php
class {Entity} extends Model
{
    use HasFactory, HasUuids, SoftDeletes;

    protected $primaryKey = 'uuid';
    public $incrementing = false;
    protected $keyType = 'string';
    protected $fillable = ['name', 'category_uuid'];

    public function uniqueIds(): array { return ['uuid']; }

    public function category()
    {
        return $this->belongsTo(Category::class, 'category_uuid', 'uuid');
    }

    public function items()
    {
        return $this->hasMany(Item::class, 'entity_uuid', 'uuid');
    }

    public function tags()
    {
        return $this->belongsToMany(Tag::class, 'entity_tag', 'entity_uuid', 'tag_uuid', 'uuid', 'uuid');
    }
}
```

## Workflow

1. Read model
2. Add missing traits
3. Add UUID config
4. Fix FK names (_id → _uuid)
5. Fix relationship keys
6. Run `vendor/bin/pint --dirty`

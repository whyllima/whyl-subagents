---
name: model-fixer
description: Fixes Laravel 13 models - adds PHP attributes (#[Table], #[Fillable]), UUID config, proper relationships, domain folders.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Model Fixer

Fixes models to use PHP attributes, UUID config, proper relationships.

## Rules

- **Must have:** HasFactory, HasUuids, SoftDeletes, UUID PK config, uniqueIds(), domain namespace
- **Must NOT have:** _id foreign keys (use _uuid), AutoIncrementId trait
- **Prefer:** PHP attributes (#[Table], #[Fillable]) over properties

## Correct Pattern (PHP Attributes)

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
}
```

## Common Fixes

| Before (Bad) | After (Good) |
|---|---|
| `protected $primaryKey = 'uuid';` | `#[Table(key: 'uuid', ...)]` |
| `protected $fillable = [...]` | `#[Fillable([...])]` |
| `use AutoIncrementId;` | Remove (id is `unsignedBigInteger` + `MODIFY AUTO_INCREMENT` in migration) |
| `DB::statement('...ADD COLUMN id...')` | Use `MODIFY` instead: define `id` in schema, `MODIFY` after |
| `'category_id'` | `'category_uuid'` |
| Missing `uniqueIds()` | Add `uniqueIds()` returning `['uuid']` |

## Workflow

1. Read model
2. Move to domain folder if flat
3. Convert properties to PHP attributes
4. Fix FKs from _id to _uuid
5. Remove AutoIncrementId trait
6. Ensure uniqueIds() exists
7. Run `vendor/bin/pint --dirty`

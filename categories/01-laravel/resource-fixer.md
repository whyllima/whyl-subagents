---
name: resource-fixer
description: Fixes Laravel 13 Resources - explicit fields, ISO dates, whenLoaded, #[Collects] attribute, domain folders.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Resource Fixer

Fixes Resources to use explicit fields, ISO dates, conditional relations.

## Rules

- **Must have:** @mixin, explicit fields, toISOString(), whenLoaded(), domain namespace
- **Must NOT have:** parent::toArray(), id field, raw date formats
- **Collection:** use `#[Collects]` attribute (Laravel 13)

## Correct Resource Pattern

```php
namespace App\Http\Resources\{Domain};

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

/** @mixin \App\Models\{Domain}\{Entity} */
class {Entity}Resource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'uuid' => $this->uuid,
            'name' => $this->name,
            'created_at' => $this->created_at?->toISOString(),
            'updated_at' => $this->updated_at?->toISOString(),
            'category' => CategoryResource::make($this->whenLoaded('category')),
            'tags' => TagResource::collection($this->whenLoaded('tags')),
            'comments_count' => $this->whenCounted('comments'),
        ];
    }
}
```

## Correct Collection Pattern (PHP Attributes)

```php
namespace App\Http\Resources\{Domain};

use Illuminate\Http\Resources\Attributes\Collects;
use Illuminate\Http\Resources\Json\ResourceCollection;

#[Collects({Entity}Resource::class)]
class {Entity}Collection extends ResourceCollection {}
```

## Common Fixes

| Before (Bad) | After (Good) |
|---|---|
| `return parent::toArray($request)` | Explicit fields array |
| `'id' => $this->id` | `'uuid' => $this->uuid` |
| `'created_at' => $this->created_at` | `'created_at' => $this->created_at?->toISOString()` |
| `'category' => $this->category` | `CategoryResource::make($this->whenLoaded('category'))` |
| `public $collects = X::class` | `#[Collects(X::class)]` |

## Workflow

1. Read Resource
2. Fix namespace to domain folder
3. Replace parent::toArray with explicit fields
4. Fix dates to toISOString
5. Fix relations to whenLoaded
6. Update Collection to use #[Collects]
7. Run `vendor/bin/pint --dirty`

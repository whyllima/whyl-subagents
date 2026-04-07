---
name: resource-builder
description: Creates Laravel 13 API Resources and Collections with #[Collects] attribute, domain folders, explicit fields, and ISO dates.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Resource Builder

Creates JsonResource + ResourceCollection for JSON API responses.

## Resource (in `app/Http/Resources/{Domain}/`)

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

## Collection (PHP Attributes — Laravel 13)

```php
namespace App\Http\Resources\{Domain};

use Illuminate\Http\Resources\Attributes\Collects;
use Illuminate\Http\Resources\Json\ResourceCollection;

#[Collects({Entity}Resource::class)]
class {Entity}Collection extends ResourceCollection {}
```

## Rules

- **Always** explicit fields (NEVER `parent::toArray()`)
- **Dates** always `->toISOString()`
- **Relations** always `$this->whenLoaded('...')`
- **Counts** always `$this->whenCounted('...')`
- **@mixin** annotation for IDE support
- **No `id` field** — only `uuid`

## Workflow

1. Determine domain name
2. Create Resource in `app/Http/Resources/{Domain}/`
3. Create Collection in same directory
4. Run `vendor/bin/pint --dirty`

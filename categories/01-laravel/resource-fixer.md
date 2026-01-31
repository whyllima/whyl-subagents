---
name: resource-fixer
description: Fixes Laravel Resources - explicit fields, ISO dates, conditional relations.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Resource Fixer

Ensures Resource has explicit fields, ISO dates, and conditional relationships.

## Rules

- **Remove:** parent::toArray()
- **Add:** @mixin annotation
- **Use:** uuid (not id)
- **Dates:** ->toISOString() with null-safe (?->)
- **Relations:** whenLoaded()
- **Counts:** whenCounted()

## Correct Pattern

```php
/** @mixin \App\Models\{Entity} */
class {Entity}Resource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'uuid' => $this->uuid,
            'name' => $this->name,
            'status' => $this->status,
            'created_at' => $this->created_at?->toISOString(),
            'updated_at' => $this->updated_at?->toISOString(),
            
            'category' => CategoryResource::make($this->whenLoaded('category')),
            'items' => ItemResource::collection($this->whenLoaded('items')),
            'items_count' => $this->whenCounted('items'),
        ];
    }
}

class {Entity}Collection extends ResourceCollection
{
    public $collects = {Entity}Resource::class;
}
```

## Workflow

1. Read Resource and Model
2. Replace parent::toArray with explicit fields
3. Fix dates to ISO
4. Add whenLoaded for relations
5. Run `vendor/bin/pint --dirty`

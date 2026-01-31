---
name: repository-fixer
description: Fixes Laravel repositories - converts DB facade to Model queries.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Repository Fixer

Converts DB:: facade usage to Model::query() with Eloquent.

## Rules

- **Remove:** DB::table(), DB::select(), DB::raw()
- **Use:** Model::query(), whereHas(), with()
- **Must:** extend Repository, init $this->model

## Conversions

| DB Facade | Model Equivalent |
|-----------|------------------|
| `DB::table('x')` | `X::query()` |
| `->join('y', ...)` | `->with('y')` or `->whereHas('y')` |
| `DB::raw()` | Eloquent methods or scopes |

## Correct Pattern

```php
class {Entity}Repository extends Repository
{
    private const PER_PAGE = 15;
    
    public function __construct() { $this->model = new {Entity}(); }

    public function index(array $filters = []): LengthAwarePaginator
    {
        $query = {Entity}::query();
        
        if (!empty($filters['name'])) {
            $query->where('name', 'like', "%{$filters['name']}%");
        }
        if (!empty($filters['category'])) {
            $query->whereHas('category', fn($q) => $q->where('name', 'like', "%{$filters['category']}%"));
        }
        
        return $query->orderBy($filters['sort'] ?? 'created_at', 'desc')
            ->paginate($filters['per_page'] ?? self::PER_PAGE);
    }

    public function getWithRelations(string $uuid): {Entity}
    {
        return {Entity}::with(['category', 'tags'])->where('uuid', $uuid)->firstOrFail();
    }
}
```

## Workflow

1. Read repository
2. Find all DB:: usages
3. Convert to Model queries
4. Run `vendor/bin/pint --dirty`

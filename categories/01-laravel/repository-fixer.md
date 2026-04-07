---
name: repository-fixer
description: Fixes Laravel 13 repositories - converts DB facade to Model queries, domain folders.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Repository Fixer

Converts DB:: facade calls to Eloquent Model queries.

## Rules

- **Must have:** extends Repository, Model::query(), domain namespace
- **Must NOT have:** DB::table, DB::select, DB::raw, DB::statement (for queries)

## Correct Pattern

```php
namespace App\Repositories\{Domain};

use App\Models\{Domain}\{Entity};
use App\Repositories\Repository;
use Illuminate\Pagination\LengthAwarePaginator;

class {Entity}Repository extends Repository
{
    private const PER_PAGE = 15;

    public function __construct()
    {
        $this->model = new {Entity}();
    }

    public function index(array $filters = []): LengthAwarePaginator
    {
        $query = {Entity}::query();

        if (! empty($filters['name'])) {
            $query->where('name', 'like', "%{$filters['name']}%");
        }

        if (! empty($filters['category'])) {
            $query->whereHas('category', fn ($q) => $q->where('name', 'like', "%{$filters['category']}%"));
        }

        return $query->orderBy('created_at', 'desc')
            ->paginate($filters['per_page'] ?? self::PER_PAGE);
    }
}
```

## Common Fixes

| Before (Bad) | After (Good) |
|---|---|
| `DB::table('entities')` | `{Entity}::query()` |
| `DB::select("SELECT...")` | `{Entity}::where(...)->get()` |
| `DB::raw(...)` | Use Eloquent scopes or whereRaw() on Model |

## Workflow

1. Read repository
2. Fix namespace to domain folder
3. Replace all DB:: with Model queries
4. Run `vendor/bin/pint --dirty`

---
name: formrequest-fixer
description: Fixes Laravel 13 FormRequests - handles create/update with UUID ignore, domain folders.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# FormRequest Fixer

Fixes FormRequests to handle both create and update with UUID-based unique ignore.

## Rules

- **Must have:** authorize(), rules(), POST+PUT handling, array syntax, domain namespace
- **Must NOT have:** pipe syntax (`required|string`), missing update handling

## Correct Pattern

```php
namespace App\Http\Requests\{Domain};

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class {Entity}Request extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        $rules = [
            'name' => ['required', 'string', 'max:255'],
        ];

        if ($this->isMethod('put') || $this->isMethod('patch')) {
            $rules['name'][] = Rule::unique('{entities}', 'name')
                ->ignore($this->route('{entity}')->uuid, 'uuid');
        } else {
            $rules['name'][] = 'unique:{entities},name';
        }

        return $rules;
    }
}
```

## Common Fixes

| Before (Bad) | After (Good) |
|---|---|
| `'name' => 'required\|string'` | `'name' => ['required', 'string']` |
| No PUT handling | Add `isMethod('put')` with `Rule::unique()->ignore()` |
| `->ignore($this->route('entity'))` | `->ignore($this->route('{entity}')->uuid, 'uuid')` |

## Workflow

1. Read FormRequest
2. Fix namespace to domain folder
3. Convert pipe syntax to array
4. Add PUT/PATCH handling if missing
5. Run `vendor/bin/pint --dirty`

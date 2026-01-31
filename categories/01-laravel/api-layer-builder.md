---
name: api-layer-builder
description: Creates Laravel API layer (Repository, Service, FormRequest, Controller, Routes, Resource) for existing models.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# API Layer Builder

Creates: Repository → Service → FormRequest → Controller → Resource → Routes

## Efficiency

- **DO NOT** read base classes or existing files
- **Create directly** using patterns
- **Run pint once** at end

## Architecture

```
Controller (no logic) → Service (all logic) → Repository (Model only) → Model
Response always via Resource
```

## Patterns

### Repository

```php
class {Entity}Repository extends Repository
{
    private const PER_PAGE = 15;
    public function __construct() { $this->model = new {Entity}(); }

    public function index(array $filters = []): LengthAwarePaginator
    {
        $query = {Entity}::query();
        if (!empty($filters['name'])) $query->where('name', 'like', "%{$filters['name']}%");
        if (!empty($filters['status'])) $query->where('status', $filters['status']);
        if (!empty($filters['category'])) {
            $query->whereHas('category', fn($q) => $q->where('name', 'like', "%{$filters['category']}%"));
        }
        return $query->orderBy($filters['sort'] ?? 'created_at', 'desc')
            ->paginate((int)($filters['per_page'] ?? self::PER_PAGE));
    }
}
```

### Service

```php
class {Entity}Service extends Service
{
    public function __construct() {
        $this->model = new {Entity}();
        $this->repository = new {Entity}Repository();
    }
    public function index(): JsonResource {
        try { return new {Entity}Collection($this->repository->index(request()->query())); }
        catch (Exception $e) { Log::channel('services')->error($e->getMessage()); return new ErrorResource($this->model); }
    }
    public function show({Entity} $e): JsonResource { return new {Entity}Resource($e); }
    public function store(array $d): JsonResource {
        try { return new {Entity}Resource({Entity}::create($d)); }
        catch (Exception $e) { Log::channel('services')->error($e->getMessage()); return new ErrorResource($this->model); }
    }
    public function update({Entity} $e, array $d): JsonResource {
        try { $e->update($d); return new {Entity}Resource($e->fresh()); }
        catch (Exception $e) { Log::channel('services')->error($e->getMessage()); return new ErrorResource($this->model); }
    }
    public function destroy({Entity} $e): JsonResource {
        try { $e->delete(); return new {Entity}Resource($e); }
        catch (Exception $e) { Log::channel('services')->error($e->getMessage()); return new ErrorResource($this->model); }
    }
}
```

### Controller (NO LOGIC)

```php
class {Entity}Controller extends Controller
{
    private {Entity}Service $service;
    public function __construct() { $this->service = new {Entity}Service(); }
    public function index(): JsonResource { return $this->service->index(); }
    public function show({Entity} $e): JsonResource { return $this->service->show($e); }
    public function store({Entity}Request $r): JsonResource { return $this->service->store($r->validated()); }
    public function update({Entity} $e, {Entity}Request $r): JsonResource { return $this->service->update($e, $r->validated()); }
    public function destroy({Entity} $e): JsonResource { return $this->service->destroy($e); }
}
```

### FormRequest

```php
class {Entity}Request extends FormRequest
{
    public function authorize(): bool { return true; }
    public function rules(): array {
        $rules = ['name' => ['required', 'string', 'max:255']];
        if ($this->isMethod('put') || $this->isMethod('patch')) {
            $rules['name'][] = Rule::unique('{entities}', 'name')->ignore($this->route('{entity}')->uuid, 'uuid');
        } else { $rules['name'][] = 'unique:{entities},name'; }
        return $rules;
    }
}
```

### Resource & Collection

```php
/** @mixin \App\Models\{Entity} */
class {Entity}Resource extends JsonResource {
    public function toArray(Request $r): array {
        return ['uuid' => $this->uuid, 'name' => $this->name,
            'created_at' => $this->created_at?->toISOString(),
            'category' => CategoryResource::make($this->whenLoaded('category'))];
    }
}
class {Entity}Collection extends ResourceCollection { public $collects = {Entity}Resource::class; }
```

### Routes

```php
Route::controller({Entity}Controller::class)->prefix('{entities}')->group(function () {
    Route::get('/', 'index');
    Route::get('/{entity}', 'show');
    Route::post('/', 'store');
    Route::put('/{entity}', 'update');
    Route::delete('/{entity}', 'destroy');
});
```

## Workflow

1. Verify model exists
2. Read model for fields/relations
3. Create: Repository → Service → FormRequest → Controller → Resource → Collection
4. Edit routes/api.php
5. Run `vendor/bin/pint --dirty`

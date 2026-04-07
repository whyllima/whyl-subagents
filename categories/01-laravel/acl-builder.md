---
name: acl-builder
description: Configures standalone modular ACL system (no Spatie) with config-driven permissions matrix (User → Role → Module → Permission), privilege escalation prevention, domain folders, and seeders. UUID support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# ACL Builder

Configures complete standalone ACL system — no external packages. Config-driven matrix with privilege escalation prevention.

## CRITICAL RULES — Read FIRST

1. **No external packages needed** — this is 100% standalone
2. **ALWAYS run `php artisan migrate` at the END**
3. **ALWAYS read existing files before modifying them** — NEVER blindly overwrite
4. **When modifying User.php or bootstrap/app.php**: read FULL file, merge additions, write COMPLETE file
5. **Controller examples are PATTERNS** — adapt to actual entity names
6. **Create ALL migration files** — system will NOT work without them
7. **Run `vendor/bin/pint --dirty` at the end**

## Architecture

```
┌──────────┐       ┌──────────┐       ┌─────────────┐       ┌─────────────┐
│   User   │──has──│   Role   │──has──│   Module    │──has──│ Permission  │
└──────────┘       └──────────┘       └─────────────┘       └─────────────┘
                         │                   │
                         └───────────────────┘
                    role_modules_permissions (pivot)
```

**Permission Format:** `{action}-{module}` (e.g. `show-rule`, `index-user`, `store-flow`)

## ACL Config (`config/acl.php`)

Central config that drives the entire permission matrix:

```php
<?php

return [
    'permissions' => [
        'show',
        'index',
        'store',
        'update',
        'delete',
        'export',
        'print',
    ],

    'modules' => [
        'user',
        'role',
        'module',
        'permission',
        // Add feature modules here...
    ],

    /*
    | Roles: selective module-permission access.
    | - super_admin: bypasses via Gate::before, no explicit assignments
    | - admin: no modules/permissions key = gets ALL modules x ALL permissions
    | - Custom roles: specify modules[] and permissions[] for selective access
    */
    'roles' => [
        'super_admin' => [
            'display_name' => 'Super Admin',
            'description' => 'Full access, bypasses all permission checks via Gate::before',
        ],
        'admin' => [
            'display_name' => 'Admin',
            'description' => 'Full CRUD on all modules',
        ],
        'manager' => [
            'display_name' => 'Manager',
            'description' => 'Manages specific modules with limited actions',
            'modules' => ['user', 'role'],
            'permissions' => ['show', 'index', 'store', 'update', 'delete'],
        ],
        'viewer' => [
            'display_name' => 'Viewer',
            'description' => 'Read-only access to specific modules',
            'modules' => ['user'],
            'permissions' => ['show', 'index'],
        ],
    ],

    'cache' => [
        'ttl' => 3600,
    ],
];
```

## Migrations

### Migration 1: Permissions and Roles

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('permissions', function (Blueprint $table) {
            $table->unsignedBigInteger('id');
            $table->uuid('uuid')->primary();
            $table->string('name');
            $table->string('guard_name')->default('api');
            $table->timestamps();
            $table->softDeletes();
            $table->unique(['name', 'guard_name']);
        });

        DB::statement('ALTER TABLE permissions MODIFY id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE');

        Schema::create('roles', function (Blueprint $table) {
            $table->unsignedBigInteger('id');
            $table->uuid('uuid')->primary();
            $table->string('name');
            $table->string('guard_name')->default('api');
            $table->timestamps();
            $table->softDeletes();
            $table->unique(['name', 'guard_name']);
        });

        DB::statement('ALTER TABLE roles MODIFY id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE');

        Schema::create('model_has_roles', function (Blueprint $table) {
            $table->uuid('role_uuid');
            $table->string('model_type');
            $table->uuid('model_uuid');
            $table->foreign('role_uuid')->references('uuid')->on('roles')->onDelete('cascade');
            $table->primary(['role_uuid', 'model_uuid', 'model_type']);
            $table->index(['model_uuid', 'model_type']);
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('model_has_roles');
        Schema::dropIfExists('roles');
        Schema::dropIfExists('permissions');
    }
};
```

### Migration 2: Modules and Module-Permission Pivots

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('modules', function (Blueprint $table) {
            $table->unsignedBigInteger('id');
            $table->uuid('uuid')->primary();
            $table->string('name');
            $table->string('guard_name')->default('api');
            $table->unique(['name', 'guard_name']);
            $table->timestamps();
            $table->softDeletes();
        });

        DB::statement('ALTER TABLE modules MODIFY id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE');

        Schema::create('model_has_permissions', function (Blueprint $table) {
            $table->unsignedBigInteger('id');
            $table->uuid('uuid')->primary();
            $table->uuid('permission_uuid');
            $table->string('model_type');
            $table->uuid('model_uuid');
            $table->foreign('permission_uuid')->references('uuid')->on('permissions')->onDelete('cascade');
            $table->index(['model_uuid', 'model_type']);
            $table->unique(['permission_uuid', 'model_uuid', 'model_type'], 'mhp_perm_model_unique');
            $table->timestamps();
        });

        DB::statement('ALTER TABLE model_has_permissions MODIFY id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE');

        Schema::create('role_modules_permissions', function (Blueprint $table) {
            $table->unsignedBigInteger('id');
            $table->uuid('uuid')->primary();
            $table->uuid('role_uuid');
            $table->uuid('model_permission_uuid');
            $table->foreign('role_uuid')->references('uuid')->on('roles')->onDelete('cascade');
            $table->foreign('model_permission_uuid')->references('uuid')->on('model_has_permissions')->onDelete('cascade');
            $table->unique(['role_uuid', 'model_permission_uuid'], 'rmp_role_perm_unique');
            $table->timestamps();
            $table->softDeletes();
        });

        DB::statement('ALTER TABLE role_modules_permissions MODIFY id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE');
    }

    public function down(): void
    {
        Schema::dropIfExists('role_modules_permissions');
        Schema::dropIfExists('model_has_permissions');
        Schema::dropIfExists('modules');
    }
};
```

## Models (in `app/Models/` — ACL models stay at root, not domain folders)

### Permission

```php
namespace App\Models;

use App\Traits\HasNameLookup;
use Illuminate\Database\Eloquent\Attributes\Fillable;
use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

#[Table(key: 'uuid', keyType: 'string', incrementing: false)]
#[Fillable(['name', 'guard_name'])]
class Permission extends Model
{
    use HasFactory, HasNameLookup, HasUuids, SoftDeletes;

    protected $hidden = ['guard_name', 'created_at', 'updated_at', 'deleted_at'];

    public function uniqueIds(): array
    {
        return ['uuid'];
    }
}
```

### Role

```php
namespace App\Models;

use App\Traits\HasNameLookup;
use Illuminate\Database\Eloquent\Attributes\Fillable;
use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

#[Table(key: 'uuid', keyType: 'string', incrementing: false)]
#[Fillable(['name', 'guard_name'])]
class Role extends Model
{
    use HasFactory, HasNameLookup, HasUuids, SoftDeletes;

    protected $hidden = ['guard_name', 'created_at', 'updated_at', 'deleted_at'];

    public function uniqueIds(): array
    {
        return ['uuid'];
    }

    public function roleModulesPermission()
    {
        return $this->hasMany(RoleModulesPermission::class, 'role_uuid', 'uuid');
    }

    public function users()
    {
        return $this->morphedByMany(\App\Models\Auth\User::class, 'model', 'model_has_roles', 'role_uuid', 'model_uuid');
    }

    public function giveModelPermissionTo(string $module, string $permission): static
    {
        $moduleModel = Module::findByName($module);
        $permissionModel = Permission::findByName($permission);

        if ($moduleModel && $permissionModel) {
            $modelHasPermission = ModelHasPermission::where([
                'permission_uuid' => $permissionModel->uuid,
                'model_type' => Module::class,
                'model_uuid' => $moduleModel->uuid,
            ])->first();

            if ($modelHasPermission) {
                RoleModulesPermission::withTrashed()->updateOrCreate([
                    'role_uuid' => $this->uuid,
                    'model_permission_uuid' => $modelHasPermission->uuid,
                ])->restore();

                $this->load(['roleModulesPermission.modelHasPermission.module', 'roleModulesPermission.modelHasPermission.permission']);
            }
        }

        return $this;
    }

    public function revokeModelPermissionTo(string $module, string $permission): static
    {
        $moduleModel = Module::findByName($module);
        $permissionModel = Permission::findByName($permission);

        if ($moduleModel && $permissionModel) {
            $modelHasPermission = ModelHasPermission::where([
                'permission_uuid' => $permissionModel->uuid,
                'model_type' => Module::class,
                'model_uuid' => $moduleModel->uuid,
            ])->first();

            if ($modelHasPermission) {
                RoleModulesPermission::where([
                    'role_uuid' => $this->uuid,
                    'model_permission_uuid' => $modelHasPermission->uuid,
                ])->delete();

                $this->load(['roleModulesPermission.modelHasPermission.module', 'roleModulesPermission.modelHasPermission.permission']);
            }
        }

        return $this;
    }
}
```

### Module

```php
namespace App\Models;

use App\Traits\HasGuard;
use App\Traits\HasNameLookup;
use Illuminate\Database\Eloquent\Attributes\Fillable;
use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

#[Table(key: 'uuid', keyType: 'string', incrementing: false)]
#[Fillable(['name', 'guard_name'])]
class Module extends Model
{
    use HasFactory, HasGuard, HasNameLookup, HasUuids, SoftDeletes;

    protected $hidden = ['guard_name', 'created_at', 'updated_at', 'deleted_at'];

    public function uniqueIds(): array
    {
        return ['uuid'];
    }

    public function permissions()
    {
        return $this->morphMany(ModelHasPermission::class, 'model', 'model_type', 'model_uuid');
    }

    public function hasPermissionTo(string $permissionName): bool
    {
        $permission = Permission::where('name', $permissionName)->first();

        if (! $permission) {
            return false;
        }

        return ModelHasPermission::where([
            'permission_uuid' => $permission->uuid,
            'model_type' => static::class,
            'model_uuid' => $this->uuid,
        ])->exists();
    }

    public function givePermissionTo(array|string ...$permissions): static
    {
        $permissions = collect($permissions)->flatten()->all();

        foreach ($permissions as $permissionName) {
            $permission = Permission::where('name', $permissionName)->first();

            if ($permission) {
                ModelHasPermission::firstOrCreate([
                    'permission_uuid' => $permission->uuid,
                    'model_type' => static::class,
                    'model_uuid' => $this->uuid,
                ]);
            }
        }

        return $this;
    }
}
```

### ModelHasPermission

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Fillable;
use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

#[Table(key: 'uuid', keyType: 'string', incrementing: false)]
#[Fillable(['permission_uuid', 'model_type', 'model_uuid'])]
class ModelHasPermission extends Model
{
    use HasFactory, HasUuids;

    public function uniqueIds(): array
    {
        return ['uuid'];
    }

    public function module()
    {
        return $this->hasMany(Module::class, 'uuid', 'model_uuid');
    }

    public function permission()
    {
        return $this->hasMany(Permission::class, 'uuid', 'permission_uuid');
    }
}
```

### RoleModulesPermission

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Fillable;
use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

#[Table(key: 'uuid', keyType: 'string', incrementing: false)]
#[Fillable(['role_uuid', 'model_permission_uuid'])]
class RoleModulesPermission extends Model
{
    use HasFactory, HasUuids, SoftDeletes;

    public function uniqueIds(): array
    {
        return ['uuid'];
    }

    public function role()
    {
        return $this->belongsTo(Role::class, 'role_uuid', 'uuid');
    }

    public function modelHasPermission()
    {
        return $this->belongsTo(ModelHasPermission::class, 'model_permission_uuid', 'uuid');
    }
}
```

## Traits

### HasNameLookup

```php
namespace App\Traits;

use Illuminate\Database\Eloquent\ModelNotFoundException;

trait HasNameLookup
{
    public static function findByName(string $name): static
    {
        $model = static::where('name', $name)->first();

        if (! $model) {
            throw new ModelNotFoundException("No record found with name: {$name}");
        }

        return $model;
    }
}
```

### HasGuard

```php
namespace App\Traits;

trait HasGuard
{
    protected static function bootHasGuard(): void
    {
        static::creating(function ($model) {
            $model->guard_name = $model->guard_name ?? 'api';
        });
    }
}
```

### HasRoles (User trait — replaces Spatie HasRoles)

```php
namespace App\Traits;

use App\Models\Role;

trait HasRoles
{
    public function roles()
    {
        return $this->morphToMany(Role::class, 'model', 'model_has_roles', 'model_uuid', 'role_uuid');
    }

    public function hasRole(string $roleName): bool
    {
        return $this->roles()->where('name', $roleName)->exists();
    }

    public function assignRole(string $roleName): static
    {
        $role = Role::findByName($roleName);
        $this->roles()->syncWithoutDetaching([$role->uuid]);

        return $this;
    }

    public function removeRole(string $roleName): static
    {
        $role = Role::findByName($roleName);
        $this->roles()->detach($role->uuid);

        return $this;
    }

    public function isSuperAdmin(): bool
    {
        return $this->hasRole('super_admin');
    }
}
```

### HasModulePermission (User trait — module-level checks)

```php
namespace App\Traits;

use App\Models\Module;
use App\Models\Permission;
use Exception;

trait HasModulePermission
{
    public function hasModuleAndPermissionTo(array $modelsHasPermissions): bool
    {
        foreach ($modelsHasPermissions as $modelPermission) {
            $parts = explode('-', $modelPermission, 2);

            if (count($parts) < 2) {
                throw new Exception('Invalid permission format. Expected: {action}-{module}');
            }

            [$action, $moduleName] = $parts;

            try {
                $module = Module::findByName($moduleName);
                $permission = Permission::findByName($action);
            } catch (Exception) {
                continue;
            }

            if ($this->userHasPermissionForModule($module, $permission)) {
                return true;
            }
        }

        return false;
    }

    protected function userHasPermissionForModule($module, $permission): bool
    {
        foreach ($this->roles as $role) {
            foreach ($role->roleModulesPermission as $rmp) {
                $mhp = $rmp->modelHasPermission;

                foreach ($mhp->module as $mod) {
                    if ($mod->uuid !== $module->uuid) {
                        continue;
                    }

                    foreach ($mhp->permission as $perm) {
                        if ($perm->uuid === $permission->uuid) {
                            return true;
                        }
                    }
                }
            }
        }

        return false;
    }
}
```

## Exception

```php
namespace App\Exceptions;

use Exception;
use Illuminate\Http\Response;

class NotLoggedInException extends Exception
{
    public static function notLoggedIn(): static
    {
        return new static('User is not logged in.', Response::HTTP_FORBIDDEN);
    }
}
```

```php
namespace App\Exceptions;

use Exception;

class UnauthorizedException extends Exception
{
    public static function forPermission(string $permission): static
    {
        return new static("User does not have the required permission: {$permission}", 403);
    }
}
```

## Middleware

```php
namespace App\Http\Middleware;

use App\Exceptions\NotLoggedInException;
use App\Exceptions\UnauthorizedException;
use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Gate;
use Symfony\Component\HttpFoundation\Response;

class ModelAndPermissionMiddleware
{
    public function handle(Request $request, Closure $next, string $permission, ?string $guard = null): Response
    {
        $user = Auth::guard($guard)->user();

        if (! $user) {
            throw NotLoggedInException::notLoggedIn();
        }

        // Super admin bypass via Gate::before
        if (Gate::forUser($user)->allows('before', $permission)) {
            return $next($request);
        }

        $permissions = is_array($permission) ? $permission : explode('|', $permission);

        if (! method_exists($user, 'hasModuleAndPermissionTo')) {
            throw UnauthorizedException::forPermission($permission);
        }

        if (! $user->hasModuleAndPermissionTo($permissions)) {
            throw UnauthorizedException::forPermission($permission);
        }

        return $next($request);
    }
}
```

## Middleware Registration (`bootstrap/app.php`) — MERGE

Read existing file first. Add inside `->withMiddleware()`:

```php
$middleware->alias([
    'permission' => \App\Http\Middleware\ModelAndPermissionMiddleware::class,
]);
```

## User Model — MERGE (do NOT replace)

Read existing User.php. ADD these traits:

```php
use App\Traits\HasRoles;
use App\Traits\HasModulePermission;

class User extends Authenticatable
{
    use HasRoles, HasModulePermission;
    // ... keep all existing code
}
```

## Privilege Escalation Prevention

**CRITICAL SECURITY — Must be in RoleRepository:**

### RoleRepository (`app/Repositories/AccessControl/RoleRepository.php`)

```php
namespace App\Repositories\AccessControl;

use App\Models\Module;
use App\Models\Permission;
use App\Models\Role;
use App\Repositories\Repository;
use Exception;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Log;

class RoleRepository extends Repository
{
    public function __construct()
    {
        $this->model = new Role();
    }

    /**
     * List roles — EXCLUDES the user's own roles.
     */
    public function index(array $filters = [])
    {
        try {
            $user = Auth::user();
            $perPage = $filters['per_page'] ?? 15;

            if ($user->isSuperAdmin()) {
                return Role::orderBy('created_at', 'desc')->paginate($perPage);
            }

            $userRoleIds = $user->roles->pluck('uuid')->toArray();

            return Role::whereNotIn('uuid', $userRoleIds)
                ->orderBy('created_at', 'desc')
                ->paginate($perPage);
        } catch (Exception $e) {
            Log::channel('repositories')->error("RoleRepository:index - {$e->getMessage()}");
            throw $e;
        }
    }

    /**
     * Bulk assign — validates EACH permission against user's own.
     */
    public function bulkAssignModulePermissions(Role $role, array $permissions)
    {
        try {
            $user = Auth::user();

            foreach ($permissions as $permissionData) {
                $module = Module::findByName($permissionData['module']);
                $permission = Permission::findByName($permissionData['permission']);

                if ($this->userCanAssignModulePermission($user, $module, $permission)) {
                    $role->giveModelPermissionTo($module->name, $permission->name);
                }
            }

            return $role;
        } catch (Exception $e) {
            Log::channel('repositories')->error("RoleRepository:bulkAssign - {$e->getMessage()}");
            throw $e;
        }
    }

    /**
     * Bulk unassign — validates EACH permission against user's own.
     */
    public function bulkUnassignModulePermissions(Role $role, array $permissions)
    {
        try {
            $user = Auth::user();

            foreach ($permissions as $permissionData) {
                $module = Module::findByName($permissionData['module']);
                $permission = Permission::findByName($permissionData['permission']);

                if ($this->userCanAssignModulePermission($user, $module, $permission)) {
                    $role->revokeModelPermissionTo($module->name, $permission->name);
                }
            }

            return $role;
        } catch (Exception $e) {
            Log::channel('repositories')->error("RoleRepository:bulkUnassign - {$e->getMessage()}");
            throw $e;
        }
    }

    private function userCanAssignModulePermission($user, $module, $permission): bool
    {
        if (! $user) {
            return false;
        }

        if ($user->isSuperAdmin()) {
            return true;
        }

        return $this->userHasModulePermission($user, $module, $permission);
    }

    private function userHasModulePermission($user, $module, $permission): bool
    {
        try {
            foreach ($user->roles as $role) {
                foreach ($role->roleModulesPermission as $rmp) {
                    $mhp = $rmp->modelHasPermission;

                    foreach ($mhp->module as $mod) {
                        if ($mod->uuid === $module->uuid) {
                            foreach ($mhp->permission as $perm) {
                                if ($perm->uuid === $permission->uuid) {
                                    return true;
                                }
                            }
                        }
                    }
                }
            }

            return false;
        } catch (Exception $e) {
            Log::channel('repositories')->error("RoleRepository:userHasModulePermission - {$e->getMessage()}");
            return false;
        }
    }
}
```

### RoleController (`app/Http/Controllers/AccessControl/RoleController.php`)

```php
namespace App\Http\Controllers\AccessControl;

use App\Http\Controllers\Controller;
use App\Http\Requests\AccessControl\RoleBulkPermissionRequest;
use App\Http\Requests\AccessControl\RoleRequest;
use App\Models\Role;
use App\Services\AccessControl\RoleService;
use Illuminate\Http\Resources\Json\JsonResource;

class RoleController extends Controller
{
    public function __construct(private RoleService $service) {}

    public function index(): JsonResource { return $this->service->index(); }
    public function show(Role $role): JsonResource { return $this->service->show($role); }
    public function store(RoleRequest $request): JsonResource { return $this->service->store($request->validated()); }
    public function update(Role $role, RoleRequest $request): JsonResource { return $this->service->update($role, $request->validated()); }
    public function destroy(Role $role): JsonResource { return $this->service->destroy($role); }

    public function bulkAssignModulePermissions(Role $role, RoleBulkPermissionRequest $request): JsonResource
    {
        return $this->service->bulkAssignModulePermissions($role, $request->validated()['permissions']);
    }

    public function bulkUnassignModulePermissions(Role $role, RoleBulkPermissionRequest $request): JsonResource
    {
        return $this->service->bulkUnassignModulePermissions($role, $request->validated()['permissions']);
    }
}
```

### RoleBulkPermissionRequest

```php
namespace App\Http\Requests\AccessControl;

use Illuminate\Foundation\Http\FormRequest;

class RoleBulkPermissionRequest extends FormRequest
{
    public function authorize(): bool { return true; }

    public function rules(): array
    {
        return [
            'permissions' => 'required|array|min:1',
            'permissions.*.module' => 'required|string|exists:modules,name',
            'permissions.*.permission' => 'required|string|exists:permissions,name',
        ];
    }
}
```

## Security Rules Summary

| Rule | Implementation |
|------|---------------|
| Cannot give permissions you don't have | `userCanAssignModulePermission()` in RoleRepository |
| Cannot remove permissions you don't have | Same validation on bulkUnassign |
| Cannot see/assign own role | `whereNotIn('uuid', $userRoleIds)` in index() |
| Separate assign/unassign permissions | `assign-role` and `unassign-role` as distinct actions |
| Super admin bypasses all | `Gate::before` + `$user->isSuperAdmin()` |
| Bulk operations validate per-item | Each permission in array checked individually |

## Seeder (Config-Driven)

```php
namespace Database\Seeders;

use App\Models\Module;
use App\Models\Permission;
use App\Models\Role;
use Illuminate\Database\Seeder;

class PermissionsSeeder extends Seeder
{
    public function run(): void
    {
        $this->createPermissions();
        $this->createModulesWithPermissions();
        $this->createRoles();
    }

    private function createPermissions(): void
    {
        foreach (config('acl.permissions') as $permissionName) {
            Permission::firstOrCreate(['name' => $permissionName, 'guard_name' => 'api']);
        }
    }

    private function createModulesWithPermissions(): void
    {
        $permissions = config('acl.permissions');

        foreach (config('acl.modules') as $moduleName) {
            $module = Module::firstOrCreate(['name' => $moduleName, 'guard_name' => 'api']);
            $module->givePermissionTo($permissions);
        }
    }

    private function createRoles(): void
    {
        $allModules = config('acl.modules');
        $allPermissions = config('acl.permissions');

        foreach (config('acl.roles') as $roleName => $roleConfig) {
            $role = Role::firstOrCreate(['name' => $roleName, 'guard_name' => 'api']);

            // super_admin bypasses via Gate::before — no explicit assignments
            if ($roleName === 'super_admin') {
                continue;
            }

            // Roles without explicit modules/permissions get ALL
            $modules = $roleConfig['modules'] ?? $allModules;
            $permissions = $roleConfig['permissions'] ?? $allPermissions;

            foreach ($modules as $moduleName) {
                foreach ($permissions as $permissionName) {
                    $role->giveModelPermissionTo($moduleName, $permissionName);
                }
            }
        }
    }
}
```

## Gate for Super Admin (AppServiceProvider)

```php
use Illuminate\Support\Facades\Gate;

Gate::before(function ($user, $ability) {
    if ($user->isSuperAdmin()) {
        return true;
    }
});
```

## Routes (`routes/api/v1.php`)

```php
use App\Http\Controllers\AccessControl\RoleController;

Route::controller(RoleController::class)->prefix('roles')->group(function () {
    Route::get('/', 'index')->middleware('permission:index-role');
    Route::get('/{role}', 'show')->middleware('permission:show-role');
    Route::post('/', 'store')->middleware('permission:store-role');
    Route::put('/{role}', 'update')->middleware('permission:update-role');
    Route::delete('/{role}', 'destroy')->middleware('permission:delete-role');
    Route::post('/{role}/assign-permissions', 'bulkAssignModulePermissions')->middleware('permission:assign-role');
    Route::post('/{role}/unassign-permissions', 'bulkUnassignModulePermissions')->middleware('permission:unassign-role');
});
```

## Workflow — Execute in this EXACT order

1. Create `config/acl.php` (ask user for modules, permissions, and roles)
2. Create migrations (2 files)
3. Create Models: Permission, Role, Module, ModelHasPermission, RoleModulesPermission
4. Create Traits: HasNameLookup, HasGuard, HasRoles, HasModulePermission
5. Create Exceptions: NotLoggedInException, UnauthorizedException
6. Create Middleware: ModelAndPermissionMiddleware
7. Read + merge `bootstrap/app.php` (add middleware alias `permission`)
8. Read + merge `User.php` (add HasRoles, HasModulePermission traits)
9. Create RoleRepository (with privilege escalation prevention)
10. Create RoleService, RoleController, RoleBulkPermissionRequest
11. Create PermissionsSeeder (config-driven)
12. Read + merge `AppServiceProvider.php` (add Gate::before)
13. Add role routes to `routes/api/v1.php`
14. `php artisan migrate`
15. `php artisan db:seed --class=PermissionsSeeder`
16. `vendor/bin/pint --dirty`

## Detection Check

```bash
ls app/Models/Module.php 2>/dev/null
ls app/Traits/HasModulePermission.php 2>/dev/null
ls app/Traits/HasRoles.php 2>/dev/null
```

If exists → update/extend. If not → create full structure.

---
name: acl-builder
description: Configures modular ACL system with config-driven permissions matrix (User → Role → Module → Permission), privilege escalation prevention, domain folders, and seeders. UUID support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# ACL Builder

Configures complete ACL system with config-driven matrix and privilege escalation prevention.

## CRITICAL RULES — Read FIRST

1. **ALWAYS run `composer require` FIRST** — before creating any files
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

## Install

```bash
composer require spatie/laravel-permission
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
```

## Config (`config/permission.php`)

```php
<?php

return [
    'models' => [
        'permission' => App\Models\Permission::class,
        'role' => App\Models\Role::class,
    ],
    'table_names' => [
        'roles' => 'roles',
        'permissions' => 'permissions',
        'model_has_permissions' => 'model_has_permissions',
        'model_has_roles' => 'model_has_roles',
        'role_has_permissions' => 'role_has_permissions',
    ],
    'column_names' => [
        'role_pivot_key' => null,
        'permission_pivot_key' => null,
        'model_morph_key' => 'model_uuid',
        'team_foreign_key' => 'team_id',
    ],
    'register_permission_check_method' => true,
    'register_octane_reset_listener' => false,
    'teams' => false,
    'use_passport_client_credentials' => false,
    'display_permission_in_exception' => false,
    'display_role_in_exception' => false,
    'enable_wildcard_permission' => false,
    'cache' => [
        'expiration_time' => \DateInterval::createFromDateString('24 hours'),
        'key' => 'spatie.permission.cache',
        'store' => 'default',
    ],
];
```

## Migrations

### Migration 1: Permission Tables (UUID)

Same as before — creates permissions, roles, model_has_permissions, model_has_roles, role_has_permissions tables with UUID primary keys.

### Migration 2: Modules Table

Same as before — creates modules and role_modules_permissions tables.

## Models (in `app/Models/`)

Permission, Role, Module, ModelHasPermission, RoleModulesPermission — same patterns as before with UUID support. ACL models stay in root `app/Models/` (not domain folders).

## Traits

HasNameLookup, HasGuard, HasModulePermission — same patterns as before.

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
     * List roles — EXCLUDES the user's own roles (prevents self-role assignment).
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
     * Bulk assign — validates EACH permission against user's own permissions.
     */
    public function bulkAssignModulePermissions(Role $role, array $permissions)
    {
        try {
            $user = Auth::user();

            foreach ($permissions as $permissionData) {
                $module = Module::findByName($permissionData['module']);
                $permission = Permission::findByName($permissionData['permission']);

                if ($module->exists() && $permission->exists()) {
                    if ($this->userCanAssignModulePermission($user, $module, $permission)) {
                        $role->giveModelPermissionTo($module->name, $permission->name);
                    }
                }
            }

            return $role;
        } catch (Exception $e) {
            Log::channel('repositories')->error("RoleRepository:bulkAssign - {$e->getMessage()}");
            throw $e;
        }
    }

    /**
     * Bulk unassign — validates EACH permission against user's own permissions.
     */
    public function bulkUnassignModulePermissions(Role $role, array $permissions)
    {
        try {
            $user = Auth::user();

            foreach ($permissions as $permissionData) {
                $module = Module::findByName($permissionData['module']);
                $permission = Permission::findByName($permissionData['permission']);

                if ($module->exists() && $permission->exists()) {
                    if ($this->userCanAssignModulePermission($user, $module, $permission)) {
                        $role->revokeModelPermissionTo($module->name, $permission->name);
                    }
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
                foreach ($role->roleModulesPermission as $roleModulePermission) {
                    $modelHasPermission = $roleModulePermission->modelHasPermission;

                    foreach ($modelHasPermission->module as $mod) {
                        if ($mod->uuid === $module->uuid) {
                            foreach ($modelHasPermission->permission as $perm) {
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

            if ($roleName === 'super_admin') {
                continue;
            }

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

## Gate for Super Admin (AppServiceProvider)

```php
Gate::before(function ($user, $ability) {
    if ($user->hasRole('super_admin')) {
        return true;
    }
});
```

## Workflow — Execute in this EXACT order

1. `composer require spatie/laravel-permission`
2. Create `config/permission.php` and `config/acl.php` (ask user for modules and roles)
3. Create migration files
4. Create Models (Permission, Role, Module, ModelHasPermission, RoleModulesPermission)
5. Create Traits (HasNameLookup, HasGuard, HasModulePermission)
6. Create Exception (NotLoggedInException)
7. Create Middleware (ModelAndPermissionMiddleware)
8. Read + merge `bootstrap/app.php` (add middleware aliases)
9. Read + merge `User.php` (add traits + isSuperAdmin)
10. Create RoleRepository (with privilege escalation prevention)
11. Create RoleService, RoleController, RoleBulkPermissionRequest
12. Create PermissionsSeeder (config-driven)
13. Read + merge `AppServiceProvider.php` (add Gate::before)
14. Add role routes to `routes/api/v1.php`
15. `php artisan migrate`
16. `php artisan db:seed --class=PermissionsSeeder`
17. `vendor/bin/pint --dirty`

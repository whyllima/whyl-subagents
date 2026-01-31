---
name: acl-builder
description: Configures modular ACL system with Module-based permissions (User → Role → Module → Permission) including middleware, traits, models, migrations, config, and seeders. UUID support.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# ACL Builder

Configures complete ACL system with three-level architecture: `User → Role → Module → Permission`.

## Architecture

```
┌──────────┐       ┌──────────┐       ┌─────────────┐       ┌─────────────┐
│   User   │──has──│   Role   │──has──│   Module    │──has──│ Permission  │
└──────────┘       └──────────┘       └─────────────┘       └─────────────┘
                         │                   │
                         └───────────────────┘
                    role_modules_permissions (pivot)
```

## Permission Format

```
{action}-{module}
Examples: show-rule, index-user, store-entity, delete-campaign
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

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        $tableNames = config('permission.table_names');
        $columnNames = config('permission.column_names');
        $pivotRole = $columnNames['role_pivot_key'] ?? 'role_id';
        $pivotPermission = $columnNames['permission_pivot_key'] ?? 'permission_id';

        Schema::create($tableNames['permissions'], function (Blueprint $table) {
            $table->uuid('uuid')->primary()->unique();
            $table->unsignedBigInteger('id');
            $table->string('name');
            $table->string('guard_name');
            $table->timestamps();
            $table->softDeletes();
            $table->unique(['name', 'guard_name']);
        });

        Schema::create($tableNames['roles'], function (Blueprint $table) {
            $table->uuid('uuid')->primary()->unique();
            $table->unsignedBigInteger('id');
            $table->string('name');
            $table->string('guard_name');
            $table->uuid('companie_id')->nullable();
            $table->timestamps();
            $table->softDeletes();
            $table->unique(['name', 'guard_name']);
        });

        Schema::create($tableNames['model_has_permissions'], function (Blueprint $table) use ($tableNames, $columnNames, $pivotPermission) {
            $table->uuid('uuid')->unique();
            $table->uuid($pivotPermission);
            $table->string('model_type');
            $table->uuid($columnNames['model_morph_key']);
            $table->index([$columnNames['model_morph_key'], 'model_type'], 'model_has_permissions_model_id_model_type_index');
            $table->foreign($pivotPermission)->references('uuid')->on($tableNames['permissions'])->onDelete('cascade');
            $table->primary([$pivotPermission, $columnNames['model_morph_key'], 'model_type'], 'model_has_permissions_permission_model_type_primary');
            $table->timestamps();
        });

        Schema::create($tableNames['model_has_roles'], function (Blueprint $table) use ($tableNames, $columnNames, $pivotRole) {
            $table->uuid($pivotRole);
            $table->string('model_type');
            $table->uuid($columnNames['model_morph_key']);
            $table->index([$columnNames['model_morph_key'], 'model_type'], 'model_has_roles_model_id_model_type_index');
            $table->foreign($pivotRole)->references('uuid')->on($tableNames['roles'])->onDelete('cascade');
            $table->primary([$pivotRole, $columnNames['model_morph_key'], 'model_type'], 'model_has_roles_role_model_type_primary');
            $table->timestamps();
        });

        Schema::create($tableNames['role_has_permissions'], function (Blueprint $table) use ($tableNames, $pivotRole, $pivotPermission) {
            $table->uuid($pivotPermission);
            $table->uuid($pivotRole);
            $table->foreign($pivotPermission)->references('uuid')->on($tableNames['permissions'])->onDelete('cascade');
            $table->foreign($pivotRole)->references('uuid')->on($tableNames['roles'])->onDelete('cascade');
            $table->primary([$pivotPermission, $pivotRole], 'role_has_permissions_permission_id_role_id_primary');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        $tableNames = config('permission.table_names');
        Schema::drop($tableNames['role_has_permissions']);
        Schema::drop($tableNames['model_has_roles']);
        Schema::drop($tableNames['model_has_permissions']);
        Schema::drop($tableNames['roles']);
        Schema::drop($tableNames['permissions']);
    }
};
```

### Migration 2: Modules Table

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('modules', function (Blueprint $table) {
            $table->uuid('uuid')->primary()->unique();
            $table->unsignedBigInteger('id');
            $table->string('name');
            $table->string('guard_name');
            $table->unique(['name', 'guard_name']);
            $table->timestamps();
            $table->softDeletes();
        });

        Schema::create('role_modules_permissions', function (Blueprint $table) {
            $table->uuid('uuid');
            $table->unsignedBigInteger('id');
            $table->uuid('role_id');
            $table->uuid('model_permission_id');
            $table->foreign('role_id')->references('uuid')->on('roles')->onDelete('cascade');
            $table->foreign('model_permission_id')->references('uuid')->on('model_has_permissions')->onDelete('cascade');
            $table->primary(['uuid', 'role_id', 'model_permission_id']);
            $table->timestamps();
            $table->softDeletes();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('role_modules_permissions');
        Schema::dropIfExists('modules');
    }
};
```

## Models

### Permission

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;
use Spatie\Permission\Models\Permission as SpatiePermission;

class Permission extends SpatiePermission
{
    use HasFactory, HasUuids, SoftDeletes;

    protected $primaryKey = 'uuid';
    protected $hidden = ['guard_name', 'created_at', 'updated_at', 'deleted_at'];
}
```

### Role

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;
use Spatie\Permission\Models\Role as SpatieRole;

class Role extends SpatieRole
{
    use HasUuids, HasFactory, SoftDeletes;

    protected $primaryKey = 'uuid';
    protected $keyType = 'string';
    protected $hidden = ['guard_name', 'created_at', 'updated_at', 'deleted_at'];

    public function roleModulesPermission()
    {
        return $this->hasMany(RoleModulesPermission::class, 'role_id', 'uuid');
    }

    public function giveModelPermissionTo(string $module, string $permission)
    {
        $moduleModel = Module::findByName($module);
        $permissionModel = Permission::findByName($permission);

        if ($moduleModel && $permissionModel) {
            if ($moduleModel->hasPermissionTo($permissionModel->name)) {
                $modelHasPermission = ModelHasPermission::where([
                    'permission_id' => $permissionModel->uuid,
                    'model_type' => get_class($moduleModel),
                    'model_uuid' => $moduleModel->uuid,
                ])->first();

                if ($modelHasPermission) {
                    RoleModulesPermission::withTrashed()->updateOrCreate([
                        'role_id' => $this->uuid,
                        'model_permission_id' => $modelHasPermission->uuid,
                    ])->restore();
                    $this->load(['roleModulesPermission.modelHasPermission.module', 'roleModulesPermission.modelHasPermission.permission']);
                }
            }
        }
        return $this;
    }

    public function revokeModelPermissionTo($module, $permission)
    {
        $moduleModel = Module::findByName($module);
        $permissionModel = Permission::findByName($permission);

        if ($moduleModel && $permissionModel) {
            if ($moduleModel->hasPermissionTo($permissionModel->name)) {
                $modelHasPermission = ModelHasPermission::where([
                    'permission_id' => $permissionModel->uuid,
                    'model_type' => get_class($moduleModel),
                    'model_uuid' => $moduleModel->uuid,
                ])->first();

                if ($modelHasPermission) {
                    RoleModulesPermission::where([
                        'role_id' => $this->uuid,
                        'model_permission_id' => $modelHasPermission->uuid,
                    ])->delete();
                    $this->load(['roleModulesPermission.modelHasPermission.module', 'roleModulesPermission.modelHasPermission.permission']);
                }
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
use Spatie\Permission\Traits\HasRoles;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Module extends Model
{
    use HasFactory, HasUuids, HasRoles, HasNameLookup, SoftDeletes, HasGuard;

    protected $primaryKey = 'uuid';
    protected $keyType = 'string';
    protected $fillable = ['name', 'guard_name'];
    protected $hidden = ['guard_name', 'created_at', 'updated_at', 'deleted_at'];

    public function givePermissionTo(...$permissions)
    {
        foreach ($permissions as $permission) {
            if (!is_string($permission)) {
                foreach ($permission as $p) {
                    $this->assignPermission($p);
                }
            } else {
                $this->assignPermission($permission);
            }
        }
        return $this;
    }

    private function assignPermission(string $permissionName): void
    {
        $findPermission = Permission::where('name', $permissionName)->first();
        if ($findPermission) {
            ModelHasPermission::updateOrCreate([
                'permission_id' => $findPermission->uuid,
                'model_type' => get_class($this),
                'model_uuid' => $this->uuid
            ]);
        }
    }
}
```

### ModelHasPermission

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class ModelHasPermission extends Model
{
    use HasFactory, HasUuids;

    protected $primaryKey = 'uuid';
    protected $keyType = 'string';
    protected $fillable = ['permission_id', 'model_type', 'model_uuid'];
    protected $hidden = ['created_at', 'updated_at'];

    public function module()
    {
        return $this->hasMany(Module::class, 'uuid', 'model_uuid');
    }

    public function permission()
    {
        return $this->hasMany(Permission::class, 'uuid', 'permission_id');
    }
}
```

### RoleModulesPermission

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class RoleModulesPermission extends Model
{
    use HasFactory, SoftDeletes, HasUuids;

    protected $primaryKey = 'uuid';
    protected $keyType = 'string';
    protected $fillable = ['role_id', 'model_permission_id'];
    protected $hidden = ['created_at', 'updated_at', 'deleted_at'];

    public function role()
    {
        return $this->belongsTo(Role::class, 'role_id', 'uuid');
    }

    public function modelHasPermission()
    {
        return $this->belongsTo(ModelHasPermission::class, 'model_permission_id', 'uuid');
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
    public static function findByName(string $name)
    {
        $model = static::where('name', $name)->first();
        if (!$model) {
            throw new ModelNotFoundException($name);
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
    protected static function bootHasGuard()
    {
        static::creating(function ($model) {
            $guard = auth()->getDefaultDriver();
            $model->guard_name = $guard;
        });
    }
}
```

### HasModulePermission (User trait)

```php
namespace App\Traits;

use Exception;
use App\Models\Module;
use App\Models\Permission;

trait HasModulePermission
{
    public function hasAnyModule(...$models): bool
    {
        return collect($models)->flatten()->contains(fn($model) => Module::findByName($model)->exists());
    }

    public function hasModuleAndPermissionTo(array $modelsHasPermissions): bool
    {
        foreach ($modelsHasPermissions as $modelPermission) {
            $modelPermission = explode('-', $modelPermission);
            if (count($modelPermission) < 2) {
                throw new Exception('Invalid model and permission format');
            }
            $model = Module::findByName($modelPermission[1]);
            if (!$model->exists) {
                continue;
            }
            $permission = Permission::findByName($modelPermission[0]);
            if ($permission->exists() && $this->userHasPermissionForModule($model, $permission)) {
                return true;
            }
        }
        return false;
    }

    protected function userHasPermissionForModule($modelInstance, $permission): bool
    {
        foreach ($this->roles as $role) {
            if ($this->roleHasPermissionForModule($role, $modelInstance, $permission)) {
                return true;
            }
        }
        return false;
    }

    private function roleHasPermissionForModule($role, $modelInstance, $permission): bool
    {
        foreach ($role->roleModulesPermission as $modulesPermission) {
            if ($this->moduleHasPermission($modulesPermission->modelHasPermission, $modelInstance, $permission)) {
                return true;
            }
        }
        return false;
    }

    private function moduleHasPermission($modelHasPermission, $modelInstance, $permission): bool
    {
        foreach ($modelHasPermission->module as $module) {
            if ($module->name === $modelInstance->name && $module->hasPermissionTo($permission->name)) {
                return $this->permissionMatches($modelHasPermission->permission, $permission);
            }
        }
        return false;
    }

    private function permissionMatches($modulePermissions, $permission): bool
    {
        foreach ($modulePermissions as $modulePermission) {
            if ($modulePermission->name === $permission->name) {
                return true;
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
    public static function notLoggedIn()
    {
        return new static('User is not logged in.', Response::HTTP_FORBIDDEN);
    }
}
```

## Middleware

```php
namespace App\Http\Middleware;

use Closure;
use Spatie\Permission\Guard;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Gate;
use App\Exceptions\NotLoggedInException;
use Symfony\Component\HttpFoundation\Response;
use Spatie\Permission\Exceptions\UnauthorizedException;

class ModelAndPermissionMiddleware
{
    public function handle(Request $request, Closure $next, $permission, $guard = null): Response
    {
        $user = $this->getUser($request, $guard);
        if (!$user) {
            throw NotLoggedInException::notLoggedIn();
        }

        if ($this->canSkipPermissionCheck($user, $permission)) {
            return $next($request);
        }

        if (!$this->userHasRequiredPermissions($user, $permission)) {
            throw UnauthorizedException::missingTraitHasRoles($user);
        }
        return $next($request);
    }

    protected function getUser(Request $request, $guard)
    {
        $user = Auth::guard($guard)->user();
        if (!$user && $request->bearerToken() && config('permission.use_passport_client_credentials')) {
            $user = Guard::getPassportClient($guard);
        }
        return $user;
    }

    protected function canSkipPermissionCheck($user, $permission): bool
    {
        return Gate::forUser($user)->allows('before', $permission);
    }

    protected function userHasRequiredPermissions($user, $permissions): bool
    {
        $modelsPermissions = is_array($permissions) ? $permissions : explode('|', $permissions);
        if (!method_exists($user, 'hasAnyModule')) {
            return false;
        }
        return $user->hasModuleAndPermissionTo($modelsPermissions);
    }
}
```

## Register Middleware (`bootstrap/app.php`)

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'model_permission' => \App\Http\Middleware\ModelAndPermissionMiddleware::class,
    ]);
})
```

## User Model

```php
namespace App\Models;

use App\Traits\HasModulePermission;
use Spatie\Permission\Traits\HasRoles;
use Tymon\JWTAuth\Contracts\JWTSubject;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements JWTSubject
{
    use HasFactory, HasRoles, SoftDeletes, HasUuids, HasModulePermission;

    protected $primaryKey = 'uuid';
    public $incrementing = false;
    protected $keyType = 'string';
    // ...
}
```

## Controller Usage

```php
class {Entity}Controller extends Controller
{
    public function __construct()
    {
        $this->middleware('model_permission:show-{entity}')->only(['show']);
        $this->middleware('model_permission:index-{entity}')->only(['index']);
        $this->middleware('model_permission:store-{entity}')->only(['store']);
        $this->middleware('model_permission:update-{entity}')->only(['update']);
        $this->middleware('model_permission:delete-{entity}')->only(['destroy']);
    }
}
```

## Seeder

```php
namespace Database\Seeders;

use App\Models\Role;
use App\Models\Module;
use App\Models\Permission;
use Illuminate\Database\Seeder;

class PermissionsSeeder extends Seeder
{
    public function run(): void
    {
        $permissions = ['show', 'index', 'store', 'update', 'delete', 'export', 'print'];
        foreach ($permissions as $permissionName) {
            Permission::firstOrCreate(['name' => $permissionName, 'guard_name' => 'api']);
        }

        $modules = ['user', 'role', 'entity', 'rule'];
        foreach ($modules as $moduleName) {
            $module = Module::firstOrCreate(['name' => $moduleName, 'guard_name' => 'api']);
            $module->givePermissionTo($permissions);
        }

        $adminRole = Role::firstOrCreate(['name' => 'Admin', 'guard_name' => 'api']);
        foreach ($modules as $moduleName) {
            foreach ($permissions as $permissionName) {
                $adminRole->giveModelPermissionTo($moduleName, $permissionName);
            }
        }
    }
}
```

## Gate for Super Admin (AppServiceProvider)

```php
Gate::before(function ($user, $ability) {
    if ($user->hasRole('Super Admin')) {
        return true;
    }
});
```

## Workflow

1. Install spatie/laravel-permission
2. Edit `config/permission.php` (model_morph_key = 'model_uuid')
3. Create migrations (permission_tables, modules)
4. Create Models (Permission, Role, Module, ModelHasPermission, RoleModulesPermission)
5. Create Traits (HasNameLookup, HasGuard, HasModulePermission)
6. Create Exception (NotLoggedInException)
7. Create Middleware (ModelAndPermissionMiddleware)
8. Register middleware in bootstrap/app.php
9. Add traits to User model (HasRoles, HasModulePermission)
10. Create Seeder (PermissionsSeeder)
11. Run migrations and seeders
12. Add middleware to Controllers
13. Run `vendor/bin/pint --dirty`

## Commands

```bash
php artisan migrate
php artisan db:seed --class=PermissionsSeeder
php artisan permission:cache-reset
```

## Detection Check

```bash
ls app/Models/Module.php 2>/dev/null
ls app/Traits/HasModulePermission.php 2>/dev/null
grep -l "model_permission" bootstrap/app.php
```

If exists → update/extend. If not → create full structure.

---
name: audit-builder
description: Configures Laravel Auditing (owen-it/laravel-auditing) with UUID support, custom resolvers, and resources. Also verifies model audit configuration.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Audit Builder

Configures complete auditing system using `owen-it/laravel-auditing` with UUID support.

## Architecture

```
Model (Auditable trait) → Audit Record → audits table
                              ↓
              Resolvers (IP, UserAgent, URL)
```

## Install

```bash
composer require owen-it/laravel-auditing
php artisan vendor:publish --provider="OwenIt\Auditing\AuditingServiceProvider" --tag="config"
php artisan vendor:publish --provider="OwenIt\Auditing\AuditingServiceProvider" --tag="migrations"
```

## Config (`config/audit.php`)

```php
<?php

return [
    'enabled' => env('AUDITING_ENABLED', true),
    'implementation' => OwenIt\Auditing\Models\Audit::class,
    'user' => [
        'morph_prefix' => 'user',
        'guards' => ['web', 'api'],
        'resolver' => OwenIt\Auditing\Resolvers\UserResolver::class
    ],
    'resolvers' => [
        'ip_address' => App\Resolvers\IpAddressResolver::class,
        'user_agent' => App\Resolvers\UserAgentResolver::class,
        'url' => App\Resolvers\UrlResolver::class,
    ],
    'events' => ['created', 'updated', 'deleted', 'restored'],
    'strict' => false,
    'exclude' => [],
    'empty_values' => true,
    'allowed_empty_values' => ['retrieved'],
    'allowed_array_values' => false,
    'timestamps' => false,
    'threshold' => 0,
    'driver' => 'database',
    'drivers' => [
        'database' => [
            'table' => 'audits',
            'connection' => null,
        ],
    ],
    'queue' => [
        'enable' => false,
        'connection' => 'sync',
        'queue' => 'default',
        'delay' => 0,
    ],
    'console' => false,
];
```

## Migration (UUID)

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateAuditsTable extends Migration
{
    public function up()
    {
        $connection = config('audit.drivers.database.connection', config('database.default'));
        $table = config('audit.drivers.database.table', 'audits');

        Schema::connection($connection)->create($table, function (Blueprint $table) {
            $morphPrefix = config('audit.user.morph_prefix', 'user');

            $table->id();
            $table->string($morphPrefix . '_type')->nullable();
            $table->uuid($morphPrefix . '_id')->nullable();
            $table->string('event');
            $table->uuidMorphs('auditable');
            $table->text('old_values')->nullable();
            $table->text('new_values')->nullable();
            $table->text('url')->nullable();
            $table->ipAddress('ip_address')->nullable();
            $table->string('user_agent', 1023)->nullable();
            $table->string('tags')->nullable();
            $table->timestamps();

            $table->index([$morphPrefix . '_id', $morphPrefix . '_type']);
        });
    }

    public function down()
    {
        $connection = config('audit.drivers.database.connection', config('database.default'));
        $table = config('audit.drivers.database.table', 'audits');
        Schema::connection($connection)->drop($table);
    }
}
```

## Resolvers

### IpAddressResolver

```php
namespace App\Resolvers;

use Illuminate\Support\Facades\Request;
use OwenIt\Auditing\Contracts\Auditable;
use OwenIt\Auditing\Contracts\Resolver;

class IpAddressResolver implements Resolver
{
    public static function resolve(Auditable $auditable): string
    {
        return Request::header('HTTP_X_FORWARDED_FOR') ?? Request::ip();
    }
}
```

### UserAgentResolver

```php
namespace App\Resolvers;

use Illuminate\Support\Facades\Request;
use OwenIt\Auditing\Contracts\Auditable;
use OwenIt\Auditing\Contracts\Resolver;

class UserAgentResolver implements Resolver
{
    public static function resolve(Auditable $auditable): string
    {
        return Request::header('User-Agent', 'N/A');
    }
}
```

### UrlResolver

```php
namespace App\Resolvers;

use Illuminate\Support\Facades\App;
use Illuminate\Support\Facades\Request;
use OwenIt\Auditing\Contracts\Auditable;
use OwenIt\Auditing\Contracts\Resolver;

class UrlResolver implements Resolver
{
    public static function resolve(Auditable $auditable): string
    {
        if (App::runningInConsole()) {
            return 'console';
        }
        return Request::url();
    }
}
```

## Model Configuration

### Required Setup

```php
use OwenIt\Auditing\Auditable;
use OwenIt\Auditing\Contracts\Auditable as ContractAuditable;

class {Entity} extends Model implements ContractAuditable
{
    use HasFactory, HasUuids, SoftDeletes, Auditable;

    protected $primaryKey = 'uuid';
    protected $keyType = 'string';

    // Exclude sensitive fields
    protected $auditExclude = [
        'password',
        'remember_token',
        'updated_at',
    ];
}
```

### User Model

```php
use OwenIt\Auditing\Auditable;
use OwenIt\Auditing\Contracts\Auditable as ContractAuditable;

class User extends Authenticatable implements ContractAuditable
{
    use Auditable;

    protected $auditExclude = [
        'password',
        'remember_token',
    ];
}
```

### Optional: Include Only Specific Fields

```php
protected $auditInclude = [
    'name',
    'email',
    'status',
];
```

### Optional: Custom Events

```php
protected $auditEvents = [
    'created',
    'updated',
    // 'deleted' not audited
];
```

### Optional: Custom Tags

```php
public function generateTags(): array
{
    return ['entity_management', $this->status];
}
```

## Resources

### AuditResource

```php
namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class AuditResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'user_type' => $this->user_type,
            'user_id' => $this->user_id,
            'event' => $this->event,
            'auditable_type' => $this->auditable_type,
            'auditable_id' => $this->auditable_id,
            'old_values' => $this->sanitizeValues($this->old_values),
            'new_values' => $this->sanitizeValues($this->new_values),
            'url' => $this->url,
            'ip_address' => $this->ip_address,
            'user_agent' => $this->user_agent,
            'tags' => $this->tags,
            'created_at' => $this->created_at?->toISOString(),
            'updated_at' => $this->updated_at?->toISOString(),
        ];
    }

    protected function sanitizeValues($values): ?array
    {
        if (!$values) return null;
        $sensitive = ['password', 'remember_token', 'api_token', 'secret', 'key', 'token'];
        foreach ($sensitive as $field) {
            if (isset($values[$field])) $values[$field] = '**********';
        }
        return $values;
    }
}
```

### AuditCollection

```php
namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class AuditCollection extends ResourceCollection
{
    public $collects = AuditResource::class;
}
```

## Disable Auditing Temporarily

```php
// For a closure
User::withoutAuditing(function () {
    User::create([...]);
});

// For an instance
$user->disableAuditing();
$user->update(['name' => 'New']);
$user->enableAuditing();
```

## Workflow

1. Install package
2. Publish config and migration
3. Edit config/audit.php (add resolvers, guards)
4. Edit migration (UUID support)
5. Create Resolvers (IpAddress, UserAgent, Url)
6. Create Resources (AuditResource, AuditCollection)
7. Run migrations
8. Add Auditable to Models
9. Run `vendor/bin/pint --dirty`

## Commands

```bash
php artisan migrate
php artisan config:clear
```

---

# Model Audit Checker

Verifies if models are correctly configured for auditing.

## Detection Commands

```bash
# Models without Auditable interface
grep -L "ContractAuditable" app/Models/*.php

# Models without Auditable trait
grep -L "use Auditable" app/Models/*.php | grep -v "User.php"

# Models with password but no auditExclude
grep -l "password" app/Models/*.php | xargs grep -L "auditExclude"
```

## Checklist

| Check | Required | Description |
|-------|----------|-------------|
| `implements ContractAuditable` | ✅ | Interface required |
| `use Auditable` | ✅ | Trait required |
| `$auditExclude` for password | ✅ | Never audit passwords |
| `$auditExclude` for remember_token | ✅ | Never audit tokens |
| `$auditExclude` for updated_at | Optional | Reduce noise |

## Bad Signs in Models

- Missing `ContractAuditable` interface
- Missing `Auditable` trait
- Password field without `$auditExclude`
- Token fields without `$auditExclude`

## Fix Prompt Format

```
@whyll-agents:model-fixer Fix {Name} - add Auditable
File: app/Models/{Name}.php
Issues: missing ContractAuditable, missing Auditable trait, password not excluded
```

## Correct Model Pattern

```php
<?php

namespace App\Models;

use OwenIt\Auditing\Auditable;
use OwenIt\Auditing\Contracts\Auditable as ContractAuditable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class {Entity} extends Model implements ContractAuditable
{
    use HasFactory, HasUuids, SoftDeletes, Auditable;

    protected $primaryKey = 'uuid';
    public $incrementing = false;
    protected $keyType = 'string';

    protected $fillable = [/*...*/];

    protected $auditExclude = [
        'updated_at',
    ];

    public function uniqueIds(): array
    {
        return ['uuid'];
    }
}
```

## Detection Check

```bash
# Check if audit is configured
ls config/audit.php 2>/dev/null
ls app/Resolvers/IpAddressResolver.php 2>/dev/null
grep -l "owen-it/laravel-auditing" composer.json
```

If exists → verify models. If not → run full setup.

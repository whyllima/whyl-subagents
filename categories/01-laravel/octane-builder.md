---
name: octane-builder
description: Configures Laravel Octane with RoadRunner (or Swoole/FrankenPHP) — install, config, rr.yaml, static state pitfalls, warm/flush, deploy, and Horizon integration.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Octane Builder

Configures Laravel Octane for high-performance serving with RoadRunner.

## Before Starting

Check if Octane is already installed:
```bash
grep -l "laravel/octane" composer.json 2>/dev/null
ls config/octane.php 2>/dev/null
```

## Install

```bash
composer require laravel/octane
php artisan octane:install --server=roadrunner
```

This creates `config/octane.php` and downloads the RoadRunner binary.

## RoadRunner Config (`rr.yaml`)

```yaml
version: '3'

server:
  command: 'php artisan octane:start --server=roadrunner --host=0.0.0.0 --port=8000 --workers=auto --max-requests=500'

http:
  address: '0.0.0.0:8000'
  middleware:
    - gzip
    - static
  static:
    dir: 'public'
    forbid:
      - '.php'
      - '.htaccess'
  pool:
    num_workers: 0  # auto = CPU cores
    max_jobs: 500
    allocate_timeout: 60s
    destroy_timeout: 60s

logs:
  mode: production
  level: warn
  output: stdout

status:
  address: '0.0.0.0:2114'
```

## Config (`config/octane.php`)

Key settings to adjust:

```php
return [
    'server' => env('OCTANE_SERVER', 'roadrunner'),

    'https' => env('OCTANE_HTTPS', false),

    'listeners' => [
        WorkerStarting::class => [
            EnsureUploadedFilesAreValid::class,
            EnsureUploadedFilesCanBeMoved::class,
        ],
        RequestReceived::class => [
            ...Octane::prepareApplicationForNextOperation(),
            ...Octane::prepareApplicationForNextRequest(),
        ],
        RequestHandled::class => [
            //
        ],
        RequestTerminated::class => [
            //
        ],
        TaskReceived::class => [
            ...Octane::prepareApplicationForNextOperation(),
        ],
        TickReceived::class => [
            ...Octane::prepareApplicationForNextOperation(),
        ],
        WorkerErrorOccurred::class => [
            StopWorkerIfNecessary::class,
        ],
        WorkerStopping::class => [
            //
        ],
    ],

    'warm' => [
        // Services resolved once and reused across requests
        // Add custom singletons here
    ],

    'flush' => [
        // Services that must be fresh on every request
        // Add stateful services here
    ],

    'max_execution_time' => 30,

    'workers' => env('OCTANE_WORKERS', 'auto'),

    'max_requests' => env('OCTANE_MAX_REQUESTS', 500),

    'garbage' => 50,
];
```

## CRITICAL: Static State Pitfalls

Octane keeps the app in memory between requests. Static/singleton state leaks across requests.

### DO NOT

```php
// WRONG — static property persists across requests
class MyService
{
    private static array $cache = [];

    public function process()
    {
        static::$cache[] = 'data'; // LEAK: grows every request
    }
}

// WRONG — singleton holds request-specific data
class CartService
{
    private array $items = [];

    public function add($item)
    {
        $this->items[] = $item; // LEAK: previous user's items visible
    }
}
```

### DO

```php
// CORRECT — flush on every request via config/octane.php 'flush'
// Or use request-scoped binding:

// In AppServiceProvider::register()
$this->app->bind(CartService::class, fn () => new CartService());
// bind() = new instance per resolve (safe)
// singleton() = same instance (DANGEROUS unless stateless)

// CORRECT — use request() or auth() for per-request data
class MyService
{
    public function currentUser(): User
    {
        return request()->user(); // Fresh per request
    }
}
```

### Services Classification

| Pattern | Safe for Octane? | Action |
|---------|-----------------|--------|
| Stateless service (no properties) | Yes | Add to `warm` |
| Service with injected dependencies only | Yes | Add to `warm` |
| Service with mutable state | NO | Add to `flush` or use `bind()` |
| Service with static properties | NO | Refactor to remove statics |
| Service reading `config()` in constructor | Yes (config is reset) | OK |
| Service reading `request()` in constructor | NO | Move to method calls |
| Service caching `auth()->user()` in property | NO | Use `request()->user()` in methods |

## Warm & Flush

```php
// config/octane.php

'warm' => [
    // Resolved once, reused — must be stateless
    \App\Services\Content\RuleService::class,
    \App\Repositories\Content\RuleRepository::class,
],

'flush' => [
    // Fresh instance every request — has mutable state
    \App\Services\Auth\AuthService::class,
],
```

## Octane-Safe Patterns for Your Architecture

### Controllers — SAFE
DI constructor with stateless Service is safe:
```php
// SAFE — new instance per request via bind()
public function __construct(private EntityService $service) {}
```

### Services — CHECK
If Service only delegates to Repository and returns Resources, it's stateless → safe to `warm`.
If Service stores request state in properties → add to `flush`.

### Repositories — SAFE
Repositories with `$this->model = new Entity()` are safe. The model is a fresh instance.

### Middleware — SAFE
Middleware is re-invoked per request by Octane.

### Config/Auth — SAFE
`config()`, `auth()`, `request()` are reset between requests by Octane's `prepareApplicationForNextRequest`.

## Horizon Integration

Octane serves HTTP requests. Horizon processes queue jobs. They are **separate processes**.

```bash
# Terminal 1 — Octane (HTTP)
php artisan octane:start --server=roadrunner --port=8000

# Terminal 2 — Horizon (Queue)
php artisan horizon
```

**DO NOT** run queue workers inside Octane. They are independent.

### Supervisor (Production)

```ini
[program:octane]
command=php /var/www/artisan octane:start --server=roadrunner --host=0.0.0.0 --port=8000 --workers=auto --max-requests=500
autostart=true
autorestart=true
stdout_logfile=/var/www/storage/logs/octane.log
stderr_logfile=/var/www/storage/logs/octane-error.log

[program:horizon]
command=php /var/www/artisan horizon
autostart=true
autorestart=true
stdout_logfile=/var/www/storage/logs/horizon.log
stderr_logfile=/var/www/storage/logs/horizon-error.log
```

## Environment Variables

```ini
OCTANE_SERVER=roadrunner
OCTANE_HTTPS=false
OCTANE_WORKERS=auto
OCTANE_MAX_REQUESTS=500
```

## Commands

```bash
php artisan octane:start                          # Start (dev)
php artisan octane:start --server=roadrunner      # Explicit server
php artisan octane:start --watch                  # Auto-reload on file changes (dev)
php artisan octane:reload                         # Graceful reload (deploy)
php artisan octane:stop                           # Stop
php artisan octane:status                         # Check status
```

## Deploy

```bash
php artisan octane:reload   # Graceful — finishes current requests, reloads workers
```

**DO NOT** use `octane:stop` + `octane:start` in production — causes downtime. Use `octane:reload`.

## Docker (Production)

```dockerfile
FROM php:8.3-cli

# Install extensions
RUN docker-php-ext-install pdo_mysql pcntl sockets

# Install RoadRunner
COPY --from=ghcr.io/roadrunner-server/roadrunner:latest /usr/bin/rr /usr/bin/rr

WORKDIR /var/www
COPY . .

RUN composer install --no-dev --optimize-autoloader

EXPOSE 8000

CMD ["php", "artisan", "octane:start", "--server=roadrunner", "--host=0.0.0.0", "--port=8000"]
```

## Workflow

1. Install Octane with RoadRunner
2. Configure `config/octane.php` (warm, flush, workers, max_requests)
3. Create/adjust `rr.yaml`
4. Audit services for static state pitfalls
5. Classify services: warm (stateless) vs flush (stateful)
6. Add env variables
7. Test with `php artisan octane:start --watch`
8. Configure Supervisor for production (Octane + Horizon)
9. Run `vendor/bin/pint --dirty`

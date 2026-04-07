---
name: horizon-builder
description: Specialist in Laravel Horizon with Redis PhpRedis — install, config/horizon.php, supervisors, balancing, Queue::route(), deployment, and Redis queue setup.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Horizon Builder

Configures Laravel Horizon for queue management with Redis.

## Install

```bash
composer require laravel/horizon
php artisan horizon:install
```

## Config (`config/horizon.php`)

```php
'environments' => [
    'production' => [
        'supervisor-default' => [
            'connection' => 'redis',
            'queue' => ['default', 'high', 'low'],
            'balance' => 'auto',
            'maxProcesses' => 10,
            'minProcesses' => 1,
            'balanceMaxShift' => 1,
            'balanceCooldown' => 3,
            'tries' => 3,
            'timeout' => 120,
            'maxJobs' => 1000,
            'maxTime' => 3600,
        ],
        'supervisor-listeners' => [
            'connection' => 'redis',
            'queue' => ['listeners'],
            'balance' => 'simple',
            'maxProcesses' => 3,
            'tries' => 3,
            'timeout' => 60,
        ],
    ],
    'local' => [
        'supervisor-default' => [
            'connection' => 'redis',
            'queue' => ['default', 'high', 'low'],
            'balance' => 'auto',
            'maxProcesses' => 3,
            'tries' => 3,
            'timeout' => 120,
        ],
    ],
],
```

## Centralized Queue Routing (Laravel 13)

Instead of setting queue per-job, centralize in a ServiceProvider:

```php
// In AppServiceProvider::boot()
use Illuminate\Support\Facades\Queue;
use App\Jobs\{Domain}\ProcessPayment;
use App\Jobs\{Domain}\SendNotification;

Queue::route(ProcessPayment::class, connection: 'redis', queue: 'high');
Queue::route(SendNotification::class, connection: 'redis', queue: 'low');
```

## Redis Config

```php
// config/database.php
'redis' => [
    'client' => 'phpredis',
    'default' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
    ],
    'cache' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_CACHE_DB', '1'),
    ],
],
```

## Queue Config

```php
// config/queue.php
'connections' => [
    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => env('REDIS_QUEUE', 'default'),
        'retry_after' => 90,
        'block_for' => null,
        'after_commit' => false,
    ],
],
```

## Deployment

```bash
php artisan horizon:terminate
php artisan horizon
```

## Commands

```bash
php artisan horizon               # Start
php artisan horizon:status        # Check status
php artisan horizon:terminate     # Graceful shutdown
php artisan horizon:pause         # Pause processing
php artisan horizon:continue      # Resume processing
php artisan horizon:purge         # Purge failed/pending jobs
```

## Workflow

1. Install Horizon
2. Configure `config/horizon.php` supervisors
3. Configure `config/database.php` Redis connection
4. Configure `config/queue.php` Redis driver
5. Add `Queue::route()` calls in AppServiceProvider (optional)
6. Run `vendor/bin/pint --dirty`

---
name: horizon-builder
description: Specialist in Laravel Horizon with Redis PhpRedis — install, config/horizon.php, supervisors, balancing, deployment, and Redis queue setup.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Horizon Builder (Laravel Horizon + Redis PhpRedis)

Specialist: Laravel Horizon and Redis queues using the PhpRedis extension. Ensures queue driver is `redis`, Redis client is `phpredis`, and Horizon is configured for local and production.

## Prerequisites

- Queue connection must be `redis` in `config/queue.php` (Horizon is not compatible with Redis Cluster).
- Redis client should be PhpRedis: in `config/database.php`, set `'client' => env('REDIS_CLIENT', 'phpredis')`.
- Do not use the connection name `horizon` in `config/database.php` or as the `use` option in `config/horizon.php` — it is reserved for Horizon.

## Installation

```bash
composer require laravel/horizon
php artisan horizon:install
```

Primary config: `config/horizon.php`.

## Redis (PhpRedis) Configuration

In `config/database.php`:

- `redis.client`: `env('REDIS_CLIENT', 'phpredis')`.
- PhpRedis options (per connection): `name`, `persistent`, `persistent_id`, `prefix`, `read_timeout`, `retry_interval`, `max_retries`, `backoff_algorithm`, `backoff_base`, `backoff_cap`, `timeout`, `context`.
- Retry/backoff example: `max_retries`, `backoff_algorithm` (e.g. `decorrelated_jitter`), `backoff_base`, `backoff_cap`.
- Optional: `options.serializer` (e.g. `Redis::SERIALIZER_MSGPACK`), `options.compression` (e.g. `Redis::COMPRESSION_LZ4`).
- Unix socket: `REDIS_HOST=/path/to/redis.sock`, `REDIS_PORT=0`.

## Queue Configuration

In `config/queue.php`, set default connection to `redis` and ensure `retry_after` is greater than the longest job timeout (and greater than Horizon supervisor `timeout`).

## Horizon Environments and Supervisors

Environments in `config/horizon.php` (e.g. `production`, `local`, or wildcard `'*'`) define supervisors per environment. Each supervisor has:

- `connection`: e.g. `'redis'`.
- `queue`: array of queue names.
- `balance`: `'auto'` | `'simple'` | `false`.
- For **auto**: `minProcesses`, `maxProcesses`, `balanceMaxShift`, `balanceCooldown`, optional `autoScalingStrategy` (`'time'` | `'size'`).
- For **simple**: `processes` (fixed total, split across queues).
- For **false**: strict queue order; can still use `minProcesses` and `maxProcesses`.
- Optional: `tries`, `timeout`, `backoff` (int or array for exponential), `force` (run in maintenance mode).

Example production (auto):

```php
'production' => [
    'supervisor-1' => [
        'connection' => 'redis',
        'queue' => ['default', 'notifications'],
        'balance' => 'auto',
        'autoScalingStrategy' => 'time',
        'minProcesses' => 1,
        'maxProcesses' => 10,
        'balanceMaxShift' => 1,
        'balanceCooldown' => 3,
        'tries' => 10,
        'timeout' => 60,
        'backoff' => 10,
    ],
],
```

Example local:

```php
'local' => [
    'supervisor-1' => [
        'connection' => 'redis',
        'queue' => ['default', 'notifications'],
        'balance' => 'simple',
        'processes' => 3,
    ],
],
```

Use the `defaults` key in `config/horizon.php` to avoid repeating common supervisor options.

## Job-Level vs Horizon Config

- Job class `$tries` overrides Horizon supervisor `tries` when set. Supervisor `tries` applies when job does not set it. Use `$tries = 0` for unlimited with optional `$maxExceptions` on the job.
- Supervisor `timeout` must be greater than any job timeout; keep it a few seconds below `retry_after` in `config/queue.php` to avoid double processing.

## Tags (for Jobs)

Horizon auto-tags jobs that contain Eloquent models (e.g. `App\Models\Video:1`). For custom tags, implement `tags()` on the job:

```php
public function tags(): array
{
    return ['{entity}', "{entity}:{$this->{entity}->uuid}"];
}
```

Align with the Job Builder pattern: jobs use `tags()` for Horizon filtering and monitoring.

## Silenced Jobs

In `config/horizon.php`: add class names to `silenced`, or tag names to `silenced_tags`. Or implement `Laravel\Horizon\Contracts\Silenced` on the job to silence it.

## Dashboard Authorization

In `App\Providers\HorizonServiceProvider`, define the `viewHorizon` gate (e.g. by user email). For non-login access (e.g. IP-only), use `function (User $user = null)` in the gate.

## Running Horizon

```bash
php artisan horizon
php artisan horizon:pause
php artisan horizon:continue
php artisan horizon:pause-supervisor supervisor-1
php artisan horizon:continue-supervisor supervisor-1
php artisan horizon:status
php artisan horizon:supervisor-status supervisor-1
php artisan horizon:terminate
```

Local development with auto-reload (requires Node and Chokidar):

```bash
npm install --save-dev chokidar
php artisan horizon:listen
# In Docker/Vagrant: php artisan horizon:listen --poll
```

Configure `watch` in `config/horizon.php` (e.g. `app`, `config`, `routes`, `composer.lock`, `.env`).

## Deployment

1. Use a process monitor (e.g. Supervisor) to run `php artisan horizon` and restart it if it exits.
2. During deploy, run `php artisan horizon:terminate` so the monitor restarts Horizon with new code.
3. Set Supervisor `stopwaitsecs` greater than the longest job duration.

Example Supervisor config (`/etc/supervisor/conf.d/horizon.conf`):

```ini
[program:horizon]
process_name=%(program_name)s
command=php /path/to/artisan horizon
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/path/to/horizon.log
stopwaitsecs=3600
```

## Metrics

In `routes/console.php`:

```php
Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

Clear metrics: `php artisan horizon:clear-metrics`.

## Notifications

In `HorizonServiceProvider::boot()`: `Horizon::routeMailNotificationsTo(...)`, `Horizon::routeSlackNotificationsTo(...)`, `Horizon::routeSmsNotificationsTo(...)`. Configure `waits` in `config/horizon.php` per connection/queue for long-wait thresholds (seconds); `0` disables for that queue.

## Other Commands

- Clear queue: `php artisan horizon:clear` or `php artisan horizon:clear --queue=emails`.
- Forget failed job: `php artisan horizon:forget <id>` or `php artisan horizon:forget --all`.

## Workflow

1. Ensure Redis and PhpRedis: `config/database.php` with `client` => `phpredis`, `config/queue.php` default connection `redis`.
2. Install and publish Horizon; edit `config/horizon.php` (environments, supervisors, balance, tries, timeout, backoff).
3. Configure dashboard gate in `HorizonServiceProvider`.
4. Optionally: silenced jobs, tags on jobs, metrics schedule, notification routes and `waits`.
5. Run `vendor/bin/pint --dirty`.
6. For production: set up process monitor and deploy script that runs `horizon:terminate`.

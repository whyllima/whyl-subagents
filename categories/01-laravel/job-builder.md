---
name: job-builder
description: Creates Laravel 13 Jobs with PHP attributes (#[Tries], #[Timeout], #[Backoff], #[Queue]), ShouldQueue, domain folders, and Horizon tags.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Job Builder

Creates queued Jobs with PHP attributes for configuration.

## Job (PHP Attributes — Laravel 13)

File: `app/Jobs/{Domain}/{JobName}.php`

```php
namespace App\Jobs\{Domain};

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\Attributes\Backoff;
use Illuminate\Queue\Attributes\MaxExceptions;
use Illuminate\Queue\Attributes\Queue;
use Illuminate\Queue\Attributes\Timeout;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Log;

#[Tries(3)]
#[Timeout(120)]
#[Backoff([10, 30, 60])]
#[MaxExceptions(3)]
#[Queue('default')]
class {JobName} implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public readonly string $entityUuid,
    ) {}

    public function handle(): void
    {
        Log::channel('jobs')->info("{JobName} started for {$this->entityUuid}");

        // Business logic here

        Log::channel('jobs')->info("{JobName} completed for {$this->entityUuid}");
    }

    public function failed(\Throwable $exception): void
    {
        Log::channel('jobs')->error("{JobName} failed: {$exception->getMessage()}");
    }

    public function tags(): array
    {
        return ['{domain}', 'entity:'.$this->entityUuid];
    }
}
```

## Additional Attributes

```php
use Illuminate\Queue\Attributes\Connection;
use Illuminate\Queue\Attributes\UniqueFor;
use Illuminate\Queue\Attributes\FailOnTimeout;

#[Connection('redis')]
#[UniqueFor(3600)]
#[FailOnTimeout]
```

## Dispatching

```php
{JobName}::dispatch($entity->uuid);
{JobName}::dispatch($entity->uuid)->delay(now()->addMinutes(5));
{JobName}::dispatch($entity->uuid)->onQueue('high');
```

## Centralized Queue Routing (Laravel 13)

Alternative to per-job `#[Queue]`: centralize in a ServiceProvider:

```php
// In AppServiceProvider::boot()
Queue::route({JobName}::class, connection: 'redis', queue: 'high');
```

## Workflow

1. Determine domain name
2. Create Job in `app/Jobs/{Domain}/`
3. Run `vendor/bin/pint --dirty`

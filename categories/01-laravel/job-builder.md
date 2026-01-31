---
name: job-builder
description: Creates Laravel Jobs with ShouldQueue, retry logic, and Horizon tags.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Job Builder

Creates: Queue Jobs with Horizon support

## Pattern

```php
class Process{Entity}Job implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 3;
    public int $timeout = 120;
    public int $backoff = 60;

    public function __construct(public {Entity} ${entity}) {}

    public function handle(): void
    {
        try {
            Log::channel('jobs')->info("Processing: {$this->{entity}->uuid}");
            // Job logic...
        } catch (Exception $e) {
            Log::channel('jobs')->error("Failed: {$e->getMessage()}");
            throw $e;
        }
    }

    public function failed(Exception $e): void
    {
        Log::channel('jobs')->error("Job failed: {$this->{entity}->uuid}");
    }

    public function tags(): array { return ['{entity}', "{entity}:{$this->{entity}->uuid}"]; }
}
```

## Dispatch

```php
Process{Entity}Job::dispatch($entity);
Process{Entity}Job::dispatch($entity)->onQueue('processing');
Process{Entity}Job::dispatch($entity)->delay(now()->addMinutes(5));
```

## Workflow

1. Create job file
2. Run `vendor/bin/pint --dirty`

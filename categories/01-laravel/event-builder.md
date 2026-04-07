---
name: event-builder
description: Creates Laravel 13 Events with PHP attributes, Listeners (sync/queued), broadcasting with Reverb, domain folders, and channel authorization.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Event Builder

Creates Events (normal + broadcastable), Listeners (sync + queued), and channel authorization.

## Event (in `app/Events/{Domain}/`)

```php
namespace App\Events\{Domain};

use App\Models\{Domain}\{Entity};
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class {Entity}Created implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public function __construct(
        public readonly {Entity} $entity,
    ) {}

    public function broadcastOn(): array
    {
        return [
            new PrivateChannel("entity.{$this->entity->uuid}"),
        ];
    }

    public function broadcastWith(): array
    {
        return ['uuid' => $this->entity->uuid];
    }
}
```

## Non-Broadcasting Event

```php
namespace App\Events\{Domain};

use App\Models\{Domain}\{Entity};
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class {Entity}Updated
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public readonly {Entity} $entity,
    ) {}
}
```

## Listener — Queued (PHP Attributes — Laravel 13)

```php
namespace App\Listeners\{Domain};

use App\Events\{Domain}\{Entity}Created;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Queue;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\Attributes\Timeout;

#[Queue('listeners')]
#[Tries(3)]
#[Timeout(60)]
class Send{Entity}Notification implements ShouldQueue
{
    public function handle({Entity}Created $event): void
    {
        // Notification logic
    }
}
```

## Listener — Sync

```php
namespace App\Listeners\{Domain};

use App\Events\{Domain}\{Entity}Updated;

class Log{Entity}Updated
{
    public function handle({Entity}Updated $event): void
    {
        // Sync logging
    }
}
```

## Channel Authorization

```php
// routes/channels.php
Broadcast::channel('entity.{uuid}', function ($user, $uuid) {
    return $user->can('view', \App\Models\{Domain}\{Entity}::findOrFail($uuid));
});
```

## Dispatching

```php
{Entity}Created::dispatch($entity);
event(new {Entity}Updated($entity));
```

## Workflow

1. Determine domain name
2. Create Event(s) in `app/Events/{Domain}/`
3. Create Listener(s) in `app/Listeners/{Domain}/`
4. Add channel authorization in `routes/channels.php` (if broadcasting)
5. Run `vendor/bin/pint --dirty`

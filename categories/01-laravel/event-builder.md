---
name: event-builder
description: Creates Laravel Events, Listeners, and Broadcasting with Reverb.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# Event Builder

Creates: Events + Listeners + Channel Authorization

## Standard Event

```php
class {Entity}Created
{
    use Dispatchable, SerializesModels;
    public function __construct(public {Entity} ${entity}) {}
}
```

## Broadcastable Event

```php
class {Entity}Updated implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;
    
    public function __construct(public {Entity} ${entity}) {}
    
    public function broadcastOn(): array { return [new PrivateChannel("{entities}.{$this->{entity}->uuid}")]; }
    public function broadcastAs(): string { return '{entity}.updated'; }
    public function broadcastWith(): array { return ['uuid' => $this->{entity}->uuid]; }
}
```

## Queued Listener

```php
class Send{Entity}Notification implements ShouldQueue
{
    use InteractsWithQueue;
    public string $queue = 'notifications';
    public int $tries = 3;

    public function handle({Entity}Created $event): void
    {
        Log::channel('events')->info("{Entity} created: {$event->{entity}->uuid}");
        // notification logic...
    }

    public function failed({Entity}Created $event, Throwable $e): void
    {
        Log::channel('events')->error("Failed: {$e->getMessage()}");
    }
}
```

## Channel (routes/channels.php)

```php
Broadcast::channel('{entities}.{uuid}', function (User $user, string $uuid) {
    ${entity} = {Entity}::find($uuid);
    return ${entity} && $user->uuid === ${entity}->author_uuid;
});
```

## Dispatch

```php
{Entity}Created::dispatch(${entity});
// or in Model: protected $dispatchesEvents = ['created' => {Entity}Created::class];
```

## Workflow

1. Create Event(s)
2. Create Listener(s)
3. Add channel authorization
4. Run `vendor/bin/pint --dirty`

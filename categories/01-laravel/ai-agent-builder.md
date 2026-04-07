---
name: ai-agent-builder
description: Creates Laravel 13 AI Agents with tools, structured output, conversation memory, streaming, and broadcasting using the Laravel AI SDK.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# AI Agent Builder

Creates AI Agents using Laravel AI SDK (`laravel/ai`).

## Before Starting

Check if AI SDK is installed:
```bash
grep -l "laravel/ai" composer.json 2>/dev/null
```

If NOT installed:
```bash
composer require laravel/ai
php artisan vendor:publish --provider="Laravel\Ai\AiServiceProvider"
php artisan migrate
```

This creates `agent_conversations` and `agent_conversation_messages` tables.

## Agent Class

File: `app/Ai/Agents/{AgentName}.php`

```php
namespace App\Ai\Agents;

use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\Conversational;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Concerns\RemembersConversations;
use Laravel\Ai\Promptable;
use Illuminate\Contracts\JsonSchema\JsonSchema;

class {AgentName} implements Agent, HasTools, HasStructuredOutput, Conversational
{
    use Promptable, RemembersConversations;

    public function __construct(
        public readonly ?string $context = null,
    ) {}

    public function instructions(): string
    {
        return <<<'PROMPT'
        You are a specialized assistant for {description}.
        PROMPT;
    }

    public function tools(): iterable
    {
        return [
            // new \App\Ai\Tools\{ToolName},
        ];
    }

    public function schema(JsonSchema $schema): array
    {
        return [
            'result' => $schema->string()->required(),
            'confidence' => $schema->number()->min(0)->max(1)->required(),
        ];
    }
}
```

## Simple Agent (no tools, no structured output)

```php
namespace App\Ai\Agents;

use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Promptable;

class {AgentName} implements Agent
{
    use Promptable;

    public function instructions(): string
    {
        return 'You are a helpful assistant.';
    }
}
```

## Tool

File: `app/Ai/Tools/{ToolName}.php`

```php
namespace App\Ai\Tools;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Tool;
use Laravel\Ai\Tools\Request;

class {ToolName} implements Tool
{
    public function description(): string
    {
        return 'Describe what this tool does.';
    }

    public function handle(Request $request): string
    {
        $param = $request['param_name'];
        return "Result: {$param}";
    }

    public function schema(JsonSchema $schema): array
    {
        return [
            'param_name' => $schema->string()->required(),
        ];
    }
}
```

## Using Agents

```php
// Basic prompt
$response = (new {AgentName})->prompt('Analyze this...');
$text = (string) $response;

// With provider override
use Laravel\Ai\Enums\Lab;
$response = (new {AgentName})->prompt('...', provider: Lab::Anthropic, model: 'claude-haiku-4-5-20251001');

// Anonymous agent (quick one-off)
$response = agent(instructions: 'You are a helpful assistant.')->prompt('Hello!');

// Failover across providers
$response = (new {AgentName})->prompt('...', provider: ['anthropic', 'openai', 'gemini']);

// Structured output access
$response = (new {AgentName})->prompt('...');
$result = $response['result'];
```

## Conversation Memory

```php
// Start conversation
$response = (new ChatAgent)->forUser($user)->prompt('Hello!');
$conversationId = $response->conversationId;

// Continue conversation
$response = (new ChatAgent)->continue($conversationId, as: $user)->prompt('Tell me more.');
```

## Streaming (SSE)

```php
return (new {AgentName})
    ->stream('Analyze this...')
    ->then(fn ($response) => Log::info($response->text));
```

## Broadcasting (via Reverb)

```php
use Illuminate\Broadcasting\Channel;

(new {AgentName})->broadcastOnQueue('Analyze this...', new Channel('agent-results'));
```

## Image Generation

```php
use Laravel\Ai\Image;

$image = Image::of('A donut on the kitchen counter')->landscape()->quality('high')->generate();
$path = $image->store('images', 's3');
```

## Embeddings & Vector Search

```php
use Laravel\Ai\Embeddings;

// Generate embeddings
$embedding = Str::of('Napa Valley has great wine.')->toEmbeddings();

// Vector search (requires pgvector + vector column in migration)
$docs = Document::query()->whereVectorSimilarTo('embedding', 'search query')->limit(10)->get();
```

## Vector Migration (PostgreSQL + pgvector)

```php
Schema::ensureVectorExtensionExists();

Schema::create('documents', function (Blueprint $table) {
    $table->id();
    $table->text('content');
    $table->vector('embedding', dimensions: 1536)->index();
    $table->timestamps();
});
```

## Environment Variables

```ini
ANTHROPIC_API_KEY=
OPENAI_API_KEY=
GEMINI_API_KEY=
```

## Workflow

1. Check if `laravel/ai` is installed (install if not)
2. Create Agent in `app/Ai/Agents/`
3. Create Tool(s) in `app/Ai/Tools/` (if needed)
4. Add env variables for chosen provider
5. Run `vendor/bin/pint --dirty`

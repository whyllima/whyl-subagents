---
name: mcp-server-builder
description: Creates Laravel 13 MCP servers with tools, resources, and prompts using laravel/mcp for exposing app functionality to external AI clients.
tools: Read, Write, Edit, Bash, Glob, Grep
---

# MCP Server Builder

Creates Model Context Protocol (MCP) servers to expose your Laravel app to external AI clients.

## Before Starting

Check if MCP package is installed:
```bash
grep -l "laravel/mcp" composer.json 2>/dev/null
```

If NOT installed:
```bash
composer require laravel/mcp
php artisan vendor:publish --tag=ai-routes
```

This creates `routes/ai.php`.

## MCP Server

File: `app/Mcp/Servers/{ServerName}Server.php`

```php
namespace App\Mcp\Servers;

use Laravel\Mcp\Server;

class {ServerName}Server extends Server
{
    public function name(): string
    {
        return '{server-name}';
    }

    public function description(): string
    {
        return 'Provides access to {description}.';
    }

    public function tools(): array
    {
        return [
            new \App\Mcp\Tools\{ToolName}Tool,
        ];
    }

    public function resources(): array
    {
        return [
            new \App\Mcp\Resources\{ResourceName}Resource,
        ];
    }

    public function prompts(): array
    {
        return [
            new \App\Mcp\Prompts\{PromptName}Prompt,
        ];
    }
}
```

## MCP Tool

File: `app/Mcp/Tools/{ToolName}Tool.php`

```php
namespace App\Mcp\Tools;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Mcp\Attributes\Description;
use Laravel\Mcp\Attributes\Name;
use Laravel\Mcp\Attributes\Title;
use Laravel\Mcp\Contracts\Tool;
use Laravel\Mcp\Tools\Request;
use Laravel\Mcp\Tools\Response;

#[Name('{tool-name}')]
#[Title('{Tool Title}')]
#[Description('{What this tool does}')]
class {ToolName}Tool extends Tool
{
    public function handle(Request $request): Response
    {
        $validated = $request->validated();
        $result = app(\App\Services\{Domain}\{Service}::class)->doSomething($validated['param']);
        return Response::text((string) $result);
    }

    public function schema(JsonSchema $schema): array
    {
        return [
            'param' => $schema->string()->required(),
        ];
    }
}
```

## MCP Resource

File: `app/Mcp/Resources/{ResourceName}Resource.php`

```php
namespace App\Mcp\Resources;

use Laravel\Mcp\Attributes\Description;
use Laravel\Mcp\Attributes\Name;
use Laravel\Mcp\Contracts\Resource;

#[Name('{resource-name}')]
#[Description('{What this resource provides}')]
class {ResourceName}Resource extends Resource
{
    public function uri(): string
    {
        return '{resource-name}://guidelines';
    }

    public function mimeType(): string
    {
        return 'text/plain';
    }

    public function read(): string
    {
        return 'Guidelines and documentation content here.';
    }
}
```

## MCP Prompt

File: `app/Mcp/Prompts/{PromptName}Prompt.php`

```php
namespace App\Mcp\Prompts;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Mcp\Attributes\Description;
use Laravel\Mcp\Attributes\Name;
use Laravel\Mcp\Contracts\Prompt;
use Laravel\Mcp\Prompts\Request;
use Laravel\Mcp\Prompts\Response;

#[Name('{prompt-name}')]
#[Description('{What this prompt generates}')]
class {PromptName}Prompt extends Prompt
{
    public function handle(Request $request): Response
    {
        return Response::text("Analyze the following: {$request['topic']}");
    }

    public function schema(JsonSchema $schema): array
    {
        return [
            'topic' => $schema->string()->required(),
        ];
    }
}
```

## Server Registration (`routes/ai.php`)

```php
use App\Mcp\Servers\{ServerName}Server;
use Laravel\Mcp\Facades\Mcp;

// Web server — HTTP for remote AI clients
Mcp::web('/mcp/{server-name}', {ServerName}Server::class)
    ->middleware(['throttle:mcp']);

// Local server — CLI for tools like Boost
Mcp::local('{server-name}', {ServerName}Server::class);
```

## Authentication

```php
// OAuth 2.1 (Passport)
Mcp::web('/mcp/{server-name}', {ServerName}Server::class)
    ->middleware(['auth:api', 'throttle:mcp']);

// Sanctum
Mcp::web('/mcp/{server-name}', {ServerName}Server::class)
    ->middleware(['auth:sanctum', 'throttle:mcp']);
```

## Artisan Commands

```bash
php artisan make:mcp-server {ServerName}Server
php artisan make:mcp-tool {ToolName}Tool
php artisan make:mcp-resource {ResourceName}Resource
php artisan make:mcp-prompt {PromptName}Prompt
```

## Workflow

1. Check if `laravel/mcp` is installed (install if not)
2. Create MCP Server in `app/Mcp/Servers/`
3. Create MCP Tool(s) in `app/Mcp/Tools/`
4. Create MCP Resource(s) in `app/Mcp/Resources/` (optional)
5. Create MCP Prompt(s) in `app/Mcp/Prompts/` (optional)
6. Register server in `routes/ai.php`
7. Run `vendor/bin/pint --dirty`

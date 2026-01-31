# WhylLima Subagents – Categories

Coleção de agentes especializados para o projeto WhylLima. Cada agente é focado em uma tarefa específica para otimizar uso de tokens e dar suporte preciso ao código.

---

## Categorias

### 01-laravel

Especialistas Laravel 12 para o backend WhylLima. Todos os agentes seguem o padrão Service-Repository com modelos UUID.

**Stack:** PHP 8.3 | Laravel 12 | Sanctum | Horizon | Reverb | PHPUnit

---

## Arquitetura

```
Controller (sem lógica) → Service (toda lógica) → Repository (acesso via Model) → Model
                                                                              ↓
Resposta sempre via Resource ←────────────────────────────────────────────────
```

**Regras:**
- **Controller:** só delega para Service, sem try-catch, sem Log, sem queries.
- **Service:** toda lógica de negócio, sempre retorna Resource.
- **Repository:** usa sempre **Model** (nunca facade `DB`).
- **Resource:** todas as respostas da API passam por Resource.

---

## Agentes – Builders (criar)

| Agente | Propósito | Cria |
|--------|-----------|------|
| `full-stack-specialist` | Feature completa do zero | Migration, Model, Repository, Service, Form Request, Controller, Routes, Resource |
| `model-builder` | Só estrutura de dados | Migration, Model |
| `api-layer-builder` | API para model existente | Repository, Service, Form Request, Controller, Routes, Resource |
| `endpoint-builder` | Só rotas | API Routes |
| `test-builder` | Testes PHPUnit | Feature Tests (DatabaseTransactions) |
| `job-builder` | Jobs em fila | Jobs, config Horizon |
| `resource-builder` | Transformação JSON | JsonResource, ResourceCollection |
| `seeder-builder` | Dados de teste/dev | Factories, Seeders |
| `event-builder` | Eventos e broadcasting | Events, Listeners, canais Reverb |

---

## Agentes – Qualidade e Fixers (auditar e corrigir)

| Agente | Propósito |
|--------|-----------|
| `quality-checker` | Audita código e gera prompts prontos para os fixers |
| `controller-fixer` | Remove lógica do Controller, delega para Service |
| `service-fixer` | Garante uso de Repository e retorno de Resources |
| `repository-fixer` | Converte uso de `DB` para queries via Model |
| `model-fixer` | Adiciona traits, UUID e relacionamentos corretos |
| `formrequest-fixer` | Ajusta validação para create/update com UUID |
| `resource-fixer` | Campos explícitos, datas ISO, `whenLoaded()` |

---

## Guia de decisão

**Criar algo novo**
- Feature completa do zero → `full-stack-specialist`
- Só banco de dados → `model-builder`
- Model existe, precisa de API → `api-layer-builder`
- Controller existe, só rotas → `endpoint-builder`
- API existe, precisa de testes → `test-builder`
- Processamento em background → `job-builder`
- Transformação JSON → `resource-builder`
- Dados de teste/dev → `seeder-builder`
- Eventos/real-time → `event-builder`

**Auditar e corrigir**
- Auditar feature ou arquivos → `quality-checker` (gera prompts para os fixers)
- Corrigir Controller → `controller-fixer`
- Corrigir Service → `service-fixer`
- Corrigir Repository → `repository-fixer`
- Corrigir Model → `model-fixer`
- Corrigir FormRequest → `formrequest-fixer`
- Corrigir Resource → `resource-fixer`

---

## Convenções

| Convenção | Padrão |
|-----------|--------|
| Primary Key | UUID, coluna `uuid` |
| Foreign Keys | `{entity}_uuid` |
| Traits do Model | HasFactory, HasUuids, SoftDeletes, AutoIncrementId, Auditable |
| Services | Estendem `App\Services\Service` |
| Repositories | Estendem `App\Repositories\Repository` |
| Repository | Nunca usa `DB::`; sempre `Model::query()` |
| Documentação | PHPDoc em inglês, sem comentários inline |
| Rotas | Agrupamento com `Route::controller()` |
| Formatação | `vendor/bin/pint --dirty` |

---

## Exemplos de uso

```text
# Criar feature completa
@whyll-agents:full-stack-specialist Create a complete feature for "Categories"

# Auditar e obter prompts de correção
@whyll-agents:quality-checker Audit Entity feature

# Corrigir um componente (usar prompt gerado pelo quality-checker)
@whyll-agents:controller-fixer Fix EntityController
File: app/Http/Controllers/EntityController.php
Issues: try-catch blocks, Log:: calls, direct queries
```

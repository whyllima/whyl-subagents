# WhylLima Subagents – Categories

Coleção de agentes especializados para o projeto WhylLima. Cada agente é focado em uma tarefa específica para otimizar uso de tokens e dar suporte preciso ao código.

---

## Categorias

### 01-laravel

Especialistas Laravel 12 para o backend WhylLima. Todos os agentes seguem o padrão Service-Repository com modelos UUID.

**Stack:** PHP 8.3 | Laravel 12 | JWT/Sanctum | Horizon | Reverb | PHPUnit

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

| Agente | Propósito | Linhas | ~Tokens |
|--------|-----------|--------|---------|
| `full-stack-specialist` | Feature completa do zero | 197 | ~500 |
| `model-builder` | Só estrutura de dados | 63 | ~160 |
| `api-layer-builder` | API para model existente | 182 | ~450 |
| `endpoint-builder` | Só rotas | 95 | ~240 |
| `test-builder` | Testes PHPUnit | 152 | ~380 |
| `job-builder` | Jobs em fila | 55 | ~140 |
| `resource-builder` | Transformação JSON | 57 | ~140 |
| `seeder-builder` | Dados de teste/dev | 68 | ~170 |
| `event-builder` | Eventos e broadcasting | 79 | ~200 |
| `jwt-auth-builder` | Autenticação JWT completa | 587 | ~1.5k |
| `acl-builder` | ACL Modular (3 níveis) | 723 | ~1.8k |
| `audit-builder` | Auditoria de Models | 421 | ~1k |

---

## Agentes – Qualidade e Fixers (auditar e corrigir)

| Agente | Propósito | Linhas | ~Tokens |
|--------|-----------|--------|---------|
| `quality-checker` | Audita código e gera prompts | 131 | ~330 |
| `controller-fixer` | Remove lógica do Controller | 88 | ~220 |
| `service-fixer` | Garante Repository e Resources | 75 | ~190 |
| `repository-fixer` | Converte `DB` para Model | 61 | ~150 |
| `model-fixer` | Traits, UUID e relacionamentos | 57 | ~140 |
| `formrequest-fixer` | Validação create/update | 50 | ~125 |
| `resource-fixer` | Campos explícitos, datas ISO | 54 | ~135 |

---

## Resumo de Tokens

| Categoria | Total Linhas | ~Tokens |
|-----------|--------------|---------|
| Builders | 2.679 | ~6.7k |
| Fixers | 516 | ~1.3k |
| **Total** | **3.195** | **~8k** |

> **Nota:** Estimativa baseada em ~2.5 tokens/linha. Tokens reais podem variar.

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
- Autenticação JWT → `jwt-auth-builder`
- Sistema de permissões modular → `acl-builder`
- Auditoria de alterações → `audit-builder`

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

# Configurar autenticação JWT
@whyll-agents:jwt-auth-builder Setup JWT authentication

# Configurar ACL modular (User → Role → Module → Permission)
@whyll-agents:acl-builder Setup modular ACL system

# Configurar auditoria de models
@whyll-agents:audit-builder Setup auditing system

# Verificar configuração de audit nos models
@whyll-agents:audit-builder Check audit configuration in models

# Auditar e obter prompts de correção
@whyll-agents:quality-checker Audit Entity feature

# Corrigir um componente (usar prompt gerado pelo quality-checker)
@whyll-agents:controller-fixer Fix EntityController
File: app/Http/Controllers/EntityController.php
Issues: try-catch blocks, Log:: calls, direct queries
```

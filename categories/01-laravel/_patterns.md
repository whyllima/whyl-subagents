# WhylLima Patterns Reference

Shared patterns for all agents. DO NOT read this file - patterns are embedded in each agent.

## Architecture

```
Controller (no logic) → Service (all logic) → Repository (Model queries) → Model
                                                                         ↓
                                    Response via Resource ←───────────────
```

## Rules Summary

| Component | Rule |
|-----------|------|
| Controller | NO logic, delegates to Service, returns JsonResource |
| Service | ALL logic, uses Repository, returns Resource/ErrorResource |
| Repository | Model::query() only, NEVER DB::, throws on error |
| Model | 5 traits, UUID PK, _uuid FKs |
| FormRequest | Handles POST + PUT/PATCH, Rule::unique()->ignore() |
| Resource | Explicit fields, toISOString(), whenLoaded() |

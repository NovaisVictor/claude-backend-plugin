# plugin-backend

Plugin para projetos backend com **Bun + Elysia + Drizzle + Clean Architecture + SOLID**.

## Instalação

```bash
claude plugin install github:NovaisVictor/claude-backend-plugin
```

## O que inclui

### Skills (conhecimento automático)

| Skill              | Ativada quando                                                             |
| ------------------ | -------------------------------------------------------------------------- |
| `architecture`     | Decisões arquiteturais, validação de estrutura, dependências entre camadas |
| `project-setup`    | Criar projeto do zero, referência de deps e configs                        |
| `drizzle-patterns` | Schemas, migrations, column naming, relations, indexes                     |
| `elysia-patterns`  | Plugin pattern, route handlers, error mapping, OpenAPI                     |
| `testing-patterns` | Unit tests com in-memory repos, e2e tests com app.handle()                 |
| `zod-validation`   | Request schemas, z.infer, validação inline                                 |

Skills são lidas automaticamente pelo Claude quando o contexto é relevante. Não é necessário invocá-las.

### Commands (invocação explícita)

| Command                                | Uso                         | O que gera                                          |
| -------------------------------------- | --------------------------- | --------------------------------------------------- |
| `/drizzle-schema Product`              | Criar schema Drizzle        | `src/database/schema/products.ts` + atualiza barrel |
| `/repository Product`                  | Criar camada de repositório | Interface + Drizzle impl + InMemory impl            |
| `/use-case CreateProduct`              | Criar use case completo     | Use case + factory + domain error + unit test       |
| `/route POST /products create-product` | Criar rota Elysia           | Route handler + routes barrel                       |
| `/unit-test CreateProduct`             | Criar unit test             | Spec com sut, beforeEach, in-memory repos           |
| `/e2e-test POST /products`             | Criar e2e test              | Spec com app.handle(new Request())                  |

### Agents (subagents especializados)

| Agent        | Modo             | Função                                 |
| ------------ | ---------------- | -------------------------------------- |
| `reviewer`   | plan (read-only) | Code review de aderência à arquitetura |
| `scaffolder` | default          | Gera feature completa de ponta a ponta |

## Workflow de scaffolding

Os commands seguem uma ordem encadeada:

```
/drizzle-schema {Entity}
  → /repository {Entity}
    → /use-case {Action}{Entity}
      → /route {METHOD} /{path} {use-case}
        → /e2e-test {METHOD} /{path}
```

Ou use o agent `scaffolder` para gerar tudo de uma vez:

> "Cria a feature completa de Product com CRUD"

## Arquitetura

```
HTTP (Elysia) → Factory → Use Case → Repository (interface) → Database (Drizzle)
```

- Use cases dependem apenas de interfaces de repository
- Factories são o único ponto de wiring (instanciam Drizzle repos)
- Testes unitários usam InMemory repos
- Testes e2e usam banco real via app.handle()

## Requisitos

- Bun runtime
- PostgreSQL 17
- Elysia + Drizzle ORM + Zod v4 + Vitest + Biome

---
description: "Padrões de Clean Architecture + SOLID para APIs com Bun, Elysia e Drizzle. Use quando precisar tomar decisões arquiteturais, validar estrutura de código, ou entender dependências entre camadas."
---

# Arquitetura — Clean Architecture + SOLID

## Camadas

```
HTTP (Elysia) → Factory (wiring) → Use Case (domínio) → Repository (interface) → Database (Drizzle)
```

**Regra de dependência:** cada camada só importa da camada abaixo. Use case nunca importa HTTP. Interface de repository nunca importa Drizzle.

## SOLID aplicado

- **S** — Uma classe, uma responsabilidade. `CreateProductUseCase` só cria produtos.
- **O** — Novo comportamento = nova classe. Novo backend de repositório implementa a interface existente.
- **L** — `InMemory*Repository` e `Drizzle*Repository` (e quaisquer outras data sources como API externa, cache, DWH) são intercambiáveis.
- **I** — Cada entidade tem sua interface de repository com apenas os métodos necessários.
- **D** — Use cases declaram dependências como interfaces no construtor. Factory injeta a implementação.

**Multi data source:** quando uma entidade vive em mais de uma fonte (Drizzle + DWH, por exemplo), criar uma implementação por fonte (`Drizzle{Entity}sRepository`, `Dwh{Entity}sRepository`). A interface é a mesma; o use case escolhe via factory ou injeção.

## Estrutura de pastas

```
src/
  index.ts                                  # Entrypoint Elysia
  env.ts                                    # Variáveis de ambiente com Zod
  database/
    client.ts                               # Singleton Drizzle
    schema/
      index.ts                              # Barrel com todas as tables + relations
      {entities}.ts                         # Uma table por arquivo
    migrations/
  repositories/
    {entities}-repository.ts                # Interface + tipo da entidade
    drizzle/
      drizzle-{entities}-repository.ts      # Implementação Drizzle
    in-memory/
      in-memory-{entities}-repository.ts    # Test double
    # outras data sources conforme necessário (api externa, cache, dwh)
  use-cases/
    {action}.ts                             # Classe do use case
    {action}.spec.ts                        # Unit test
    errors/
      resource-not-found-error.ts
      {entity}-already-exists-error.ts
    factories/
      make-{action}-use-case.ts             # Factory — único ponto de wiring
  http/
    controllers/
      {domain}/
        routes.ts                           # Barrel Elysia com prefix
        {action}.ts                         # Route handler
        {action}.spec.ts                    # E2E test
    plugins/
    middlewares/
```

## Imports proibidos

- Use case importando `db` → usar método do repository
- Use case importando `Drizzle*Repository` → usar factory
- Use case importando `Elysia` → HTTP não pertence ao domínio
- Route importando request schema do use case → rotas definem seus próprios Zod schemas
- Route importando `db` → usar use case
- In-memory repo importando `db` ou `drizzle-orm` → deve ser puro

## Fluxo de dependência

```
src/index.ts → {domain}Routes
  routes.ts → {action}Route
    {action}.ts (route) → make{Action}UseCase (factory) + {DomainError}
      factory → Drizzle{Entity}sRepository + {Action}UseCase
        use case → {Entity}sRepository (interface) + z (zod) + {DomainError}
```

## Naming

| Item | Convenção | Exemplo |
|------|-----------|---------|
| Arquivos | kebab-case | `create-product.ts` |
| Classes | PascalCase | `CreateProductUseCase` |
| Interface repository | `{Entity}sRepository` | `ProductsRepository` |
| Drizzle impl | `Drizzle{Entity}sRepository` | `DrizzleProductsRepository` |
| In-memory impl | `InMemory{Entity}sRepository` | `InMemoryProductsRepository` |
| Outros data sources | `{Source}{Entity}sRepository` | `DwhGamesCatalogRepository` |
| Factory | `make{PascalCase}UseCase` | `makeCreateProductUseCase` |
| Request schema | `{camelCase}RequestSchema` | `createProductRequestSchema` |
| Route export | `{camelCase}Route` | `createProductRoute` |
| Routes barrel | `{domain}Routes` | `productsRoutes` |
| SUT (testes) | `sut` | `let sut: CreateProductUseCase` |
| DB tables | snake_case string | `'product_items'` |
| DB columns | snake_case string | `'user_id'` |
| TS properties | camelCase | `userId` |
| Path alias | `@/*` → `./src/*` | `@/repositories/...` |

---
description: Gerar rota Elysia e atualizar barrel de rotas do domínio.
---

Formato de $ARGUMENTS: `{METHOD} /{path} {use-case-kebab-name}`
Exemplos: `POST /products create-product`, `GET /products/:id get-product-by-id`

## Nomes derivados

- HTTP method: primeiro token
- Path: segundo token
- Use case name: terceiro token (kebab-case)
- Entity: primeiro segmento do path (kebab) — pasta em `src/http/routes/{entity}/`
- Action variable: camelCase do use case + `Route`
- Factory function: `make` + PascalCase do use case + `UseCase`
- Arquivo de rota: `{use-case-kebab}.route.ts`
- Arquivo de E2E: `{use-case-kebab}.route.spec.ts`

## Pré-requisitos

1. Ler `src/use-cases/{use-case-kebab}.ts` para entender tipos e domain errors
2. Ler `src/use-cases/errors/` para identificar erros do use case

## Gerar route handler

`src/http/routes/{entity}/{use-case-kebab}.route.ts`:

```typescript
import Elysia from 'elysia'
import z from 'zod'
import { make{UseCasePascal}UseCase } from '@/use-cases/factories/make-{use-case-kebab}-use-case'

export const {actionVariable} = new Elysia().{method}(
  '{path}',
  async ({ body, params, status }) => {
    const useCase = make{UseCasePascal}UseCase()

    const result = await useCase.execute({
      // TODO: mapear input validado
    })

    // TODO: status e response adequados
  },
  {
    detail: {
      summary: 'TODO: descrição da rota',
      tags: ['{Entity}s'],
    },
    // TODO: body, params, response schemas
  },
)
```

**Não capturar domain errors aqui.** O `errorHandlerPlugin` central mapeia (ver skill `error-handling`). Garanta que os erros lançados pelo use case estão registrados em `src/http/plugins/error-handler.ts`.

## Criar/atualizar barrel

`src/http/routes/{entity}/index.ts`:

Se não existir, criar:

```typescript
import Elysia from 'elysia'
import { {actionVariable} } from './{use-case-kebab}.route'

export const {entity}Routes = new Elysia()
  .use({actionVariable})
```

Se já existir, adicionar import e `.use()`.

> Convenção: paths completos no route handler (`'/products'`, `'/products/:id'`). O barrel não usa `prefix` — rotas declaram seu próprio path.

## Próximos passos

Informar ao usuário:

1. Preencher Zod schemas de body, params e response
2. Mapear input para o `execute()` do use case
3. Verificar se cada `DomainError` lançado está mapeado em `error-handler.ts`
4. Registrar `{domain}Routes` no app principal se necessário
5. `/e2e-test {METHOD} /{path}` para gerar teste E2E (co-located em `{action}.route.spec.ts`)

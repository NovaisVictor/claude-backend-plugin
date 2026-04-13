---
description: Gerar rota Elysia e atualizar barrel de rotas do domínio.
---

Formato de $ARGUMENTS: `{METHOD} /{path} {use-case-kebab-name}`
Exemplos: `POST /products create-product`, `GET /products/:id get-product-by-id`

## Nomes derivados

- HTTP method: primeiro token
- Path: segundo token
- Use case name: terceiro token (kebab-case)
- Domain: primeiro segmento do path
- Action variable: camelCase do use case + `Route`
- Factory function: `make` + PascalCase do use case + `UseCase`

## Pré-requisitos

1. Ler `src/use-cases/{use-case-kebab}.ts` para entender tipos e domain errors
2. Ler `src/use-cases/errors/` para identificar erros do use case

## Gerar route handler

`src/http/controllers/{domain}/{use-case-kebab}.ts`:

```typescript
import Elysia from 'elysia'
import z from 'zod'
import { make{UseCasePascal}UseCase } from '@/use-cases/factories/make-{use-case-kebab}-use-case'
import { ResourceNotFoundError } from '@/use-cases/errors/resource-not-found-error'

export const {actionVariable} = new Elysia().{method}(
  '{path}',
  async ({ body, params, status }) => {
    const useCase = make{UseCasePascal}UseCase()

    try {
      const result = await useCase.execute({
        // TODO: mapear input validado
      })

      // TODO: status e response adequados
    } catch (err) {
      if (err instanceof ResourceNotFoundError) {
        return status(404, { message: err.message })
      }
      throw err
    }
  },
  {
    detail: {
      summary: 'TODO: descrição da rota',
      tags: ['{Domain}'],
    },
    // TODO: body, params, response schemas
  },
)
```

## Criar/atualizar routes barrel

`src/http/controllers/{domain}/routes.ts`:

Se não existir, criar:

```typescript
import Elysia from 'elysia'
import { {actionVariable} } from './{use-case-kebab}'

export const {domain}Routes = new Elysia({ prefix: '/{domain}' })
  .use({actionVariable})
```

Se já existir, adicionar import e `.use()`.

## Próximos passos

Informar ao usuário:

1. Preencher Zod schemas de body, params e response
2. Mapear input para o `execute()` do use case
3. Ajustar error handling para os domain errors
4. Registrar `{domain}Routes` no app principal se necessário
5. `/e2e-test {METHOD} /{path}` para gerar teste E2E

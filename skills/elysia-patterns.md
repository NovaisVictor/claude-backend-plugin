---
description: "Convenções de Elysia: plugin pattern, route handlers, error mapping, OpenAPI, Zod validation inline. Use quando trabalhar com rotas HTTP, controllers, middlewares, ou qualquer código que envolva Elysia."
---

# Elysia — Convenções

## Plugin pattern

Cada rota é uma instância `new Elysia()` independente em `src/http/routes/{entity}/{action}.route.ts`:

```typescript
import Elysia from 'elysia'
import z from 'zod'
import { makeCreateProductUseCase } from '@/use-cases/factories/make-create-product-use-case'

export const createProductRoute = new Elysia().post(
  '/products',
  async ({ body, status }) => {
    const useCase = makeCreateProductUseCase()

    const { product } = await useCase.execute({
      name: body.name,
      userId: body.userId,
    })

    return status(201, { productId: product.id })
  },
  {
    detail: {
      summary: 'Create a new product',
      tags: ['Products'],
    },
    body: z.object({
      name: z.string().min(1),
      userId: z.string().uuid(),
    }),
    response: {
      201: z.object({ productId: z.string().uuid() }),
    },
  },
)
```

Domain errors são tratados pelo `errorHandlerPlugin` central — rotas não usam `try/catch` para esses erros. Ver `error-handling.md`.

## Routes barrel

Cada entidade exporta um barrel em `src/http/routes/{entity}/index.ts`:

```typescript
import Elysia from 'elysia'
import { createProductRoute } from './create-product.route'
import { getProductByIdRoute } from './get-product-by-id.route'

export const productsRoutes = new Elysia()
  .use(createProductRoute)
  .use(getProductByIdRoute)
```

Registrar em `src/index.ts`:

```typescript
app
  .use(errorHandlerPlugin)
  .use(productsRoutes)
```

## Regras

- Cada rota exporta um `new Elysia()` (composável, independente, testável)
- Arquivo de rota termina em `.route.ts` (`create-product.route.ts`)
- Arquivo de E2E test termina em `.route.spec.ts` (`create-product.route.spec.ts`)
- Barrel da entidade em `index.ts` (não `routes.ts`)
- Zod schemas definidos **inline** no objeto de opções da rota, não importados de fora
- `import z from 'zod'` (default import, Zod v4)
- **Não capturar domain errors em rota** — `errorHandlerPlugin` central mapeia (ver `error-handling.md`)
- Routes registradas em `src/index.ts` via `.use({entity}Routes)`
- Route handler chama a **factory**, nunca instancia repos diretamente
- Route handler nunca importa `db`
- Path do route handler é o path completo (`'/products'`, `'/products/:id'`) — sem prefix no barrel

## Error → Status code

Mapeamento centralizado em `src/http/plugins/error-handler.ts` via `statusFor()`. Ver `error-handling.md` para detalhes. Tabela resumida:

| Domain Error | Status |
|---|---|
| `*NotFoundError` | 404 |
| `*AlreadyExistsError` | 409 |
| `*InvalidCredentialsError` | 401 |
| `*UnauthorizedError` / `*ForbiddenError` | 403 |
| Constraint genérica | 400 |
| Zod validation (automático) | 400 |
| Erro desconhecido | 500 |

## OpenAPI

Toda rota deve ter `detail` com `summary` e `tags`. Tags agrupam por domínio (PascalCase plural).

```typescript
{
  detail: {
    summary: 'Descrição curta do que a rota faz',
    tags: ['Products'],
  },
}
```

## CRUD HTTP methods padrão

| Ação | Method | Status sucesso |
|------|--------|----------------|
| Create | POST | 201 |
| List | GET | 200 |
| Get by ID | GET | 200 |
| Update (parcial) | PATCH | 200 |
| Delete | DELETE | 204 |

Update usa **PATCH** — body com todos os campos opcionais. PUT só se a semântica for "substituição completa" (raro).

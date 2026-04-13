---
description: "Convenções de Elysia: plugin pattern, route handlers, error mapping, OpenAPI, Zod validation inline. Use quando trabalhar com rotas HTTP, controllers, middlewares, ou qualquer código que envolva Elysia."
---

# Elysia — Convenções

## Plugin pattern

Cada rota é uma instância `new Elysia()` independente:

```typescript
import Elysia from 'elysia'
import z from 'zod'
import { makeCreateProductUseCase } from '@/use-cases/factories/make-create-product-use-case'
import { ProductAlreadyExistsError } from '@/use-cases/errors/product-already-exists-error'

export const createProductRoute = new Elysia().post(
  '/products',
  async ({ body, status }) => {
    const useCase = makeCreateProductUseCase()

    try {
      const { product } = await useCase.execute({
        name: body.name,
        userId: body.userId,
      })

      return status(201, { productId: product.id })
    } catch (err) {
      if (err instanceof ProductAlreadyExistsError) {
        return status(409, { message: err.message })
      }
      throw err
    }
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

## Routes barrel

```typescript
import Elysia from 'elysia'
import { createProductRoute } from './create-product'
import { getProductRoute } from './get-product'

export const productsRoutes = new Elysia({ prefix: '/products' })
  .use(createProductRoute)
  .use(getProductRoute)
```

## Regras

- Cada rota exporta um `new Elysia()` (composável, independente, testável)
- Zod schemas definidos **inline** no objeto de opções da rota, não importados de fora
- `import z from 'zod'` (default import, Zod v4)
- Sempre re-throw de erros desconhecidos (`throw err`) — nunca engolir falhas
- Domain errors capturados individualmente com `instanceof`
- Routes barrel usa `new Elysia({ prefix: '/{domain}' })`
- Routes registradas em `src/index.ts` via `.use(domainRoutes)`
- Route handler chama a **factory**, nunca instancia repos diretamente
- Route handler nunca importa `db`

## Error → Status code

| Domain Error | Status |
|---|---|
| `ResourceNotFoundError` | 404 |
| `*AlreadyExistsError` | 409 |
| `*InvalidCredentialsError` | 401 |
| `*UnauthorizedError` | 403 |
| Zod validation (automático) | 400 |
| Erro desconhecido (re-thrown) | 500 |

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

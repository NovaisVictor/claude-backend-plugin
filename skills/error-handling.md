---
description: "Padrão de tratamento de erros centralizado com DomainError base + errorHandlerPlugin. Use quando trabalhar com domain errors, mapping de status code, ou onError do Elysia."
---

# Error Handling — DomainError + errorHandlerPlugin

## Princípio

Use cases **lançam** domain errors. Rotas **não** capturam domain errors. Um plugin Elysia central faz o mapeamento `error → status code` em um único lugar.

## DomainError base

Localização: `src/use-cases/errors/domain-error.ts`

```typescript
export abstract class DomainError extends Error {
  abstract readonly code: string

  constructor(message: string) {
    super(message)
    this.name = this.constructor.name
  }
}
```

- `code` em **SCREAMING_SNAKE_CASE** (`PRODUCT_NOT_FOUND`, `TAG_IN_USE`).
- `message` em **inglês**, voltada para o desenvolvedor — não é user-facing.
- Use case **lança** o erro. Nunca retorna `null`/`Result<T>` em violação de domínio.

## Subclasses por entidade

`src/use-cases/errors/{kebab-entity}-not-found-error.ts`:

```typescript
import { DomainError } from './domain-error'

export class ProductNotFoundError extends DomainError {
  readonly code = 'PRODUCT_NOT_FOUND'

  constructor() {
    super('Product not found.')
  }
}
```

Naming:
- `{Entity}NotFoundError`
- `{Entity}AlreadyExistsError`
- `{Entity}{Constraint}Error` para regras específicas (ex: `TagOrganizationMismatchError`, `TagInUseError`)

## errorHandlerPlugin

Localização: `src/http/plugins/error-handler.ts`

```typescript
import Elysia from 'elysia'
import { DomainError } from '@/use-cases/errors/domain-error'
import { ProductNotFoundError } from '@/use-cases/errors/product-not-found-error'
import { ProductAlreadyExistsError } from '@/use-cases/errors/product-already-exists-error'

function statusFor(error: DomainError): number {
  if (error instanceof ProductNotFoundError) return 404
  if (error instanceof ProductAlreadyExistsError) return 409
  return 400
}

export const errorHandlerPlugin = new Elysia({ name: 'error-handler' })
  .onError(({ error, status }) => {
    if (error instanceof DomainError) {
      return status(statusFor(error), {
        code: error.code,
        message: error.message,
      })
    }
  })
```

Registrar **uma vez** em `src/index.ts`:

```typescript
app.use(errorHandlerPlugin).use(productsRoutes)
```

## Regras

- **Toda subclass nova de `DomainError` precisa ser importada e mapeada em `statusFor()`.** Sem isso, vira 400 silencioso (ou 500 se outro tipo).
- Rota: **só `throw err`** quando precisar propagar — sem `try/catch` de domain error.
- Erros desconhecidos (não-DomainError) seguem o fluxo padrão do Elysia (500).

## Status code padrão

| Categoria | Código |
|---|---|
| `*NotFoundError` | 404 |
| `*AlreadyExistsError` | 409 |
| `*InvalidCredentialsError` | 401 |
| `*UnauthorizedError` / `*ForbiddenError` | 403 |
| Constraint genérica | 400 |
| Erro desconhecido | 500 |
| Zod validation (automático) | 400 |

## Exemplo de rota (sem try/catch)

```typescript
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
    detail: { summary: 'Create a new product', tags: ['Products'] },
    body: z.object({ name: z.string().min(1), userId: z.string().uuid() }),
    response: { 201: z.object({ productId: z.string().uuid() }) },
  },
)
```

## Exemplo de teste

```typescript
it('should throw ProductAlreadyExistsError when name conflicts', async () => {
  await sut.execute({ name: 'duplicated', userId: 'u' })

  await expect(sut.execute({ name: 'duplicated', userId: 'u' }))
    .rejects.toBeInstanceOf(ProductAlreadyExistsError)
})
```

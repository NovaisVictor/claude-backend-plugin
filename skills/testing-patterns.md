---
description: "Convenções de testes: unit tests com in-memory repos, e2e tests com app.handle(), padrão sut, beforeEach. Use quando trabalhar com testes, Vitest, assertions, ou qualquer código em arquivos .spec.ts."
---

# Testes — Convenções

## TDD workflow

Test-first com etapas explícitas:

1. **Listar casos** como `it.todo('should X')` cobrindo: happy path, cada domain error, edge cases.
2. **Promover** 1 `it.todo` → `it` com `expect`s que falham.
3. **Rodar** `bun run test {arquivo}` e ver vermelho.
4. **Implementar o mínimo** no use case para passar.
5. **Refactor** quando verde. Loop até cobrir os `it.todo`s restantes.

```typescript
describe('CreateProductUseCase', () => {
  beforeEach(() => {
    productsRepository = new InMemoryProductsRepository()
    sut = new CreateProductUseCase(productsRepository)
  })

  it.todo('should create a product with valid input')
  it.todo('should throw ProductAlreadyExistsError when name conflicts')
  it.todo('should normalize name to lowercase before persisting')
})
```

Promova um por vez, escreva os `expect`s que falham, implemente, refatore.

## Unit tests (use cases)

Localização: `src/use-cases/{action}.spec.ts`

```typescript
import { describe, it, expect, beforeEach } from 'vitest'
import { InMemoryProductsRepository } from '@/repositories/in-memory/in-memory-products-repository'
import { CreateProductUseCase } from './create-product'
import { ProductAlreadyExistsError } from './errors/product-already-exists-error'

let productsRepository: InMemoryProductsRepository
let sut: CreateProductUseCase

describe('CreateProductUseCase', () => {
  beforeEach(() => {
    productsRepository = new InMemoryProductsRepository()
    sut = new CreateProductUseCase(productsRepository)
  })

  it('should be able to create a product', async () => {
    const { product } = await sut.execute({
      name: 'Product 1',
      userId: 'user-id',
    })

    expect(product.id).toEqual(expect.any(String))
    expect(product.name).toBe('Product 1')
  })

  it('should not be able to create duplicate product', async () => {
    await sut.execute({ name: 'Product 1', userId: 'user-id' })

    await expect(() =>
      sut.execute({ name: 'Product 1', userId: 'user-id' }),
    ).rejects.toBeInstanceOf(ProductAlreadyExistsError)
  })
})
```

### Regras unit test

- `sut` é obrigatório como nome da variável do system under test
- `let` no escopo do módulo para repos e sut
- `beforeEach` sempre cria instâncias novas
- Um teste happy path + um teste por domain error + edge cases relevantes
- **Co-located**: `.spec.ts` ao lado do `.ts` do use case
- Apenas in-memory repos — nunca `Drizzle*Repository` ou `db`
- Se usa lógica de tempo: `vi.useFakeTimers()` no `beforeEach`, `vi.useRealTimers()` no `afterEach`
- Imports com `@/`

## E2E tests (controllers)

Localização: `src/http/controllers/{domain}/{action}.spec.ts`

```typescript
import { describe, it, expect } from 'vitest'
import { app } from '@/index'

describe('POST /products (e2e)', () => {
  it('should be able to create a product', async () => {
    const response = await app.handle(
      new Request('http://localhost/products', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          name: 'Product 1',
          userId: 'some-user-id',
        }),
      }),
    )

    expect(response.status).toBe(201)
    const data = await response.json()
    expect(data.productId).toEqual(expect.any(String))
  })

  it('should return 400 when body is invalid', async () => {
    const response = await app.handle(
      new Request('http://localhost/products', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({}),
      }),
    )

    expect(response.status).toBe(400)
  })
})
```

### Regras e2e test

- `app.handle(new Request(...))` — API nativa do Elysia, sem supertest
- Testa com banco real via Drizzle
- Se precisa de dados existentes, seed via `db.insert()` no `beforeAll`
- Testa success path + cada domain error mapeado
- Status esperados: POST=201, GET=200, PUT/PATCH=200, DELETE=204

## Scripts

```bash
bun run test           # unit tests (src/use-cases)
bun run test:watch     # unit tests watch mode
bun run test:e2e       # e2e tests (src/http)
bun run test:e2e:watch # e2e tests watch mode
bun run test:coverage  # cobertura
```

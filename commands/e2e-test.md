Gerar teste E2E para uma rota Elysia existente. Argumentos em $ARGUMENTS: `{METHOD} /{path}`.

Exemplos: `POST /products`, `GET /products/:id`, `DELETE /orders/:id`

## Nomes derivados

- HTTP method: primeiro token
- Path: segundo token
- Domain: primeiro segmento do path

## Pré-requisitos

1. Encontrar o controller em `src/http/controllers/{domain}/` que corresponde ao path e method
2. Ler para entender: Zod schemas, use case chamado, domain errors mapeados
3. Encontrar onde o app Elysia é exportado (`src/index.ts` ou `src/app.ts`)

## Gerar teste

`src/http/controllers/{domain}/{action}.spec.ts`:

```typescript
import { describe, it, expect } from 'vitest'
import { app } from '@/index'

describe('{Action} (e2e)', () => {
  it('should be able to {success scenario}', async () => {
    const response = await app.handle(
      new Request('http://localhost{path}', {
        method: '{METHOD}',
        headers: { 'Content-Type': 'application/json' },
        // body: JSON.stringify({ TODO }),
      }),
    )

    expect(response.status).toBe({expectedStatus})
  })

  it('should return 400 when body is invalid', async () => {
    const response = await app.handle(
      new Request('http://localhost{path}', {
        method: '{METHOD}',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({}),
      }),
    )

    expect(response.status).toBe(400)
  })

  // Um teste por domain error mapeado na rota
})
```

## Status esperados

- POST que cria: 201
- GET: 200
- PUT/PATCH: 200
- DELETE: 204

Para paths com parâmetros (`:id`), substituir por ID real (do seed ou hardcoded).

## Próximos passos

Informar ao usuário:
1. Preencher request body conforme Zod schema da rota
2. Adicionar assertions no response body
3. Adicionar `beforeAll` se necessário para seed de dados
4. `vitest run src/http/controllers/{domain}/{action}.spec.ts`

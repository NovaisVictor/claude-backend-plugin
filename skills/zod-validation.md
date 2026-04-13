---
description: "Convenções de validação com Zod v4: request schemas, z.infer, default import, validação em use cases e rotas. Use quando trabalhar com validação de dados, Zod schemas, tipos inferidos, ou input parsing."
---

# Zod — Convenções

## Import

Sempre default import (Zod v4):

```typescript
import z from 'zod'
```

## Request schema no use case

Exportar o schema com nome `{camelCaseAction}RequestSchema`. Tipo derivado via `z.infer`:

```typescript
export const createProductRequestSchema = z.object({
  name: z.string().min(1),
  userId: z.string().uuid(),
})

type CreateProductUseCaseRequest = z.infer<typeof createProductRequestSchema>
```

Chamar `parse()` no topo do `execute()`:

```typescript
async execute(raw: CreateProductUseCaseRequest): Promise<Response> {
  const { name, userId } = createProductRequestSchema.parse(raw)
  // ...
}
```

## Validação inline nas rotas

Schemas definidos inline no objeto de opções da rota Elysia, não importados do use case:

```typescript
{
  body: z.object({
    name: z.string().min(1),
    userId: z.string().uuid(),
  }),
  params: z.object({
    id: z.string().uuid(),
  }),
  response: {
    201: z.object({ productId: z.string().uuid() }),
    200: z.object({ product: productResponseSchema }),
  },
}
```

## Validação de env

```typescript
import z from 'zod'

const envSchema = z.object({
  DATABASE_URL: z.string().min(1),
  PORT: z.coerce.number().default(3333),
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
})

export const env = envSchema.parse(process.env)
```

## Regras

- Nunca escrever tipos manualmente quando `z.infer` resolve
- Response interfaces no use case são `interface` plain (sem Zod) — só wrappam entity types
- Schemas de rota e de use case são independentes (podem ter campos diferentes)
- Usar `.min(1)` em strings obrigatórias para evitar strings vazias
- Usar `.uuid()` em campos de ID
- Usar `.coerce` para conversão de tipos de ambiente

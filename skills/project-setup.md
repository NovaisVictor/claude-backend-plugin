---
description: "Setup de novo projeto backend com Bun + Elysia + Drizzle + Vitest + Biome. Use quando o usuário pedir para criar um projeto do zero, configurar um novo repositório, ou precisar de referência sobre as dependências e configurações padrão."
---

# Project Setup — Bun + Elysia + Drizzle

## Stack

| Camada | Tecnologia |
|--------|-----------|
| Runtime | Bun |
| Web framework | Elysia |
| Database | PostgreSQL 17 |
| ORM | Drizzle ORM |
| Validação | Zod v4 |
| Linter/Formatter | Biome |
| Testes | Vitest |

## Dependências

```bash
# Produção
bun add elysia @elysiajs/openapi drizzle-orm pg zod

# Dev
bun add -d @biomejs/biome bun-types drizzle-kit vitest vite-tsconfig-paths @vitest/coverage-v8
```

## Scripts padrão

```json
{
  "dev": "bun run --watch src/index.ts",
  "build": "bun build src/index.ts --outdir dist --target bun",
  "test": "vitest run --reporter=verbose src/use-cases",
  "test:watch": "vitest --reporter=verbose src/use-cases",
  "test:e2e": "vitest run --reporter=verbose src/http",
  "test:e2e:watch": "vitest --reporter=verbose src/http",
  "test:coverage": "vitest run --coverage",
  "db:generate": "bun --env-file .env drizzle-kit generate",
  "db:migrate": "bun --env-file .env drizzle-kit migrate",
  "db:studio": "bun --env-file .env drizzle-kit studio",
  "lint": "bunx biome check .",
  "lint:fix": "bunx biome check --write ."
}
```

## Configurações

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2021",
    "module": "ES2022",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "types": ["bun-types"],
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["src/**/*", "*.config.ts"]
}
```

### drizzle.config.ts

```typescript
import { defineConfig } from 'drizzle-kit'

export default defineConfig({
  schema: './src/database/schema/**',
  out: './src/database/migrations',
  dialect: 'postgresql',
  casing: 'snake_case',
  dbCredentials: { url: process.env.DATABASE_URL! },
})
```

### vitest.config.ts

```typescript
import { defineConfig } from 'vitest/config'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [tsconfigPaths()],
  test: { dir: 'src' },
})
```

### docker-compose.yml

```yaml
services:
  postgres:
    image: postgres:17
    environment:
      POSTGRES_DB: myapi
      POSTGRES_USER: docker
      POSTGRES_PASSWORD: docker
    ports:
      - "5432:5432"
```

## Arquivos iniciais obrigatórios

### src/env.ts

```typescript
import z from 'zod'

const envSchema = z.object({
  DATABASE_URL: z.string().min(1),
  PORT: z.coerce.number().default(3333),
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
})

export const env = envSchema.parse(process.env)
```

### src/database/client.ts

```typescript
import { drizzle } from 'drizzle-orm/node-postgres'
import { env } from '@/env'
import { schema } from './schema'

export const db = drizzle(env.DATABASE_URL, { schema, casing: 'snake_case' })
```

### src/database/schema/index.ts

```typescript
export const schema = {
  // Tables
  // Relations
}
```

### src/index.ts

```typescript
import openapi from '@elysiajs/openapi'
import { Elysia } from 'elysia'
import { env } from '@/env'

export const app = new Elysia()
  .use(openapi())
  .listen(env.PORT)

console.log(`Server running at http://localhost:${env.PORT}`)
```

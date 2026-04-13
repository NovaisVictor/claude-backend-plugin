# claude-backend-plugin

Plugin para projetos backend com **Bun + Elysia + Drizzle + Clean Architecture + SOLID**.

## Instalação

### 1. Adicionar o marketplace (apenas na primeira vez)

```bash
claude plugin marketplace add NovaisVictor/claude-marketplace
```

### 2. Instalar o plugin

```bash
claude plugin install claude-backend-plugin@novais-plugins
```

---

## Setup de projeto do zero

### 1. Criar projeto

```bash
mkdir my-api && cd my-api
bun init -y
```

### 2. Instalar dependências

```bash
# Produção
bun add elysia @elysiajs/openapi drizzle-orm pg zod

# Dev
bun add -d @biomejs/biome bun-types drizzle-kit vitest vite-tsconfig-paths @vitest/coverage-v8
```

### 3. Configurar arquivos

**tsconfig.json**

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

**biome.json**

```json
{
  "$schema": "https://biomejs.dev/schemas/2.2.4/schema.json",
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 80,
    "lineEnding": "lf"
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "suspicious": { "noAssignInExpressions": "off" }
    }
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "semicolons": "asNeeded",
      "indentWidth": 2
    }
  },
  "assist": { "actions": { "source": { "organizeImports": "on" } } }
}
```

**drizzle.config.ts**

```typescript
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/database/schema/**",
  out: "./src/database/migrations",
  dialect: "postgresql",
  casing: "snake_case",
  dbCredentials: { url: process.env.DATABASE_URL! },
});
```

**vitest.config.ts**

```typescript
import { defineConfig } from "vitest/config";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  plugins: [tsconfigPaths()],
  test: { dir: "src" },
});
```

**docker-compose.yml**

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

**.env**

```env
DATABASE_URL=postgresql://docker:docker@localhost:5432/myapi
```

### 4. Adicionar scripts ao package.json

```json
{
  "scripts": {
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
}
```

### 5. Criar estrutura de pastas e arquivos iniciais

```bash
mkdir -p src/database/schema src/repositories src/use-cases/errors src/use-cases/factories src/http/controllers src/http/plugins src/http/middlewares
```

**src/env.ts**

```typescript
import z from "zod";

const envSchema = z.object({
  DATABASE_URL: z.string().min(1),
  PORT: z.coerce.number().default(3333),
  NODE_ENV: z
    .enum(["development", "test", "production"])
    .default("development"),
});

export const env = envSchema.parse(process.env);
```

**src/database/client.ts**

```typescript
import { drizzle } from "drizzle-orm/node-postgres";
import { env } from "@/env";
import { schema } from "./schema";

export const db = drizzle(env.DATABASE_URL, { schema, casing: "snake_case" });
```

**src/database/schema/index.ts**

```typescript
export const schema = {
  // Tables
  // Relations
};
```

**src/index.ts**

```typescript
import openapi from "@elysiajs/openapi";
import { Elysia } from "elysia";
import { env } from "@/env";

export const app = new Elysia().use(openapi()).listen(env.PORT);

console.log(`Server running at http://localhost:${env.PORT}`);
```

### 6. Iniciar

```bash
docker compose up -d
bun run dev
```

---

## O que o plugin inclui

### Skills (conhecimento automático)

| Skill              | Ativada quando                                             |
| ------------------ | ---------------------------------------------------------- |
| `architecture`     | Decisões arquiteturais, validação de estrutura             |
| `project-setup`    | Criar projeto do zero, referência de deps e configs        |
| `drizzle-patterns` | Schemas, migrations, column naming, relations              |
| `elysia-patterns`  | Plugin pattern, route handlers, error mapping, OpenAPI     |
| `testing-patterns` | Unit tests com in-memory repos, e2e tests com app.handle() |
| `zod-validation`   | Request schemas, z.infer, validação inline                 |

### Commands

| Command                                | Uso                                           |
| -------------------------------------- | --------------------------------------------- |
| `/drizzle-schema Product`              | Criar schema Drizzle                          |
| `/repository Product`                  | Interface + Drizzle impl + InMemory impl      |
| `/use-case CreateProduct`              | Use case + factory + domain error + unit test |
| `/route POST /products create-product` | Route handler + routes barrel                 |
| `/unit-test CreateProduct`             | Unit test para use case existente             |
| `/e2e-test POST /products`             | E2E test para rota existente                  |

### Agents

| Agent        | Função                                             |
| ------------ | -------------------------------------------------- |
| `reviewer`   | Code review de aderência à arquitetura (read-only) |
| `scaffolder` | Gera feature completa de ponta a ponta             |

## Workflow de scaffolding

```
/drizzle-schema {Entity} → /repository {Entity} → /use-case {Action}{Entity} → /route {METHOD} /{path} {use-case} → /e2e-test {METHOD} /{path}
```

Ou peça ao agent scaffolder: _"Cria a feature completa de Product com CRUD"_

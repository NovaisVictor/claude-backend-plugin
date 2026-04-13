---
description: Gerar schema Drizzle para a entidade em $ARGUMENTS (PascalCase singular, e.g. `Product`).
---

## Nomes derivados

- Entity: PascalCase singular → `Product`
- Table variable: camelCase plural → `products`
- Table name string: snake_case plural → `'products'`
- Arquivo: `src/database/schema/{entities}.ts` (kebab-case plural)

## Gerar arquivo

Criar `src/database/schema/{entities}.ts`:

```typescript
import { randomUUIDv7 } from 'bun'
import { relations } from 'drizzle-orm'
import { index, pgTable, text, timestamp, uuid } from 'drizzle-orm/pg-core'

export const {entities} = pgTable(
  '{entities_snake}',
  {
    id: uuid('id')
      .primaryKey()
      .$defaultFn(() => randomUUIDv7()),
    // TODO: adicionar colunas
    createdAt: timestamp('created_at').defaultNow().notNull(),
    updatedAt: timestamp('updated_at')
      .defaultNow()
      .$onUpdate(() => new Date())
      .notNull(),
  },
  (table) => [
    // TODO: adicionar indexes
  ],
)

export const {entities}Relations = relations({entities}, ({ one, many }) => ({
  // TODO: definir relações
}))
```

## Atualizar barrel

Em `src/database/schema/index.ts`:

- Adicionar import de `{entities}` e `{entities}Relations`
- Adicionar ao objeto `schema` nas seções Tables e Relations

## Próximos passos

Informar ao usuário:

1. Adicionar colunas e relações ao schema
2. `bun run db:generate`
3. `bun run db:migrate`
4. `/repository {Entity}` para gerar a camada de repositório

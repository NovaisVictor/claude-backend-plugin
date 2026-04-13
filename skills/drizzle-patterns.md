---
description: "Convenções de Drizzle ORM: schemas, migrations, column naming, relations, indexes. Use quando trabalhar com banco de dados, schemas, migrations, ou qualquer código que envolva drizzle-orm ou drizzle-kit."
---

# Drizzle ORM — Convenções

## Schema

```typescript
import { randomUUIDv7 } from 'bun'
import { relations } from 'drizzle-orm'
import { index, pgTable, text, timestamp, uuid } from 'drizzle-orm/pg-core'

export const products = pgTable(
  'products',
  {
    id: uuid('id')
      .primaryKey()
      .$defaultFn(() => randomUUIDv7()),
    name: text('name').notNull(),
    userId: uuid('user_id')
      .notNull()
      .references(() => users.id, { onDelete: 'cascade' }),
    createdAt: timestamp('created_at').defaultNow().notNull(),
    updatedAt: timestamp('updated_at')
      .defaultNow()
      .$onUpdate(() => new Date())
      .notNull(),
  },
  (table) => [
    index('products_user_id_idx').on(table.userId),
  ],
)

export const productsRelations = relations(products, ({ one }) => ({
  user: one(users, { fields: [products.userId], references: [users.id] }),
}))
```

## Regras

- Primary key sempre `randomUUIDv7()` de `'bun'` (UUIDv7 é time-ordered, melhor para indexes)
- Column names sempre com string explícita em snake_case: `uuid('id')`, `text('name')`, `timestamp('created_at')`
- Properties TypeScript em camelCase — o Drizzle client com `casing: 'snake_case'` faz o mapping
- Toda table tem `id`, `createdAt`, `updatedAt`
- Foreign keys com `{ onDelete: 'cascade' }` por padrão
- Indexes em colunas de foreign key e colunas usadas em WHERE
- Relations em export separado: `{table}Relations`
- Importar apenas o que usar de `'drizzle-orm/pg-core'`
- Não importar `relations` se a entidade não tem relações

## Schema barrel (src/database/schema/index.ts)

```typescript
import { products, productsRelations } from './products'

export const schema = {
  // Tables
  products,
  // Relations
  productsRelations,
}
```

Ao criar nova entidade, adicionar import e entries no objeto `schema`.

## Tipo da entidade

Derivar sempre de `InferSelectModel`:

```typescript
import type { InferSelectModel } from 'drizzle-orm'
import type { products } from '@/database/schema'

export type Product = InferSelectModel<typeof products>
```

Nunca definir o tipo manualmente.

## Migrations

```bash
bun run db:generate   # gera SQL
bun run db:migrate    # aplica
bun run db:studio     # interface visual
```

Gerar a camada de repositório para a entidade em $ARGUMENTS (PascalCase singular, e.g. `Product`).

Cria 3 arquivos: interface, implementação Drizzle, implementação in-memory.

## Nomes derivados

- Entity type: `{Entity}` (PascalCase singular)
- Interface: `{Entity}sRepository`
- Table variable: camelCase plural → `{entities}`
- Arquivos:
  - `src/repositories/{entities}-repository.ts`
  - `src/repositories/drizzle/drizzle-{entities}-repository.ts`
  - `src/repositories/in-memory/in-memory-{entities}-repository.ts`

## Pré-requisito

Ler `src/database/schema/{entities}.ts` para entender colunas e tipos. Se não existir, avisar para rodar `/drizzle-schema {Entity}` primeiro.

## Gerar interface

`src/repositories/{entities}-repository.ts`:

```typescript
import type { InferSelectModel } from 'drizzle-orm'
import type { {entities} } from '@/database/schema'

export type {Entity} = InferSelectModel<typeof {entities}>

export interface {Entity}sRepository {
  create(data: Omit<{Entity}, 'id' | 'createdAt' | 'updatedAt'>): Promise<{Entity}>
  findById(id: string): Promise<{Entity} | null>
  // TODO: adicionar métodos específicos do domínio
}
```

## Gerar implementação Drizzle

`src/repositories/drizzle/drizzle-{entities}-repository.ts`:

```typescript
import { eq } from 'drizzle-orm'
import { db } from '@/database/client'
import { {entities} } from '@/database/schema'
import type { {Entity}sRepository, {Entity} } from '../{entities}-repository'

export class Drizzle{Entity}sRepository implements {Entity}sRepository {
  async create(data: Omit<{Entity}, 'id' | 'createdAt' | 'updatedAt'>) {
    const [{entity}] = await db.insert({entities}).values(data).returning()
    return {entity}
  }

  async findById(id: string) {
    const {entity} = await db.query.{entities}.findFirst({
      where: eq({entities}.id, id),
    })
    return {entity} ?? null
  }
}
```

## Gerar implementação in-memory

`src/repositories/in-memory/in-memory-{entities}-repository.ts`:

```typescript
import { randomUUIDv7 } from 'bun'
import type { {Entity}, {Entity}sRepository } from '../{entities}-repository'

export class InMemory{Entity}sRepository implements {Entity}sRepository {
  public items: {Entity}[] = []

  async create(data: Omit<{Entity}, 'id' | 'createdAt' | 'updatedAt'>) {
    const {entity}: {Entity} = {
      id: randomUUIDv7(),
      ...data,
      createdAt: new Date(),
      updatedAt: new Date(),
    }
    this.items.push({entity})
    return {entity}
  }

  async findById(id: string) {
    return this.items.find((item) => item.id === id) ?? null
  }
}
```

## Próximos passos

Informar ao usuário:
1. Adicionar métodos específicos do domínio na interface e implementar nos dois repos
2. `/use-case {Action}{Entity}` para gerar um use case

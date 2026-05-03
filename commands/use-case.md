---
description: Gerar use case completo para o nome em $ARGUMENTS (PascalCase, e.g. `CreateProduct`).
---

Cria 4 arquivos: use case, factory, domain error, unit test.

## Nomes derivados

- Classe: `{Argument}UseCase`
- Entity: extraída do nome da ação (`CreateProduct` → `Product`)
- Repository interface: `{Entity}sRepository`
- Arquivos:
  - `src/use-cases/{kebab-action}.ts`
  - `src/use-cases/factories/make-{kebab-action}-use-case.ts`
  - `src/use-cases/errors/{kebab-entity}-already-exists-error.ts`
  - `src/use-cases/{kebab-action}.spec.ts`

## Pré-requisito

Ler `src/repositories/{entities}-repository.ts` para entender métodos e tipo. Se não existir, avisar para rodar `/repository {Entity}` primeiro.

## Gerar use case

`src/use-cases/{kebab-action}.ts`:

```typescript
import z from 'zod'
import type { {Entity} } from '@/repositories/{entities}-repository'
import type { {Entity}sRepository } from '@/repositories/{entities}-repository'

export const {camelCaseAction}RequestSchema = z.object({
  // TODO: definir campos de input
})

type {Argument}UseCaseRequest = z.infer<typeof {camelCaseAction}RequestSchema>

interface {Argument}UseCaseResponse {
  {entity}: {Entity}
}

export class {Argument}UseCase {
  constructor(private {entity}sRepository: {Entity}sRepository) {}

  async execute(raw: {Argument}UseCaseRequest): Promise<{Argument}UseCaseResponse> {
    const { /* TODO */ } = {camelCaseAction}RequestSchema.parse(raw)

    // TODO: implementar lógica de negócio
    // 1. Verificar regras de negócio
    // 2. Lançar domain error se violada
    // 3. Persistir via repository
    // 4. Retornar response
    throw new Error('Not implemented')
  }
}
```

## Gerar domain error

`src/use-cases/errors/{kebab-entity}-already-exists-error.ts`:

```typescript
import { DomainError } from './domain-error'

export class {Entity}AlreadyExistsError extends DomainError {
  readonly code = '{ENTITY}_ALREADY_EXISTS'

  constructor() {
    super('{Entity} already exists.')
  }
}
```

Criar `domain-error.ts` (base abstrata) se não existir — ver skill `error-handling`.

**Após criar o erro**, mapeá-lo em `src/http/plugins/error-handler.ts` via `statusFor()` (409 para `*AlreadyExists`, 404 para `*NotFound`).

## Gerar factory

`src/use-cases/factories/make-{kebab-action}-use-case.ts`:

```typescript
import { Drizzle{Entity}sRepository } from '@/repositories/drizzle/drizzle-{entities}-repository'
import { {Argument}UseCase } from '../{kebab-action}'

export function make{Argument}UseCase() {
  const {entity}sRepository = new Drizzle{Entity}sRepository()
  return new {Argument}UseCase({entity}sRepository)
}
```

## Gerar unit test (TDD com it.todo)

`src/use-cases/{kebab-action}.spec.ts`:

```typescript
import { describe, it, expect, beforeEach } from 'vitest'
import { InMemory{Entity}sRepository } from '@/repositories/in-memory/in-memory-{entities}-repository'
import { {Argument}UseCase } from './{kebab-action}'
import { {Entity}AlreadyExistsError } from './errors/{kebab-entity}-already-exists-error'

let {entity}sRepository: InMemory{Entity}sRepository
let sut: {Argument}UseCase

describe('{Argument} Use Case', () => {
  beforeEach(() => {
    {entity}sRepository = new InMemory{Entity}sRepository()
    sut = new {Argument}UseCase({entity}sRepository)
  })

  it.todo('should be able to {happy path}')
  it.todo('should throw {Entity}AlreadyExistsError when {scenario}')
  // TODO: adicionar it.todo para cada edge case relevante
})
```

Lista todos os casos primeiro como `it.todo`. Promova um por vez para `it`, escreva os `expect`s que falham, implemente o use case até passar, refatore. Loop.

## Próximos passos

Informar ao usuário:

1. Implementar lógica de negócio no use case
2. Preencher campos do request schema
3. Completar cenários de teste
4. `/route {METHOD} /{entities} {kebab-action}` para gerar a rota HTTP

Gerar unit test para um use case existente. Nome do use case em $ARGUMENTS (PascalCase, e.g. `CreateProduct`).

## Nomes derivados

- Arquivo: `src/use-cases/{kebab-action}.spec.ts`

## Pré-requisitos

1. Ler `src/use-cases/{kebab-action}.ts` para descobrir:
   - Dependências do construtor (repositories)
   - Campos do request schema
   - Tipo de response
   - Domain errors lançados
2. Ler cada domain error em `src/use-cases/errors/`
3. Ler cada in-memory repo em `src/repositories/in-memory/`

## Gerar teste

`src/use-cases/{kebab-action}.spec.ts`:

```typescript
import { describe, it, expect, beforeEach } from 'vitest'
import { {UseCaseName}UseCase } from './{kebab-action}'
import { InMemory{Entity}sRepository } from '@/repositories/in-memory/in-memory-{entities}-repository'
import { {DomainError} } from './errors/{kebab-error}-error'

let {entity}sRepository: InMemory{Entity}sRepository
let sut: {UseCaseName}UseCase

describe('{UseCaseName} Use Case', () => {
  beforeEach(() => {
    {entity}sRepository = new InMemory{Entity}sRepository()
    sut = new {UseCaseName}UseCase({entity}sRepository)
  })

  it('should be able to {happy path}', async () => {
    const result = await sut.execute({ /* TODO */ })
    expect(result.{entity}.id).toEqual(expect.any(String))
  })

  // Um teste por domain error:
  it('should not be able to {scenario que causa o erro}', async () => {
    await expect(() =>
      sut.execute({ /* TODO */ }),
    ).rejects.toBeInstanceOf({DomainError})
  })
})
```

Se o arquivo já existir, avisar antes de sobrescrever.

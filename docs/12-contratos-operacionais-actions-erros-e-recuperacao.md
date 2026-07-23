# Contratos Operacionais, Actions, Erros e Recuperação

**Versão da proposta:** `operations-v0.2`  
**Status:** em validação

## 1. Objetivo

Definir a fachada operacional usada pelo GPT para acessar o backend sem ultrapassar o limite de Actions, sem expor microendpoints e sem obrigar o GPT a adivinhar como criar, corrigir ou continuar uma operação.

Este documento define como todos os sistemas de jogo são expostos ao GPT.

## 2. Restrição arquitetural

```text
Limite: 30 Actions expostas
Planejadas: 22 Actions
Reservadas: 8 Actions
```

O limite afeta apenas a fachada OpenAPI do GPT. O backend interno pode possuir quantos serviços, comandos, validadores, filas e repositórios forem necessários.

## 3. Arquitetura canônica

```text
GPT
↓
Action Gateway REST + OpenAPI
↓
Serviços de aplicação e domínio
↓
Persistência, filas e integrações
```

Regras:

1. REST + OpenAPI é a interface canônica das Actions do GPT.
2. GraphQL pode ser avaliado futuramente para o frontend, mas não substitui a fachada do GPT.
3. Regras de negócio não ficam presas a controllers ou resolvers.
4. Cada Action chama serviços de aplicação tipados.
5. Endpoints administrativos internos não precisam ser expostos como Actions.

## 4. Action versus operação de domínio

Uma Action representa um domínio ou fluxo coerente. Operações específicas usam um enum discriminador dentro da Action.

Exemplo:

```text
Action: manageInventory
Operações:
EQUIP
UNEQUIP
TRANSFER
CONSUME
LOOT
HARVEST
DROP
DESTROY
SPLIT_STACK
MERGE_STACK
RESERVE
RELEASE_RESERVATION
```

Isso não equivale a uma Action genérica. Todas as operações compartilham as mesmas invariantes de inventário.

## 5. Catálogo inicial de Actions

### Contexto e conteúdo

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 1 | `loadGame` | Carregar campanha, jogador, sessões ativas e resumo |
| 2 | `searchContent` | Pesquisar definições, instâncias, versões e semelhantes |
| 3 | `getContent` | Obter conteúdo completo por ID, código e versão |
| 4 | `manageContent` | Criar, validar, revisar, versionar, migrar ou persistir pacotes |

### Atores e relações

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 5 | `materializeActors` | Validar e materializar um ou vários atores completos |
| 6 | `manageActor` | Alterar estado, comportamento, facção, controle ou escopo |
| 7 | `manageRelationships` | Relações, memórias, grupo, recrutamento e domesticação |

### Inventário

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 8 | `manageInventory` | Equipar, transferir, consumir, saquear, coletar, reservar ou destruir |

### Encontros

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 9 | `startOrLoadEncounter` | Criar ou recuperar encontro |
| 10 | `checkpointEncounter` | Persistir checkpoint intermediário |
| 11 | `submitEncounterResolution` | Validar e persistir resolução consolidada |

### Comércio

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 12 | `startOrLoadTrade` | Carregar sessão, estoques, saldos e faixas |
| 13 | `submitTrade` | Persistir compra, venda e negociação |

### Fabricação

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 14 | `startOrLoadCraft` | Carregar receita, reservas, oficina e probabilidades |
| 15 | `submitCraftResolution` | Persistir consumo, qualidade, instância e XP |

### Perícias

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 16 | `startOrLoadSkillSession` | Carregar desafio de perícia completo |
| 17 | `submitSkillResolution` | Validar resultado, custos, consequências e XP |

### Sistemas futuros reservados

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 18 | `manageMission` | Criar, aceitar, atualizar, concluir, falhar ou abandonar missão |
| 19 | `applyProgression` | Aplicar XP, nível, atributos, perícias e desbloqueios |
| 20 | `advanceWorldTime` | Descanso, viagem, recuperação e atividades demoradas |
| 21 | `manageWorldState` | Locais, facções, reputação, crimes, eventos e consequências |
| 22 | `cancelSession` | Cancelar sessão de forma válida |

O catálogo é versionado. Renomear, fundir ou dividir Actions exige análise de compatibilidade.

## 6. Operações de `manageContent`

```text
SEARCH_OR_VALIDATE_EQUIVALENCE
VALIDATE_CONTENT
CREATE_DEFINITION
CREATE_VARIANT
CREATE_INSTANCE
REVIEW_CONTENT
MIGRATE_CONTENT
VALIDATE_CONTENT_BUNDLE
PERSIST_CONTENT_BUNDLE
PROMOTE_CONTENT_BUNDLE
REVIEW_CONTENT_BUNDLE
```

As operações em lote seguem `bundles-v0.1` ou versão posterior.

## 7. Operações de `materializeActors`

```text
VALIDATE_AND_MATERIALIZE
MATERIALIZE_FROM_VALIDATED_BUNDLE
MATERIALIZE_GROUP
LOAD_MATERIALIZATION
PROMOTE_MATERIALIZED_ACTOR
```

`materializeActors` e `manageContent` reutilizam o mesmo motor interno de validação de pacotes, resolução de referências e transação.

## 8. Regra para novos sistemas

Antes de criar nova Action:

```text
1. verificar se uma Action existente atende;
2. adicionar novo enum quando o domínio for o mesmo;
3. adicionar perfil de snapshot quando apenas a leitura mudar;
4. criar nova Action somente com ciclo, segurança, atomicidade
   ou contrato de erro realmente diferente;
5. atualizar o orçamento de Actions.
```

Uma nova Action exige justificativa objetiva.

## 9. Envelope comum de solicitação

Toda mutação persistente aceita, quando aplicável:

```json
{
  "operation": "ENUM_EXPLICITO",
  "requestId": "request-id",
  "idempotencyKey": "idempotency-key",
  "baseStateVersion": 15,
  "dryRun": false,
  "context": {},
  "payload": {}
}
```

`payload` é discriminado por operação e nunca aceita JSON arbitrário sem schema.

## 10. Envelope comum de resposta

```json
{
  "status": "SUCCESS",
  "requestId": "request-id",
  "previousStateVersion": 15,
  "newStateVersion": 16,
  "result": {},
  "snapshot": null,
  "snapshotDelta": {},
  "warnings": [],
  "issues": [],
  "recoveryActions": [],
  "availableNextOperations": []
}
```

Status canônicos:

```text
SUCCESS
PARTIAL_SUCCESS
REQUIRES_ACTION
REQUIRES_MATERIALIZATION
REQUIRES_REVIEW
CONFLICT
REJECTED
CANCELLED
```

## 11. Validação acumulativa

O backend deve validar todos os problemas independentes detectáveis antes de responder.

Não deve:

```text
validar o primeiro campo
→ retornar erro
→ esperar nova chamada
→ descobrir o segundo erro
```

Deve:

```text
validar schema, orçamento, referências, vínculos e prontidão
→ acumular issues e warnings
→ retornar uma resposta completa
```

Dependências bloqueadas podem ser marcadas como `NOT_EVALUATED_DUE_TO_DEPENDENCY`, deixando claro por que não foram avaliadas.

## 12. Erros acionáveis e localizados

Cada problema informa:

```text
code
severity
blocking
clientRef
nodeId, quando existente
domain
path
ruleCode
message
expected
received
difference
recovery
```

Exemplo:

```json
{
  "code": "PRIMARY_ATTRIBUTE_BUDGET_MISMATCH",
  "severity": "ERROR",
  "blocking": true,
  "clientRef": "actor:bandit-leader",
  "domain": "ATTRIBUTES",
  "path": "/nodes/0/data/primaryAttributes",
  "ruleCode": "PRIMARY_POINTS_BY_LEVEL",
  "message": "A distribuição não usa o orçamento exigido para o nível 5.",
  "expected": { "totalPrimaryPoints": 81 },
  "received": { "totalPrimaryPoints": 79 },
  "difference": 2,
  "recovery": {
    "canRetry": true,
    "suggestedAction": "REDISTRIBUTE_PRIMARY_ATTRIBUTES"
  }
}
```

O backend não deve retornar apenas `400`, `invalid request` ou `operation failed`.

## 13. Componentes opcionais

Ausência de componente não aplicável é válida.

Exemplos:

- ator sem equipamento;
- criatura sem inventário;
- espírito sem dinheiro;
- NPC sem magia;
- animal sem perícias sociais;
- comerciante sem perfil de combate.

Se o componente for enviado, deve ser validado integralmente.

O backend distingue:

```text
OPTIONAL_NOT_PROVIDED
NOT_APPLICABLE
REQUIRED_MISSING
PROVIDED_INVALID
```

## 14. Criação e materialização em lote

Pacotes podem conter:

- definições novas;
- referências a definições existentes;
- variantes;
- instâncias;
- atores;
- itens;
- equipamentos;
- ações;
- habilidades;
- magias;
- perícias;
- passivas;
- drops;
- relações entre todos os nós.

O GPT usa `clientRef` para referenciar nós antes de receber IDs definitivos.

Modos:

```text
VALIDATE_ONLY
MATERIALIZE_TEMPORARY
PERSIST_IF_VALID
PERSIST_REQUIRED
PROMOTE_TEMPORARY
```

A criação completa de ator usa `ALL_OR_NOTHING` por padrão.

## 15. Reutilização e criação automática

Cada nó declara:

```text
REUSE_REQUIRED
REUSE_OR_CREATE
CREATE_NEW
CREATE_VARIANT
INLINE_TEMPORARY
UPDATE_EXISTING
REVIEW_EXISTING
```

O backend:

1. pesquisa conteúdo equivalente;
2. reutiliza IDs e versões compatíveis;
3. cria apenas o que falta e está autorizado;
4. valida todos os vínculos;
5. devolve `referenceMap` com `CREATED`, `REUSED`, `UPDATED` ou `TEMPORARY`.

O backend não fica bloqueado por um pacote misto contendo conteúdos já cadastrados e conteúdos novos.

## 16. Persistência atômica

Padrão:

```text
validar tudo
→ construir plano
→ abrir transação
→ criar e vincular
→ validar estado final
→ commit
```

Com erro bloqueante:

```text
rollback total
```

Não pode existir:

- ator sem vínculos obrigatórios;
- item criado sem proprietário quando exigido;
- equipamento vinculado a slot inválido;
- ação vinculada sem definição válida;
- materiais consumidos sem resultado correspondente.

## 17. Materialização temporária

Conteúdo temporário pode:

- participar de combate;
- ser avaliado;
- consumir recursos;
- sofrer dano e condições;
- produzir eventos;
- ser promovido posteriormente.

Ele não precisa poluir o catálogo permanente.

A resposta informa:

```text
materializationId
materializationVersion
referenceMap
readiness
snapshot
promotionPolicy
```

## 18. Perfis de prontidão

```text
ACTOR_COMPLETE
COMBAT_READY
TRADE_READY
CRAFT_READY
EVALUATION_READY
COMPANION_READY
PERSISTENCE_READY
```

Antes de iniciar uma sessão, o backend retorna:

```text
READY
INCOMPLETE
INVALID
REQUIRES_MATERIALIZATION
REQUIRES_REVIEW
```

Um combate não pode começar como `READY` se arma, ação, custo, munição ou ficha estiverem incompletos.

## 19. Identidade e referências

Usar:

```text
UUID → identidade técnica
code → referência legível de definição
version → versão utilizada
clientRef → referência local no pacote
bigint/sequence → ordenação e números amigáveis
```

Relacionamentos persistentes usam UUID e chaves estrangeiras. `clientRef` é convertido por `referenceMap`.

## 20. Idempotência e concorrência

Toda operação que cria ou transfere valor é idempotente.

Quando:

```text
baseStateVersion != currentStateVersion
```

retornar `CONFLICT` com opções de recarga, reaplicação segura ou cancelamento.

Repetir a mesma `idempotencyKey` e o mesmo hash retorna o resultado anterior. Reutilizar a chave com payload diferente retorna:

```text
IDEMPOTENCY_PAYLOAD_MISMATCH
```

## 21. Snapshots, perfis e deltas

Perfis:

```text
GAME_SUMMARY
COMBAT
TRADE
CRAFT
SKILL
INVENTORY
EVALUATION
RELATIONSHIP
MISSION
WORLD
```

Depois do carregamento completo, usar:

```text
snapshotDelta
changedEntities
invalidatedReferences
requiredReloads
```

## 22. Checkpoint, retomada e cancelamento

Sessões longas suportam:

- carregar;
- criar;
- salvar checkpoint;
- continuar;
- cancelar;
- detectar conflito;
- recuperar último estado válido;
- concluir de forma idempotente.

`cancelSession` usa `sessionType`, evitando uma Action de cancelamento por domínio.

## 23. Segurança de mutações

Não expor:

```text
executeAnything
executeGraphQL
updateAnyEntity
patchDatabaseRecord
runArbitraryCommand
```

Mutações representam comandos de domínio e possuem schema discriminado.

## 24. Requisitos para o Codex

A implementação deve possuir:

1. módulo `action-gateway` separado;
2. OpenAPI com `operationId` estáveis;
3. teste que falhe acima de 30 Actions;
4. teste que mantenha alvo em até 22 sem decisão versionada;
5. schemas discriminados por operação e `nodeType`;
6. validação acumulativa;
7. resolução de `clientRef`;
8. grafo de dependências e persistência ordenada;
9. UUIDs, códigos e versões;
10. `referenceMap` no retorno;
11. idempotência e `stateVersion`;
12. transação e rollback de pacotes;
13. materialização temporária e promoção;
14. perfis de prontidão;
15. erros acionáveis testados;
16. exemplos completos de sucesso e rejeição;
17. métricas de uso, recuperação e tamanho de payload;
18. serviços reutilizáveis pelo GPT e frontend.

## 25. Pendências

- autenticação e autorização por Action;
- limites de payload e paginação;
- timeout e repetição;
- retenção de idempotência;
- catálogo final de `nodeType` e `relationType`;
- duração de materializações temporárias;
- política de promoção de UUIDs temporários;
- formato final de `snapshotDelta`;
- exemplos OpenAPI completos;
- testes de correção automática após múltiplos erros.
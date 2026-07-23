# Contratos Operacionais, Actions, Erros e Recuperação

**Versão da proposta:** `operations-v0.1`  
**Status:** em validação

## 1. Objetivo

Definir a fachada operacional usada pelo GPT para acessar o backend sem ultrapassar o limite de Actions, sem expor microendpoints e sem obrigar o GPT a adivinhar como continuar após erros.

Este documento não substitui os sistemas de jogo. Ele define como todos eles são expostos ao GPT.

## 2. Restrição arquitetural

O GPT do projeto possui limite de:

```text
30 Actions expostas
```

Orçamento inicial:

```text
22 Actions planejadas
8 Actions reservadas
```

As vagas reservadas não devem ser consumidas sem justificativa arquitetural.

O limite afeta somente a fachada OpenAPI disponível ao GPT. O backend interno pode possuir quantos serviços, comandos, validadores, filas e repositórios forem necessários.

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
2. GraphQL pode ser avaliado futuramente para consultas do frontend, mas não substitui a fachada operacional do GPT.
3. Regras de negócio não ficam presas aos controllers REST nem a futuros resolvers GraphQL.
4. Cada Action chama serviços de aplicação específicos.
5. Endpoints administrativos internos não precisam ser expostos como Actions.

## 4. Action versus operação de domínio

Uma Action representa um domínio ou fluxo operacional coerente.

Uma operação interna representa uma intenção específica dentro desse domínio.

Exemplo:

```text
Action: manageInventory
Operações:
- EQUIP
- UNEQUIP
- TRANSFER
- CONSUME
- LOOT
- HARVEST
- DROP
- DESTROY
- SPLIT_STACK
- MERGE_STACK
- RESERVE
- RELEASE_RESERVATION
```

Isso não equivale a uma Action genérica capaz de executar qualquer coisa. Todas as operações de `manageInventory` compartilham o mesmo domínio, invariantes, regras de propriedade, localização, quantidade, peso e atomicidade.

## 5. Catálogo inicial de Actions

### 5.1 Contexto e conteúdo

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 1 | `loadGame` | Carregar campanha, jogador, sessões ativas e resumo do estado |
| 2 | `searchContent` | Pesquisar definições, instâncias, versões e conteúdos semelhantes |
| 3 | `getContent` | Obter conteúdo completo por ID, código e versão |
| 4 | `manageContent` | Criar, revisar, validar, versionar ou migrar conteúdo |

### 5.2 Atores e relações

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 5 | `materializeActors` | Materializar um ou vários atores completos |
| 6 | `manageActor` | Alterar estado, comportamento, facção, controle ou escopo do ator |
| 7 | `manageRelationships` | Relações, memórias, grupo, recrutamento e domesticação |

### 5.3 Inventário

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 8 | `manageInventory` | Equipar, transferir, consumir, saquear, coletar, reservar ou destruir itens |

### 5.4 Encontros

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 9 | `startOrLoadEncounter` | Criar ou recuperar encontro |
| 10 | `checkpointEncounter` | Persistir checkpoint intermediário |
| 11 | `submitEncounterResolution` | Validar e persistir resolução consolidada |

### 5.5 Comércio

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 12 | `startOrLoadTrade` | Carregar sessão comercial, estoques, saldos e faixas |
| 13 | `submitTrade` | Persistir compra, venda e negociação consolidadas |

### 5.6 Fabricação

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 14 | `startOrLoadCraft` | Carregar receita, reservas, ferramentas, oficina e probabilidades |
| 15 | `submitCraftResolution` | Persistir consumo, qualidade, instância produzida e XP |

### 5.7 Perícias

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 16 | `startOrLoadSkillSession` | Carregar desafio de perícia com parâmetros completos |
| 17 | `submitSkillResolution` | Validar resultado, custos, consequências e XP |

### 5.8 Sistemas futuros já reservados no catálogo

| Nº | `operationId` | Finalidade |
|---:|---|---|
| 18 | `manageMission` | Criar, aceitar, atualizar, concluir, falhar ou abandonar missão |
| 19 | `applyProgression` | Aplicar XP, nível, atributos, perícias, profissões e desbloqueios |
| 20 | `advanceWorldTime` | Descanso, viagem, recuperação e atividades demoradas |
| 21 | `manageWorldState` | Locais, facções, reputação, crimes, eventos e consequências |
| 22 | `cancelSession` | Cancelar encontro, comércio, fabricação ou perícia de forma válida |

O catálogo é inicial e versionado. Renomear, fundir ou dividir Actions exige análise de compatibilidade com o schema OpenAPI e com as instruções do GPT.

## 6. Regra para novos sistemas

Ao documentar um novo sistema, seguir esta ordem:

```text
1. verificar se ele pode usar uma Action existente;
2. adicionar novo enum de operação quando pertencer ao mesmo domínio;
3. adicionar perfil de snapshot quando apenas a leitura mudar;
4. criar nova Action somente quando existir ciclo de vida, segurança,
   atomicidade ou contrato de erro realmente diferente;
5. atualizar o orçamento de Actions.
```

Uma nova Action exige resposta afirmativa para pelo menos uma destas perguntas:

- possui ciclo de sessão próprio?
- possui transação autoritativa própria?
- possui permissões ou riscos significativamente diferentes?
- possui entrada e saída incompatíveis com os domínios existentes?
- agrupá-la tornaria o schema confuso ou inseguro para o GPT?

## 7. Envelope comum de solicitação

Toda mutação persistente deve aceitar, quando aplicável:

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

Campos:

- `operation`: intenção interna permitida pela Action;
- `requestId`: correlação e auditoria;
- `idempotencyKey`: evita duplicação por repetição da chamada;
- `baseStateVersion`: controle de concorrência;
- `dryRun`: valida sem persistir quando suportado;
- `context`: campanha, mundo, sessão, ator e demais referências;
- `payload`: dados tipados da operação selecionada.

`payload` nunca deve ser um JSON arbitrário sem schema por operação.

## 8. Envelope comum de resposta

```json
{
  "status": "SUCCESS",
  "requestId": "request-id",
  "previousStateVersion": 15,
  "newStateVersion": 16,
  "result": {},
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

## 9. Erros acionáveis

Um erro precisa informar:

- código estável;
- mensagem legível;
- domínio e campo afetados;
- regra violada;
- valores esperados e recebidos quando seguro;
- possibilidade de correção;
- próxima Action e operação sugeridas;
- parâmetros sugeridos quando possível.

Exemplo:

```json
{
  "status": "REQUIRES_ACTION",
  "issues": [
    {
      "code": "ITEM_NOT_EQUIPPABLE",
      "domain": "INVENTORY",
      "field": "targetSlot",
      "message": "Uma arma de duas mãos já ocupa MAIN_HAND e OFF_HAND.",
      "ruleCode": "TWO_HANDED_SLOT_OCCUPANCY"
    }
  ],
  "recoveryActions": [
    {
      "operationId": "manageInventory",
      "operation": "UNEQUIP",
      "suggestedParameters": {
        "itemInstanceId": "greatsword-instance"
      }
    },
    {
      "operationId": "manageInventory",
      "operation": "CANCEL_EQUIP",
      "suggestedParameters": {}
    }
  ]
}
```

O backend não deve retornar apenas `400`, `invalid request` ou `operation failed` sem orientação suficiente.

## 10. Disponibilidade e prontidão

Snapshots e respostas devem informar:

```text
availableActions
availableOperations
availableNextOperations
constraints
recoveryActions
```

Antes de iniciar uma sessão complexa, o backend deve indicar:

```text
READY
INCOMPLETE
INVALID
REQUIRES_MATERIALIZATION
REQUIRES_REVIEW
```

Exemplo: um combate não pode iniciar como `READY` se uma arma usada não possui perfil de ataque completo.

## 11. Idempotência e concorrência

Toda operação que possa criar ou transferir valor precisa ser idempotente:

- dinheiro;
- itens;
- XP;
- recompensas;
- drops;
- fabricação;
- progressão;
- relações persistentes;
- conclusão de missão.

Quando:

```text
baseStateVersion != currentStateVersion
```

O backend retorna `CONFLICT`, informa as versões e oferece recuperação por recarga, reaplicação segura ou cancelamento.

Repetir a mesma `idempotencyKey` deve devolver o mesmo resultado autoritativo, não executar novamente a consequência.

## 12. Snapshots, perfis e deltas

Para reduzir tokens e tráfego, usar perfis de snapshot:

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

Depois do carregamento completo, respostas podem usar:

```text
snapshotDelta
changedEntities
invalidatedReferences
requiredReloads
```

Definições já carregadas podem ser referenciadas por código e versão sem repetir todo o conteúdo, desde que o GPT possua a versão correta no contexto da sessão.

## 13. Operações em lote

Preferir lote quando as operações compartilham a mesma transação ou contexto:

- materializar grupo de atores;
- transferir vários itens;
- comprar e vender vários itens;
- aplicar consequências de encontro;
- criar várias instâncias em fabricação em lote;
- atualizar vários objetivos da mesma missão.

O lote deve retornar resultado individual por item ou entidade e garantir política explícita de atomicidade total ou parcial.

## 14. Checkpoint, retomada e cancelamento

Sessões longas devem suportar:

- carregar;
- criar;
- salvar checkpoint;
- continuar;
- cancelar;
- detectar conflito;
- recuperar do último estado válido;
- concluir de forma idempotente.

`cancelSession` recebe `sessionType` e aplica a política correspondente sem exigir uma Action de cancelamento para cada domínio.

## 15. Segurança de mutações

Não expor ao GPT:

```text
executeAnything
executeGraphQL
updateAnyEntity
patchDatabaseRecord
runArbitraryCommand
```

Também não expor mutações genéricas que aceitem qualquer JSON.

As mutações devem representar comandos de domínio, preservar invariantes e possuir schema discriminado por `operation`.

## 16. Política para GraphQL

GraphQL pode ser usado futuramente pelo frontend para consultas flexíveis, desde que reutilize os mesmos serviços de aplicação e regras de domínio.

O GPT continua utilizando REST + OpenAPI porque:

- cada `operationId` possui finalidade clara;
- os inputs e outputs são previsíveis;
- os erros podem orientar a próxima operação;
- mutações ficam delimitadas;
- o GPT não precisa construir consultas livres;
- o limite é administrado por agrupamento de domínio.

GraphQL não deve ser adotado apenas para esconder várias operações atrás de uma Action genérica.

## 17. Requisitos para o Codex

A implementação deve possuir:

1. módulo de `action-gateway` separado dos serviços de domínio;
2. OpenAPI com `operationId` estáveis e únicos;
3. contador automatizado que falhe quando houver mais de 30 Actions expostas;
4. teste que mantenha o alvo planejado em até 22, salvo decisão versionada;
5. schemas discriminados por operação;
6. envelopes comuns de solicitação e resposta;
7. idempotência nas mutações críticas;
8. controle por `stateVersion`;
9. erros acionáveis e testados;
10. exemplos completos por Action;
11. métricas de uso, falhas, recuperação e tamanho de resposta;
12. serviços internos reutilizáveis por GPT, frontend e administração.

## 18. Pendências

- validar o limite no editor do GPT e no schema final;
- revisar se 22 Actions são suficientes após os próximos sistemas;
- definir autenticação e autorização por Action;
- definir limites de payload e paginação;
- definir política de timeout e repetição;
- definir atomicidade de operações em lote;
- definir retenção de idempotência;
- definir códigos de erro por domínio;
- definir formato final de `snapshotDelta`;
- criar exemplos OpenAPI completos;
- testar escolha de Action pelo GPT em cenários ambíguos;
- testar recuperação automática após erros.
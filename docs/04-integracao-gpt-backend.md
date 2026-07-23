# Integração entre GPT, Backend e Frontend

**Versão da proposta:** `integration-v0.8`  
**Status:** em validação

## 1. Objetivo

Permitir que o GPT conduza interações, crie e revise conteúdos e resolva sessões localmente, enquanto o backend mantém autoridade sobre dados persistidos, regras versionadas, validações e consequências permanentes.

A integração deve funcionar dentro do limite de 30 Actions do GPT sem expor um endpoint para cada microação.

## 2. Regra central

```text
Backend = autoridade de dados, regras, validação e persistência.
GPT = condutor, criador, revisor e resolvedor local dentro das regras carregadas.
Frontend = visualização e administração usando o mesmo domínio.
```

Conteúdos não são imutáveis. O GPT pode propor correção, balanceamento, evolução e migração. O backend valida antes de persistir.

Depois que uma sessão começa, o GPT não acrescenta retroativamente recursos ausentes do snapshot.

## 3. Action Gateway

A interface do GPT utiliza:

```text
REST + OpenAPI
```

Arquitetura:

```text
GPT
↓
Action Gateway com no máximo 30 operationIds
↓
Serviços de aplicação e domínio
↓
Persistência e integrações
```

Regras:

- alvo inicial de 22 Actions expostas;
- 8 vagas reservadas;
- operações relacionadas são agrupadas por domínio;
- serviços internos não contam como Actions;
- endpoints administrativos não precisam ser expostos ao GPT;
- GraphQL pode ser avaliado para o frontend, não como substituto das Actions do GPT;
- não expor comandos arbitrários ou mutações genéricas sem schema.

O contrato completo está em `operations-v0.1`.

## 4. Responsabilidades

### GPT

- interpretar a intenção do jogador;
- escolher a Action e a operação interna compatíveis;
- carregar fichas, definições, variantes, instâncias, ações, perícias, receitas e estoques;
- usar regras e parâmetros versionados;
- manter estado local;
- calcular resultados permitidos;
- registrar rolagens, operações e decisões;
- reutilizar definições existentes;
- não criar cadastro novo apenas por mudança de qualidade;
- propor variante somente quando houver diferença real;
- usar somente itens, quantidades, cargas, dinheiro e recompensas existentes;
- avaliar sugestões do jogador;
- seguir `recoveryActions` e `availableNextOperations` retornados;
- enviar histórico e resultado consolidado.

### Backend

- persistir definições, variantes, instâncias, atores, pilhas, localizações, propriedade e sessões;
- localizar conteúdo equivalente e impedir duplicação semântica;
- calcular atributos, orçamento, preços, qualidade e pontuações efetivas;
- materializar atores, inventários, contêineres e drops;
- gerar snapshots completos ou deltas;
- controlar `stateVersion` e versões de regras;
- validar e reproduzir resoluções;
- controlar reservas;
- aplicar consumo, criação, XP, saldos, propriedade e localização atomicamente;
- congelar valores resolvidos nas instâncias;
- preservar IDs, versões, vínculos, origem e histórico;
- garantir idempotência nas mutações críticas;
- retornar erros acionáveis com próxima operação sugerida;
- manter o schema exposto dentro do orçamento de Actions.

### Frontend

- exibir catálogo de definições sem duplicação por qualidade;
- agrupar instâncias por definição, variante e qualidade;
- mostrar valores resolvidos, condição, durabilidade e proveniência;
- acompanhar combate, saque, comércio e fabricação;
- permitir revisão administrativa com os mesmos serviços de domínio;
- poder usar REST ou GraphQL futuramente sem duplicar regras.

## 5. Padrão de sessão

```text
1. GPT solicita criação ou carregamento.
2. Backend retorna snapshot completo e versionado.
3. GPT conduz escolhas e resoluções localmente.
4. GPT mantém log estruturado.
5. Checkpoints podem ser enviados.
6. GPT envia resultado consolidado.
7. Backend recalcula e valida.
8. Backend persiste o resultado autoritativo.
9. GPT apresenta o retorno final.
```

Campos comuns:

```text
sessionId
stateVersion
ruleVersions
lifecycleStatus
snapshot
snapshotDelta
resolutionPolicy
availableOperations
availableNextOperations
constraints
recoveryActions
```

## 6. Versões

```json
{
  "ruleVersions": {
    "attributes": "attributes-v0.4",
    "combat": "combat-v0.2",
    "equipment": "equipment-v0.5",
    "actions": "actions-v0.1",
    "economy": "economy-v0.3",
    "skills": "skills-v0.1",
    "crafting": "crafting-v0.2",
    "actors": "actors-v0.1",
    "inventory": "inventory-v0.2",
    "operations": "operations-v0.1",
    "integration": "integration-v0.8"
  }
}
```

Toda resolução, criação ou revisão persistente informa as versões utilizadas.

## 7. Catálogo resumido de Actions

O catálogo inicial possui 22 Actions:

```text
loadGame
searchContent
getContent
manageContent
materializeActors
manageActor
manageRelationships
manageInventory
startOrLoadEncounter
checkpointEncounter
submitEncounterResolution
startOrLoadTrade
submitTrade
startOrLoadCraft
submitCraftResolution
startOrLoadSkillSession
submitSkillResolution
manageMission
applyProgression
advanceWorldTime
manageWorldState
cancelSession
```

Novos sistemas devem reutilizar uma Action existente sempre que pertencerem ao mesmo domínio e às mesmas invariantes.

## 8. Envelope operacional comum

Solicitação persistente:

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

Resposta:

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

`payload` é discriminado por `operation`; não é um JSON arbitrário.

## 9. Prontidão e recuperação

Antes de uma sessão complexa, o backend informa:

```text
READY
INCOMPLETE
INVALID
REQUIRES_MATERIALIZATION
REQUIRES_REVIEW
```

Status de operação:

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

Erros precisam informar código, campo, regra, mensagem e correções possíveis.

Exemplo:

```json
{
  "status": "REQUIRES_ACTION",
  "issues": [
    {
      "code": "ITEM_NOT_EQUIPPABLE",
      "field": "targetSlot",
      "message": "Uma arma de duas mãos já ocupa os dois slots."
    }
  ],
  "recoveryActions": [
    {
      "operationId": "manageInventory",
      "operation": "UNEQUIP",
      "suggestedParameters": {
        "itemInstanceId": "greatsword-instance"
      }
    }
  ]
}
```

## 10. Idempotência e concorrência

Mutações que criam ou transferem valor usam `idempotencyKey`:

- dinheiro;
- itens;
- XP;
- drops;
- recompensas;
- fabricação;
- progressão;
- missões;
- relações persistentes.

Quando:

```text
baseStateVersion != currentStateVersion
```

O backend retorna `CONFLICT` com versões e opções de recarga, reaplicação segura ou cancelamento.

## 11. Identidade de item no snapshot

Todo item materializado deve permitir distinguir:

```text
definitionCode
definitionVersion
familyCode
variantCode
itemInstanceId
quality
resolvedPowerBudget
resolvedModifiers
resolvedBaseBuyPrice
resolvedBaseSellPrice
condition
durability
ruleVersions
```

Qualidade sem `itemInstanceId` representa apenas simulação ou prévia, nunca nova definição automática.

## 12. Criação de instância

```text
GPT escolhe definição ou receita
→ backend localiza definição e variante
→ backend impede duplicação de cadastro
→ sessão resolve sucesso e qualidade
→ backend calcula orçamento e valores finais
→ backend cria instância
→ valores resolvidos são congelados
```

Exemplo:

```json
{
  "operation": "CREATE_ITEM_INSTANCE",
  "definitionCode": "elven-dagger",
  "definitionVersion": 1,
  "variantCode": "BASE",
  "source": {
    "type": "CRAFT_SESSION",
    "referenceId": "craft-session-id"
  }
}
```

## 13. Prevenção de duplicação semântica

Antes de criar definição ou variante, o backend pesquisa por:

- código;
- família;
- nome normalizado;
- categoria;
- material;
- função mecânica;
- ações e passivas;
- tags;
- receita e distribuição.

Possíveis retornos:

```text
USE_EXISTING_DEFINITION
USE_EXISTING_VARIANT
CREATE_NEW_VARIANT
CREATE_NEW_DEFINITION
REVIEW_REQUIRED
```

Mudança somente de qualidade sempre reutiliza definição ou variante existente.

## 14. Materialização de atores

```text
avistamento mínimo
→ interação provável
→ GPT usa materializeActors
→ backend carrega definições
→ cria ator, equipamentos e inventário completos
→ congela snapshot
```

Itens do ator são instâncias vinculadas a definições existentes, com qualidade e valores resolvidos.

## 15. Combate

O snapshot contém:

- participantes materializados;
- atributos e recursos;
- posições e linha do tempo;
- ações disponíveis;
- equipamentos e instâncias relevantes;
- munições, consumíveis, cargas e durabilidade;
- condições e recargas;
- política de rolagens.

O backend valida existência do item, consumo, alcance, tempo, rolagens, dano e resultado.

Item consumido ou destruído não reaparece no saque.

## 16. Inventário e saque

O domínio usa `manageInventory` com operações discriminadas como:

```text
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

O backend valida existência, quantidade, propriedade, localização, compatibilidade de pilha, peso, slots, vínculo, reservas e ausência de duplicação.

## 17. Comércio

`startOrLoadTrade` retorna:

- instâncias e pilhas do estoque;
- qualidade e condição;
- preços-base resolvidos;
- preços de referência;
- faixas mínima e máxima;
- saldos;
- perícia, passivas e relação;
- reservas.

`submitTrade` confirma a transação consolidada e idempotente.

## 18. Fabricação

`startOrLoadCraft` retorna:

- receita;
- definição e variante de saída;
- materiais e ferramentas reservados;
- perícia e profissão;
- chance de sucesso;
- limiares e teto de qualidade;
- perfil de distribuição;
- versões das regras.

`submitCraftResolution` cria uma nova instância por sucesso, nunca uma nova definição por qualidade.

## 19. Revisão e migração

Motivos:

```text
CORRECTION
BALANCE_REVISION
NARRATIVE_EVOLUTION
INSTANCE_STATE_CHANGE
SCOPE_PROMOTION
MIGRATION
```

Escopos:

```text
DEFINITION_FUTURE_INSTANCES
CURRENT_INSTANCE
SELECTED_INSTANCES
ALL_COMPATIBLE_INSTANCES
```

Preferência:

- manter ID e código;
- criar nova versão;
- registrar motivo;
- preservar vínculos;
- não recalcular eventos históricos;
- migrar instâncias apenas explicitamente.

## 20. Congelamento

Instâncias persistem valores resolvidos para evitar alterações silenciosas quando mudarem:

- multiplicadores de qualidade;
- custos de modificadores;
- perfis de distribuição;
- preços-base;
- passivas escaláveis;
- fórmulas de fabricação.

## 21. Checkpoints, lotes e deltas

Preferir operações em lote para materialização de grupos, transferências, comércio, consequências de encontro e fabricação em lote.

Sessões longas suportam checkpoint, retomada e cancelamento por `cancelSession`.

Depois do snapshot inicial, o backend pode retornar:

```text
snapshotDelta
changedEntities
invalidatedReferences
requiredReloads
```

## 22. Erros acionáveis

Exemplos de códigos:

```text
DUPLICATE_ITEM_DEFINITION
VARIANT_NOT_REQUIRED
QUALITY_MUST_BE_INSTANCE_LEVEL
POWER_BUDGET_EXCEEDED
INCOMPATIBLE_STACK
INSTANCE_VERSION_CONFLICT
MIGRATION_REQUIRED
ACTION_NOT_AVAILABLE
OPERATION_NOT_ALLOWED
STATE_VERSION_CONFLICT
IDEMPOTENCY_CONFLICT
SNAPSHOT_INCOMPLETE
```

O retorno sempre explica o problema e oferece correções possíveis.
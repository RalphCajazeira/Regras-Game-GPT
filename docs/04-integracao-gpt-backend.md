# Integração entre GPT, Backend e Frontend

**Versão da proposta:** `integration-v0.7`  
**Status:** em validação

## 1. Objetivo

Permitir que o GPT conduza interações, crie e revise conteúdos e resolva sessões localmente, enquanto o backend mantém autoridade sobre dados persistidos, regras versionadas, validações e consequências permanentes.

## 2. Regra central

```text
Backend = autoridade de dados, regras, validação e persistência.
GPT = condutor, criador, revisor e resolvedor local dentro das regras carregadas.
Frontend = visualização e administração usando o mesmo domínio.
```

Conteúdos não são imutáveis. O GPT pode propor correção, balanceamento, evolução e migração. O backend valida antes de persistir.

Depois que uma sessão começa, o GPT não acrescenta retroativamente recursos ausentes do snapshot.

## 3. Responsabilidades

### GPT

- interpretar a intenção do jogador;
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
- enviar histórico e resultado consolidado.

### Backend

- persistir definições, variantes, instâncias, atores, pilhas, localizações, propriedade e sessões;
- localizar conteúdo equivalente e impedir duplicação semântica;
- calcular atributos, orçamento, preços, qualidade e pontuações efetivas;
- materializar atores, inventários, contêineres e drops;
- gerar snapshots completos;
- controlar `stateVersion` e versões de regras;
- validar e reproduzir resoluções;
- controlar reservas;
- aplicar consumo, criação, XP, saldos, propriedade e localização atomicamente;
- congelar valores resolvidos nas instâncias;
- preservar IDs, versões, vínculos, origem e histórico;
- retornar erros acionáveis.

### Frontend

- exibir catálogo de definições sem duplicação por qualidade;
- agrupar instâncias por definição, variante e qualidade;
- mostrar valores resolvidos, condição, durabilidade e proveniência;
- acompanhar combate, saque, comércio e fabricação;
- permitir revisão administrativa com as mesmas operações.

## 4. Padrão de sessão

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
resolutionPolicy
```

## 5. Versões

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
    "integration": "integration-v0.7"
  }
}
```

Toda resolução, criação ou revisão persistente informa as versões utilizadas.

## 6. Identidade de item no snapshot

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

Qualidade sem `itemInstanceId` representa apenas uma simulação ou prévia, nunca uma nova definição automática.

## 7. Criação de instância

Fluxo:

```text
GPT escolhe definição ou receita
→ backend localiza definição e variante
→ backend impede duplicação de cadastro
→ sessão resolve sucesso e qualidade
→ backend calcula orçamento e valores finais
→ backend cria instância
→ valores resolvidos são congelados
```

Exemplo de solicitação:

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

Exemplo de retorno:

```json
{
  "itemInstanceId": "item-003",
  "definitionCode": "elven-dagger",
  "definitionVersion": 1,
  "variantCode": "BASE",
  "quality": "RARE",
  "resolvedPowerBudget": {},
  "resolvedModifiers": {},
  "resolvedBaseBuyPrice": 240,
  "resolvedBaseSellPrice": 96,
  "ruleVersions": {}
}
```

## 8. Prevenção de duplicação semântica

Antes de criar definição ou variante, o backend deve pesquisar por:

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

Mudança somente de qualidade sempre retorna `USE_EXISTING_DEFINITION` ou `USE_EXISTING_VARIANT`.

## 9. Materialização de atores

```text
avistamento mínimo
→ interação provável
→ GPT solicita materialização
→ backend carrega definições
→ cria ator, equipamentos e inventário completos
→ congela snapshot
```

Itens do ator devem ser instâncias vinculadas a definições existentes, com qualidade e valores resolvidos.

## 10. Combate

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

## 11. Inventário e saque

Snapshot:

```json
{
  "inventorySessionId": "inventory-session-id",
  "stateVersion": 21,
  "ruleVersions": {
    "inventory": "inventory-v0.2",
    "equipment": "equipment-v0.5",
    "actors": "actors-v0.1",
    "integration": "integration-v0.7"
  },
  "actor": {},
  "inventory": {
    "stacks": [],
    "instances": [],
    "containers": [],
    "equipmentSlots": {},
    "currencyBalance": 0,
    "weightSummary": {}
  },
  "availableOperations": [],
  "activeReservations": []
}
```

O backend valida existência, quantidade, propriedade, localização, compatibilidade de pilha, peso, slots, vínculo, reservas e ausência de duplicação.

## 12. Comércio

O snapshot comercial contém:

- instâncias e pilhas do estoque;
- qualidade e condição;
- preços-base resolvidos;
- preços de referência;
- faixas mínima e máxima;
- saldos;
- perícia, passivas e relação;
- reservas.

A transação não altera definição, qualidade ou preço-base congelado da instância.

## 13. Fabricação

O snapshot contém:

- receita;
- definição e variante de saída;
- materiais e ferramentas reservados;
- perícia e profissão;
- chance de sucesso;
- limiares e teto de qualidade;
- perfil de distribuição;
- versões das regras.

O backend cria uma nova instância por sucesso, nunca uma nova definição por qualidade.

## 14. Revisão e migração

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
- migrar instâncias apenas de forma explícita.

## 15. Congelamento

Instâncias persistem valores resolvidos para evitar alterações silenciosas quando mudarem:

- multiplicadores de qualidade;
- custos de modificadores;
- perfis de distribuição;
- preços-base;
- passivas escaláveis;
- fórmulas de fabricação.

## 16. Operações conceituais

```text
create/load/checkpoint/submitEncounter
create/load/submitTradeSession
create/load/submitSkillSession
create/load/submitCraftSession
create/load/submitInventorySession
createItemDefinition
createItemVariant
createItemInstance
reviewContent
migrateInstances
cancelSession
```

## 17. Erros acionáveis

Exemplos:

```text
DUPLICATE_ITEM_DEFINITION
VARIANT_NOT_REQUIRED
QUALITY_MUST_BE_INSTANCE_LEVEL
POWER_BUDGET_EXCEEDED
INCOMPATIBLE_STACK
INSTANCE_VERSION_CONFLICT
MIGRATION_REQUIRED
```

O retorno deve explicar o problema e oferecer correções possíveis.

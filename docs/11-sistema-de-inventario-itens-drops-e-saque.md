# Sistema de Inventário, Itens, Drops e Saque

**Versão da proposta:** `inventory-v0.2`  
**Status:** em validação

## 1. Objetivo

Definir como itens são identificados, instanciados, agrupados, armazenados, equipados, consumidos, transferidos, saqueados, destruídos e persistidos.

O sistema deve evitar dois problemas:

1. o GPT inventar itens ou drops depois que uma interação começou;
2. o banco possuir vários cadastros do mesmo item apenas porque a qualidade é diferente.

## 2. Princípios

1. Definição, variante e instância são conceitos diferentes.
2. Qualidade pertence à instância, não cria automaticamente nova definição.
3. A definição usa qualidade Comum como referência.
4. Todo item persistente possui identidade, localização e propriedade conhecidas.
5. Equipar, consumir, vender, fabricar, transferir ou destruir altera o mesmo estado autoritativo.
6. Itens empilháveis só compartilham pilha quando seus dados relevantes são compatíveis.
7. Itens carregados por ator materializado são definidos antes da interação.
8. Drops usam regras previamente declaradas e resultado reproduzível.
9. O GPT pode criar ou revisar conteúdo por operação válida, mas não altera retroativamente um snapshot.
10. Toda operação persistente é validada e atômica.

## 3. Camadas do domínio

```text
Definição de Item
→ Variante opcional
→ Instância de Item
→ Pilha ou unidade individual
→ Localização
→ Propriedade
→ Operação
→ Histórico
```

## 4. Categorias

```text
EQUIPMENT
CONSUMABLE
AMMUNITION
MATERIAL
TOOL
QUEST_ITEM
KEY_ITEM
CONTAINER
TREASURE
MISCELLANEOUS
```

## 5. Definição de item

A definição é reutilizável e versionada.

```json
{
  "code": "elven-dagger",
  "version": 1,
  "name": "Adaga Élfica",
  "category": "EQUIPMENT",
  "referenceQuality": "COMMON",
  "familyCode": "elven-dagger",
  "variantCode": "BASE",
  "stackable": false,
  "unitWeight": 0.7,
  "currencyCode": "CROWN",
  "commonBaseBuyPrice": 120,
  "commonBaseSellPrice": 48,
  "equipmentProfileCode": "elven-dagger-equipment",
  "tags": ["ELVEN", "DAGGER", "FINESSE"],
  "tradePolicy": {
    "tradable": true,
    "sellable": true,
    "droppable": true,
    "destroyable": true
  }
}
```

A definição não recebe um novo código apenas porque uma unidade é Rara ou Épica.

## 6. Variante

Variante existe apenas quando há diferença real de identidade ou mecânica.

```json
{
  "familyCode": "elven-dagger",
  "variantCode": "MOON_SILVER",
  "name": "Adaga Élfica Lunar"
}
```

Não é variante:

- qualidade;
- condição;
- durabilidade;
- fabricante;
- proprietário;
- nome personalizado sem efeito mecânico.

Pode ser variante:

- material estrutural diferente;
- elemento;
- distribuição de poder;
- ações ou passivas diferentes;
- receita ou função própria.

## 7. Instância de item

A instância representa uma unidade concreta.

```json
{
  "itemInstanceId": "item-instance-001",
  "definitionCode": "elven-dagger",
  "definitionVersion": 1,
  "variantCode": "BASE",
  "quantity": 1,
  "quality": "RARE",
  "resolvedPowerBudget": {
    "referenceCommonPoints": 10,
    "qualityMultiplier": 1.4,
    "maximumPoints": 14,
    "usedPoints": 14
  },
  "resolvedModifiers": {
    "physicalAttack": 11,
    "physicalAccuracy": 3
  },
  "resolvedBaseBuyPrice": 240,
  "resolvedBaseSellPrice": 96,
  "condition": {
    "conditionPercent": 100,
    "state": "PRISTINE"
  },
  "durability": {
    "current": 100,
    "maximum": 100
  },
  "owner": {
    "ownerType": "ACTOR",
    "ownerId": "player-1"
  },
  "location": {
    "type": "ACTOR_INVENTORY",
    "referenceId": "player-1"
  },
  "binding": "UNBOUND",
  "provenance": {
    "acquisitionMode": "CRAFTED",
    "craftedByActorId": "player-1"
  },
  "ruleVersions": {
    "equipment": "equipment-v0.5",
    "crafting": "crafting-v0.2",
    "inventory": "inventory-v0.2"
  }
}
```

A instância pode possuir:

- qualidade concreta;
- valores resolvidos;
- condição e durabilidade;
- cargas;
- nome personalizado;
- vínculo;
- origem;
- proprietário;
- localização;
- estado de roubo;
- histórico.

## 8. Congelamento dos valores

A instância guarda os resultados usados em sua criação:

- definição e versão;
- variante;
- qualidade;
- orçamento de poder;
- modificadores;
- preços-base;
- ações ou passivas escaláveis;
- versões de regra;
- semente de geração.

Alterar multiplicadores futuros não muda silenciosamente itens já criados. Migração exige operação explícita.

## 9. Pilhas

Uma pilha representa unidades compatíveis.

```json
{
  "stackId": "stack-id",
  "definitionCode": "arrow",
  "definitionVersion": 1,
  "variantCode": "BASE",
  "quality": "COMMON",
  "quantity": 36,
  "stackLimit": 100,
  "unitWeight": 0.05,
  "totalWeight": 1.8
}
```

Itens só podem ser unidos quando forem compatíveis em:

- definição e versão;
- variante;
- qualidade;
- condição ou faixa permitida;
- cargas;
- vínculo;
- modificadores resolvidos;
- estado de roubo;
- proprietário e localização;
- políticas especiais.

Equipamentos com durabilidade ou histórico individual normalmente permanecem como instâncias separadas.

## 10. Agrupamento visual

O frontend ou GPT pode agrupar visualmente sem perder as instâncias:

```text
Adaga Élfica [Comum] ×2
Adaga Élfica [Rara] ×1
```

Internamente:

```text
item-001 COMMON
item-002 COMMON
item-003 RARE
```

O catálogo administrativo mostra uma única definição:

```text
Adaga Élfica
├── 2 instâncias Comuns
└── 1 instância Rara
```

## 11. Localização autoritativa

Uma instância ou quantidade só pode estar em um local por vez:

```text
ACTOR_INVENTORY
EQUIPPED_SLOT
CONTAINER
MERCHANT_STOCK
WORLD_GROUND
CORPSE_LOOT
HARVEST_SOURCE
STASH
CRAFT_RESERVATION
TRADE_RESERVATION
CONSUMED
DESTROYED
```

A mesma unidade não pode estar equipada, vendida e reservada simultaneamente.

## 12. Propriedade e vínculo

Proprietários:

```text
ACTOR
PARTY
FACTION
MERCHANT
WORLD
UNOWNED
```

Vínculos:

```text
UNBOUND
BOUND_TO_ACTOR
BOUND_TO_PARTY
BOUND_TO_FACTION
QUEST_LOCKED
```

## 13. Peso e capacidade

```text
Peso Total = soma(peso unitário × quantidade)
```

Faixas propostas:

```text
NORMAL
ENCUMBERED
HEAVY
OVERLOADED
```

Sobrecarga pode afetar movimento, velocidade de ação, corrida e custo de Vigor. Valores exatos precisam de simulação.

## 14. Consumíveis, munições e cargas

A ação declara o item necessário:

```json
{
  "itemCosts": [
    {
      "definitionCode": "arrow",
      "quantity": 1,
      "consumeAt": "ON_RESOLVE"
    }
  ]
}
```

Momentos:

```text
ON_START
ON_RESOLVE
ON_SUCCESS
```

- flecha disparada reduz a pilha;
- poção bebida deixa o inventário;
- ferramenta pode perder durabilidade;
- item com cargas reduz `charges.current`;
- item consumido não aparece depois no saque.

## 15. Reservas

```text
CRAFT_RESERVATION
TRADE_RESERVATION
QUEST_RESERVATION
SYSTEM_RESERVATION
```

Enquanto reservado, o item não pode ser usado por operação incompatível.

## 16. Condição e durabilidade

```text
Qualidade = potencial da instância
Condição = conservação atual
Durabilidade = desgaste mecânico
```

Estados:

```text
PRISTINE
USED
WORN
DAMAGED
BROKEN
DESTROYED
```

Item quebrado pode perder ações, modificadores ou possibilidade de equipar, conforme regra carregada.

## 17. Proveniência e roubo

Pode registrar:

- fabricado por;
- encontrado em;
- comprado de;
- saqueado de;
- roubado de;
- recebido em missão;
- gerado com ator ou evento.

```json
{
  "theftState": {
    "stolen": true,
    "originalOwnerId": "merchant-1",
    "witnessed": false,
    "knownToFactionCodes": []
  }
}
```

## 18. Fontes de saque

### Itens carregados

Equipamentos, inventário, consumíveis, munições, ferramentas e dinheiro realmente possuídos.

### Drops naturais

Pele, carne, presas, essência, núcleo e materiais do corpo. Não ficam no inventário do ator.

### Recompensas externas

Baús, esconderijos, objetivos, objetos do cenário e recompensas de missão.

Não devem ser atribuídas ao corpo sem regra explícita.

## 19. Regras de drop

```json
{
  "dropRules": [
    {
      "itemCode": "alpha-wolf-hide",
      "sourceType": "HARVEST",
      "mode": "GUARANTEED",
      "quantity": { "minimum": 1, "maximum": 1 },
      "conditions": ["BODY_NOT_BURNED"]
    },
    {
      "itemCode": "beast-core",
      "sourceType": "HARVEST",
      "mode": "CHANCE",
      "chancePercent": 15,
      "quantity": { "minimum": 1, "maximum": 1 }
    }
  ]
}
```

Modos:

```text
GUARANTEED
CHANCE
CONDITIONAL
CHOICE
SCRIPTED
```

Momentos:

```text
ON_MATERIALIZATION
ON_DEFEAT
ON_HARVEST
ON_OBJECTIVE_COMPLETION
```

Resultados aleatórios usam semente persistida e não podem ser rerrolados ao recarregar.

## 20. Condição do corpo

A forma da derrota só altera coleta quando a regra já existir:

- corpo queimado pode danificar pele;
- cabeça destruída pode impedir presas intactas;
- núcleo removido não reaparece;
- veneno não destrói carne sem regra declarada.

## 21. Pilha de saque

```json
{
  "lootContainerId": "loot-bandit-1",
  "sourceActorId": "bandit-1",
  "currencyBalance": 18,
  "items": [],
  "harvestSources": [],
  "accessPolicy": {
    "allowedActorIds": ["player-1"],
    "expiresAtWorldTime": null
  }
}
```

A pilha reflete o estado real final.

## 22. Operações

```text
CREATE_INSTANCE
TRANSFER_ITEM
SPLIT_STACK
MERGE_STACK
EQUIP_ITEM
UNEQUIP_ITEM
CONSUME_ITEM
USE_CHARGE
RESERVE_ITEM
RELEASE_RESERVATION
DROP_ITEM
DESTROY_ITEM
LOOT_ITEM
HARVEST_RESOURCE
REPAIR_ITEM
MIGRATE_INSTANCE
```

## 23. Criação e revisão

O GPT pode:

- localizar definição existente;
- propor nova definição ou variante;
- criar instância por operação válida;
- revisar definição, variante ou instância;
- solicitar migração explícita.

O backend deve impedir duplicação semântica, validar qualidade, orçamento, localização, propriedade e histórico.

## 24. Responsabilidades

### GPT

- reutilizar definição existente;
- não cadastrar o mesmo item por qualidade;
- usar apenas quantidades e instâncias carregadas;
- registrar consumo, transferência e saque;
- propor variante somente com diferença real;
- avaliar revisões de forma coerente.

### Backend

- manter definições, variantes e instâncias;
- resolver qualidade e valores finais;
- controlar localização, propriedade e reservas;
- validar pilhas, peso, slots e vínculos;
- impedir duplicação e uso simultâneo;
- preservar versões e proveniência;
- aplicar operações atomicamente.

## 25. Pendências

- limites de pilha por categoria;
- faixas de condição compatíveis em pilhas;
- penalidades de sobrecarga;
- durabilidade e reparo;
- contêineres aninhados;
- tempo de coleta e saque;
- divisão de saque em grupo;
- identificação de itens desconhecidos;
- expiração de cadáveres e pilhas;
- migração de instâncias;
- agrupamento visual no frontend.

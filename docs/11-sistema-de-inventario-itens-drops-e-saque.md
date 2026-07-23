# Sistema de Inventário, Itens, Drops e Saque

**Versão da proposta:** `inventory-v0.1`  
**Status:** em validação

## 1. Objetivo

Definir como itens são criados, identificados, armazenados, empilhados, equipados, consumidos, transferidos, saqueados, destruídos e persistidos.

O sistema deve impedir que o GPT invente itens, munições, consumíveis, dinheiro ou drops depois que uma interação já começou. Ao mesmo tempo, o GPT pode criar ou revisar definições e instâncias por meio das operações adequadas, desde que a proposta seja coerente e validada pelo backend.

## 2. Princípios

1. Definição de item e instância de item são conceitos diferentes.
2. Todo item persistente possui identidade, localização e propriedade conhecidas.
3. Quantidade, peso, condição, cargas e durabilidade são dados mecânicos, não descrições narrativas livres.
4. Equipamentos, consumíveis, munições, materiais, ferramentas e itens de missão usam a mesma estrutura-base.
5. Itens empilháveis somente compartilham uma pilha quando seus dados relevantes são compatíveis.
6. Equipar, consumir, transferir, saquear, fabricar ou destruir itens altera o mesmo estado autoritativo.
7. Itens carregados por um ator materializado são definidos antes da interação relevante.
8. Drops naturais e recompensas usam regras previamente declaradas e resultado reproduzível.
9. O GPT pode propor criação e revisão, mas não acrescenta retroativamente recursos a um snapshot em andamento.
10. Toda alteração persistente é validada e aplicada atomicamente pelo backend.

## 3. Camadas do domínio

```text
Definição de Item
→ Instância de Item
→ Pilha ou unidade individual
→ Localização
→ Propriedade
→ Operação
→ Histórico
```

## 4. Categorias de item

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

A categoria organiza o comportamento padrão, mas cada definição declara explicitamente suas propriedades.

### 4.1 Equipamento

Utiliza as regras de `equipment-v0.4` ou versão posterior:

- slots;
- modificadores;
- ações concedidas;
- passivas;
- proficiências;
- qualidade;
- peso;
- requisitos.

### 4.2 Consumível

Pode fornecer uma ou mais ações de uso e declarar:

- quantidade consumida;
- momento do consumo;
- efeitos;
- tempo de uso;
- alvo;
- cargas;
- recipiente vazio ou subproduto.

### 4.3 Munição

É consumida por ações compatíveis, como disparos de arco, besta ou armas especiais.

### 4.4 Material

Pode ser utilizado em fabricação, reparo, aprimoramento, encantamento, coleta ou missão.

### 4.5 Ferramenta

Pode possuir qualidade, condição, durabilidade, proficiências requeridas e bônus de perícia.

### 4.6 Item de missão ou chave

Pode possuir restrições de descarte, venda, transferência, destruição e duplicação.

## 5. Definição de item

A definição é reutilizável e versionada.

```json
{
  "code": "minor-healing-potion",
  "version": 1,
  "name": "Poção de Cura Menor",
  "category": "CONSUMABLE",
  "stackable": true,
  "stackLimit": 20,
  "unitWeight": 0.25,
  "currencyCode": "CROWN",
  "baseBuyPrice": 30,
  "baseSellPrice": 12,
  "grantedActionCodes": ["drink-minor-healing-potion"],
  "tags": ["POTION", "HEALING"],
  "tradePolicy": {
    "tradable": true,
    "sellable": true,
    "droppable": true,
    "destroyable": true
  }
}
```

Campos comuns:

- código e versão;
- nome e descrição;
- categoria e tags;
- empilhamento;
- peso unitário;
- preços-base;
- ações, passivas ou proficiências concedidas;
- políticas de comércio, descarte e destruição;
- requisitos;
- compatibilidades;
- dados específicos da categoria.

## 6. Instância de item

A instância representa o item concreto existente no mundo.

```json
{
  "itemInstanceId": "item-instance-id",
  "definitionCode": "iron-dagger",
  "definitionVersion": 3,
  "quantity": 1,
  "quality": "RARE",
  "condition": {
    "conditionPercent": 86,
    "state": "USED"
  },
  "durability": {
    "current": 72,
    "maximum": 100
  },
  "charges": null,
  "owner": {
    "ownerType": "ACTOR",
    "ownerId": "bandit-1"
  },
  "location": {
    "type": "EQUIPPED_SLOT",
    "referenceId": "bandit-1",
    "slot": "MAIN_HAND"
  },
  "binding": "UNBOUND",
  "provenance": {
    "acquisitionMode": "GENERATED_WITH_ACTOR",
    "sourceId": "bandit-1"
  }
}
```

Uma instância pode possuir:

- qualidade concreta;
- condição e durabilidade;
- quantidade;
- cargas;
- modificadores próprios permitidos;
- nome personalizado;
- vínculo;
- origem;
- proprietário;
- localização;
- estado de roubo;
- histórico relevante.

## 7. Pilhas

Uma pilha representa várias unidades compatíveis.

```json
{
  "stackId": "stack-id",
  "definitionCode": "arrow",
  "definitionVersion": 1,
  "quantity": 36,
  "stackLimit": 100,
  "unitWeight": 0.05,
  "totalWeight": 1.8
}
```

### 7.1 Compatibilidade de pilha

Itens somente podem ser mesclados quando forem compatíveis em todos os campos relevantes:

- mesma definição e versão;
- mesma qualidade, quando aplicável;
- mesma condição ou faixa de condição definida;
- mesmas cargas;
- mesmo vínculo;
- mesmos modificadores próprios;
- mesma identificação e estado de roubo;
- nenhuma propriedade única incompatível.

Equipamentos individualizados normalmente não são empilháveis.

### 7.2 Operações de pilha

```text
SPLIT_STACK
MERGE_STACKS
ADD_QUANTITY
REMOVE_QUANTITY
```

O backend impede quantidade negativa, excedente do limite ou duplicação.

## 8. Localizações de item

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
DESTROYED
CONSUMED
```

Um item persistente deve possuir exatamente uma localização autoritativa por vez, salvo estruturas explicitamente compostas.

## 9. Contêineres

Um contêiner pode possuir outros itens.

```json
{
  "containerId": "backpack-instance",
  "containerType": "ITEM_INSTANCE",
  "ownerActorId": "player-1",
  "capacity": {
    "maximumWeight": 25,
    "maximumSlots": null
  },
  "contents": []
}
```

A primeira versão prioriza peso. Volume, grade espacial e tamanho físico ficam opcionais para evolução futura.

Contêineres podem ser:

- mochila;
- baú;
- bolsa;
- cofre;
- cadáver;
- pilha de saque;
- estoque de comerciante;
- depósito do grupo.

## 10. Peso e capacidade de carga

```text
Peso Total = soma(peso unitário × quantidade)
```

O peso do item equipado continua contando para carga, salvo regra explícita.

`carryCapacity` define a capacidade normal do ator.

Faixas propostas:

```text
NORMAL: peso ≤ 100% da capacidade
ENCUMBERED: peso > 100% e ≤ 125%
HEAVY: peso > 125% e ≤ 150%
OVERLOADED: peso > 150%
```

Possíveis consequências, ainda sujeitas a simulação:

- redução de `movementSpeed`;
- redução de `physicalActionSpeed`;
- aumento de custo de Vigor;
- limitação de corrida;
- impossibilidade de usar certas ações quando sobrecarregado.

O backend calcula a faixa e entrega os modificadores finais. O GPT não inventa penalidades diferentes das regras carregadas.

## 11. Equipar e desequipar

Equipar é uma transferência entre `ACTOR_INVENTORY` e `EQUIPPED_SLOT`.

O backend valida:

- propriedade;
- existência da instância;
- requisitos;
- slots permitidos;
- slots ocupados;
- uso de uma ou duas mãos;
- incompatibilidades;
- condições mínimas;
- vínculo;
- estado do ator;
- tempo ou ação necessária, quando estiver em encontro.

A operação é atômica. Se um item de duas mãos exigir liberar a mão secundária, todas as mudanças devem ser aplicadas juntas ou nenhuma é persistida.

## 12. Consumíveis, munições e cargas

Uma ação declara o recurso material necessário:

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

Momentos possíveis:

```text
ON_START
ON_RESOLVE
ON_SUCCESS
```

O snapshot deve informar as pilhas e instâncias disponíveis. O GPT registra a unidade utilizada; o backend confirma a remoção.

Exemplos:

- uma flecha disparada deixa a pilha de munição;
- uma poção bebida deixa o inventário;
- uma poção interrompida pode ou não ser consumida conforme a ação;
- uma ferramenta pode perder durabilidade sem desaparecer;
- um item com cargas reduz `charges.current`.

## 13. Propriedade, vínculo e origem

### 13.1 Propriedade

```text
ACTOR
PARTY
FACTION
MERCHANT
WORLD
UNOWNED
```

### 13.2 Vínculo

```text
UNBOUND
BOUND_TO_ACTOR
BOUND_TO_PARTY
BOUND_TO_FACTION
QUEST_LOCKED
```

### 13.3 Origem e proveniência

Pode registrar:

- fabricado por;
- encontrado em;
- recebido em missão;
- comprado de;
- saqueado de;
- roubado de;
- criado junto a um ator;
- gerado por evento.

Esses dados ajudam comércio, reputação, crimes, missões e narrativa.

## 14. Itens roubados e restritos

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

O estado de roubo não altera automaticamente a definição. Ele pertence à instância.

Comerciantes podem recusar, denunciar ou pagar menos conforme regras econômicas e conhecimento do crime.

## 15. Condição e durabilidade

Qualidade e condição são conceitos separados.

```text
Qualidade = potencial e nível de fabricação
Condição = estado atual de conservação
Durabilidade = recurso mecânico de desgaste
```

Estados propostos:

```text
PRISTINE
USED
WORN
DAMAGED
BROKEN
DESTROYED
```

Uma instância quebrada pode:

- perder ações concedidas;
- reduzir modificadores;
- ficar inequipável;
- exigir reparo;
- continuar utilizável de forma limitada quando a definição permitir.

As fórmulas de desgaste e reparo permanecem no sistema de fabricação e qualidade.

## 16. Fontes de saque

O saque é dividido em fontes distintas.

### 16.1 Itens carregados

São equipamentos, inventário, consumíveis, munições, ferramentas e dinheiro realmente possuídos pelo ator.

Se forem consumidos, destruídos, transferidos ou perdidos antes da derrota, não aparecem no saque.

### 16.2 Drops naturais

Representam partes do corpo, essência, núcleo, pele, carne, presas, minérios incorporados ou outros recursos coletáveis.

Não ficam no inventário do ator.

### 16.3 Recompensas do encontro ou local

Podem vir de:

- baús;
- esconderijos;
- recompensas de missão;
- objetos do cenário;
- tesouros protegidos;
- conclusão de objetivo.

Não devem ser atribuídas ao corpo de um ator sem regra explícita.

## 17. Regras de drop

Uma definição pode declarar:

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

## 18. Momento da resolução do drop

A política deve ser declarada:

```text
ON_MATERIALIZATION
ON_DEFEAT
ON_HARVEST
ON_OBJECTIVE_COMPLETION
```

Para evitar rerrolagem ou invenção, qualquer resultado aleatório usa a `generationSeed` da instância ou outra semente persistida.

Mesmo quando o resultado exato só é revelado na derrota ou coleta, ele deve ser reproduzível e não pode mudar ao recarregar a cena.

## 19. Condição do corpo e coleta

A forma de derrota pode afetar drops naturais somente quando a regra existir antes da resolução.

Exemplos:

- corpo queimado reduz qualidade da pele;
- cabeça destruída impede coleta de presas intactas;
- núcleo removido por habilidade especial deixa de aparecer depois;
- veneno não destrói automaticamente carne sem regra declarada.

O GPT registra os fatos mecânicos relevantes. O backend aplica as condições das regras de coleta.

## 20. Pilha de saque

Ao derrotar ou abandonar um ator, itens recuperáveis podem ser transferidos para uma pilha de saque ou cadáver.

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

A pilha deve refletir o estado final real:

- itens consumidos não aparecem;
- equipamentos destruídos podem aparecer como sucata;
- dinheiro transferido antes da derrota não reaparece;
- munição restante mantém a quantidade correta.

## 21. Saque e conhecimento do jogador

O backend pode conhecer todo o conteúdo, enquanto o jogador conhece apenas o que foi observado ou procurado.

Estados possíveis:

```text
VISIBLE
CONCEALED
HIDDEN
IDENTIFIED
UNIDENTIFIED
```

Buscar um corpo, abrir um contêiner ou usar Avaliação pode revelar itens conforme ação, tempo, perícia, ocultação e contexto.

O GPT não revela itens ocultos antes de uma descoberta válida.

## 22. Operações canônicas

```text
ADD_ITEM
REMOVE_ITEM
MOVE_ITEM
TRANSFER_ITEM
SPLIT_STACK
MERGE_STACKS
EQUIP_ITEM
UNEQUIP_ITEM
CONSUME_ITEM
USE_CHARGE
DAMAGE_ITEM
REPAIR_ITEM
DROP_ITEM
LOOT_ITEM
HARVEST_ITEM
DESTROY_ITEM
RESERVE_FOR_CRAFT
RELEASE_RESERVATION
RESERVE_FOR_TRADE
```

Toda operação informa:

- ator ou origem;
- destino;
- instância ou pilha;
- quantidade;
- versão do estado;
- motivo;
- regra utilizada.

## 23. Reservas

Itens usados em fabricação ou comércio podem ser reservados para impedir uso duplo.

```json
{
  "reservationId": "reservation-id",
  "type": "CRAFT_RESERVATION",
  "itemInstanceIds": [],
  "stackAllocations": [],
  "expiresAt": null
}
```

Enquanto reservados, não podem ser vendidos, consumidos, equipados ou reservados por outra operação incompatível.

## 24. Criação e revisão pelo GPT

O GPT pode propor:

- nova definição de item;
- nova instância permitida por uma regra;
- correção de categoria;
- ajuste de empilhamento;
- revisão de peso;
- revisão de preço-base;
- alteração de ação concedida;
- correção de drop;
- correção de vínculo ou política comercial;
- migração de instâncias compatíveis.

A proposta deve declarar:

```text
CORRECTION
BALANCE_REVISION
NARRATIVE_EVOLUTION
INSTANCE_STATE_CHANGE
MIGRATION
```

O backend valida coerência, orçamento, propriedade, localização, impacto econômico, vínculos e histórico.

Uma revisão não acrescenta retroativamente itens a um inventário congelado sem uma causa válida de estado, evento ou migração explícita.

## 25. Sessão de inventário ou saque

Operações simples podem ser diretas. Interações complexas podem usar snapshot.

```json
{
  "inventorySessionId": "inventory-session-id",
  "stateVersion": 21,
  "ruleVersions": {
    "inventory": "inventory-v0.1",
    "equipment": "equipment-v0.4",
    "actors": "actors-v0.1"
  },
  "actor": {},
  "containers": [],
  "equipmentSlots": {},
  "availableOperations": [],
  "weightSummary": {},
  "constraints": {}
}
```

O GPT pode conduzir:

- inspeção;
- comparação;
- seleção múltipla;
- organização;
- saque;
- equipar e desequipar;
- descarte;
- preparação para comércio ou fabricação.

Ao final, envia operações consolidadas. O backend recalcula e persiste atomicamente.

## 26. Erros acionáveis

Exemplo de munição insuficiente:

```json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_ITEM_QUANTITY",
    "message": "A ação exige 1 Flecha, mas nenhuma unidade compatível está disponível.",
    "details": {
      "definitionCode": "arrow",
      "required": 1,
      "available": 0
    },
    "recoveryActions": [
      "Escolha outra ação.",
      "Equipe ou transfira munição compatível.",
      "Cancele o disparo antes da resolução."
    ]
  }
}
```

Exemplo de item ocupado:

```json
{
  "success": false,
  "error": {
    "code": "ITEM_RESERVED_BY_ANOTHER_OPERATION",
    "message": "O material está reservado por uma sessão de fabricação ativa.",
    "recoveryActions": [
      "Conclua a fabricação.",
      "Cancele a reserva.",
      "Selecione outra instância compatível."
    ]
  }
}
```

## 27. Responsabilidades

### GPT

- carregar o inventário ou snapshot necessário;
- usar somente instâncias, pilhas e quantidades existentes;
- distinguir conhecimento real de conteúdo oculto;
- registrar consumo, munição, cargas e transferências;
- não inventar saque após a derrota;
- propor criação e revisão quando necessário;
- avaliar sugestões do jogador segundo as regras;
- enviar operações persistentes ao backend.

### Backend

- manter definições, instâncias, pilhas, propriedade e localização;
- calcular peso e sobrecarga;
- validar empilhamento, slots, consumo e quantidade;
- controlar reservas;
- resolver drops de forma reproduzível;
- criar pilhas de saque e fontes de coleta;
- transferir itens e moeda atomicamente;
- impedir duplicação, quantidade negativa e uso duplo;
- preservar versões, origem e histórico;
- retornar erros acionáveis.

### Frontend futuro

- exibir inventário, peso, slots e contêineres;
- diferenciar itens identificados, ocultos, vinculados e roubados;
- permitir organização, comparação, equipamento e transferência;
- mostrar reservas, durabilidade, condição e origem;
- utilizar as mesmas operações e validações do GPT.

## 28. Pendências de validação

- limites de pilha por categoria;
- faixas e penalidades de sobrecarga;
- política definitiva de durabilidade;
- compatibilidade por condição em pilhas;
- tempo de saque e busca;
- identificação de itens;
- expiração de cadáveres e pilhas de saque;
- acesso compartilhado do grupo;
- divisão de saque;
- recuperação de projéteis;
- contêineres aninhados;
- itens quebrados e sucata;
- políticas completas para itens roubados, únicos e vinculados.

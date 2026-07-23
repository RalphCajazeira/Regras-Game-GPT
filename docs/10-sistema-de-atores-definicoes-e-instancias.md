# Sistema de Atores, Definições e Instâncias

**Versão da proposta:** `actors-v0.1`  
**Status:** em validação

## 1. Objetivo

Definir um modelo único para qualquer entidade capaz de agir, possuir estado, participar de relações ou receber consequências no jogo.

O termo técnico **Ator** pode representar:

- personagem do jogador;
- pessoa ou NPC;
- animal;
- monstro;
- criatura;
- espírito;
- morto-vivo;
- construto;
- invocação;
- companheiro;
- comerciante;
- chefe.

O sistema deve permitir criação guiada e completa somente quando a entidade se tornar relevante, impedindo que o GPT invente posteriormente equipamentos, itens, dinheiro, ações, magias ou recompensas que não existiam.

## 2. Princípios

1. Todo participante mecânico utiliza a mesma estrutura-base de Ator.
2. Natureza, espécie, arquétipo, papel, facção e forma de controle são conceitos separados.
3. Uma entidade distante ou irrelevante pode existir apenas como avistamento.
4. Antes de qualquer interação mecânica relevante, a entidade deve ser materializada com ficha completa.
5. A ficha materializada define previamente atributos, recursos, ações, passivas, equipamentos, inventário, dinheiro e recompensas aplicáveis.
6. O GPT pode propor, revisar e atualizar qualquer definição ou instância quando houver motivo coerente.
7. O backend valida estrutura, orçamento, vínculos, coerência e impacto, mas não transforma conteúdos existentes em dados imutáveis.
8. Sugestões do jogador são avaliadas pelo GPT; não são aplicadas automaticamente.
9. Revisões mantêm identidade, versão e histórico.
10. Eventos passados não são recalculados automaticamente.

## 3. Natureza, espécie e papel

O que o ator **é** não deve ser confundido com o papel que exerce.

### 3.1 Natureza — `nature`

```text
PERSON
ANIMAL
MONSTER
CREATURE
SPIRIT
UNDEAD
CONSTRUCT
ELEMENTAL
SUMMON
OTHER
```

### 3.2 Espécie — `speciesCode`

Exemplos:

```text
HUMAN
WOLF
ALPHA_WOLF
FOREST_ELF
FIRE_SPIRIT
CRYSTAL_GOLEM
```

A espécie pertence a uma definição própria e pode fornecer características biológicas, espirituais ou estruturais.

### 3.3 Arquétipo — `archetypeCode`

Representa função ou construção mecânica reutilizável:

```text
BANDIT_SWORDSMAN
BANDIT_ARCHER
ALPHA_PACK_LEADER
TRAVELING_MERCHANT
FIRE_CASTER
HEAVY_GUARDIAN
```

### 3.4 Papel e controle

```json
{
  "controlMode": "GPT_CONTROLLED",
  "partyRole": "NONE",
  "factionCode": "ROAD_BANDITS",
  "hostilityState": "HOSTILE"
}
```

Valores possíveis de controle:

```text
PLAYER_CONTROLLED
GPT_CONTROLLED
ALLIED_AUTONOMOUS
SCRIPTED
SUMMONER_DIRECTED
```

Um Lobo Alfa domesticado continua sendo `ANIMAL`. O que muda é sua facção, vínculo, domesticação, lealdade e participação no grupo.

## 4. Definição, instância e conhecimento

### 4.1 Definição de ator

Modelo reutilizável que informa como criar indivíduos daquele tipo.

Pode conter:

- natureza e espécie;
- faixa de nível;
- orçamento e tendências de atributos;
- ações e passivas obrigatórias ou possíveis;
- equipamentos possíveis;
- inventário possível;
- dinheiro permitido;
- comportamento-base;
- XP-base;
- tabelas de saque e coleta;
- resistências, imunidades e fraquezas;
- regras de recrutamento ou domesticação;
- tags temáticas;
- versão da definição.

Exemplo:

```json
{
  "code": "alpha-wolf",
  "version": 1,
  "nature": "ANIMAL",
  "speciesCode": "WOLF",
  "archetypeCode": "ALPHA_PACK_LEADER",
  "levelRange": { "minimum": 4, "maximum": 8 },
  "requiredActionCodes": ["wolf-bite", "alpha-howl"],
  "possibleActionCodes": ["wolf-charge"],
  "equipmentPolicy": "NATURAL_ONLY",
  "tamingPolicyCode": "WILD_ALPHA_TAMING",
  "baseXpReward": 120,
  "tags": ["BEAST", "PACK_LEADER", "TAMABLE"]
}
```

### 4.2 Instância de ator

Indivíduo concreto criado a partir de uma definição.

A instância possui valores exatos:

- nível;
- atributos;
- recursos atuais;
- ações e passivas;
- equipamentos;
- inventário;
- dinheiro;
- recompensas resolvidas;
- posição;
- condições;
- vínculos;
- estado atual;
- semente de geração;
- versão da definição usada.

### 4.3 Conhecimento do jogador

A ficha completa pode ser conhecida pelo GPT sem ser revelada integralmente ao jogador.

```json
{
  "mechanicalState": {
    "currentHealth": 120,
    "hiddenInventory": [],
    "weaknessCodes": ["FIRE"]
  },
  "playerKnowledge": {
    "identityStatus": "SPECIES_KNOWN",
    "healthInformation": "HEALTHY",
    "knownWeaknessCodes": [],
    "knownActionCodes": ["wolf-bite"]
  }
}
```

Avaliação, observação, investigação e experiências anteriores modificam somente o conhecimento disponível, não a ficha real.

## 5. Ciclo de relevância

### 5.1 Avistamento — `SIGHTING`

Representação mínima para algo percebido, mas ainda sem interação relevante.

```json
{
  "sightingId": "sighting-42",
  "apparentNature": "PERSON",
  "apparentArchetype": "BANDIT",
  "estimatedCount": 4,
  "distanceMeters": 80,
  "visibleEquipmentTags": ["SWORD"],
  "awarenessState": "UNAWARE"
}
```

Um avistamento não autoriza o GPT a inventar inventário oculto, magias ou consumíveis.

### 5.2 Materializado — `MATERIALIZED`

A ficha completa é criada antes de uma interação mecânica relevante.

Gatilhos comuns:

- ataque iniciado pelo jogador ou ator;
- uso de Avaliação;
- aproximação para conversar;
- perseguição, roubo, investigação ou infiltração;
- tentativa de recrutamento ou domesticação;
- participação em missão;
- possibilidade de saque;
- consequência persistente.

### 5.3 Persistente — `PERSISTENT`

A instância continua existindo depois da cena quando:

- recebe identidade relevante;
- sobrevive e pode reaparecer;
- torna-se aliado, rival ou companheiro;
- participa de missão;
- possui relação ou memória relevante;
- recebe consequência permanente;
- o GPT ou jogador identifica relevância narrativa coerente.

## 6. Escopo de persistência

```text
SCENE
ENCOUNTER
CAMPAIGN
WORLD
```

- `SCENE`: representação temporária de curta duração.
- `ENCOUNTER`: ficha completa durante uma interação ou encontro.
- `CAMPAIGN`: permanece vinculada à campanha.
- `WORLD`: entidade relevante em todo o mundo ou compartilhada por campanhas.

Promoções reutilizam a mesma instância:

```text
ENCOUNTER → CAMPAIGN
CAMPAIGN → WORLD
```

Não deve ser criada uma cópia nova ao promover um bandido sobrevivente, animal domesticado ou NPC recorrente.

## 7. Materialização completa

Quando um ator se torna relevante, sua ficha deve estar pronta para todas as interações plausíveis naquele contexto.

### 7.1 Identidade

- ID e código da definição;
- nome ou descrição anônima;
- natureza;
- espécie;
- arquétipo;
- variante;
- nível;
- categoria de ameaça;
- facção;
- tags.

### 7.2 Mecânica

- cinco atributos primários;
- atributos secundários;
- Vida, Mana e Vigor;
- resistências, imunidades e fraquezas;
- perícias e proficiências;
- ações, habilidades, magias e passivas;
- regras de comportamento e uso das ações.

### 7.3 Posse

- equipamentos e slots;
- inventário;
- consumíveis;
- munições;
- ferramentas;
- dinheiro;
- itens ocultos;
- itens de missão.

### 7.4 Recompensas

- XP-base;
- itens carregados;
- equipamentos recuperáveis;
- drops naturais;
- materiais coletáveis;
- condições de saque;
- subprodutos.

### 7.5 Comportamento

- agressividade;
- medo;
- moral;
- tendência de fuga;
- prioridades de combate;
- objetivos;
- capacidade de diálogo;
- regras de rendição;
- possibilidade de recrutamento ou domesticação.

## 8. Congelamento durante a interação

Depois da materialização, os dados relevantes ficam congelados na versão do snapshot usado.

O GPT não pode acrescentar retroativamente:

- poção inexistente;
- magia não cadastrada;
- arma oculta não materializada;
- dinheiro adicional;
- imunidade criada depois do ataque;
- drop inventado após a derrota.

Alterações válidas durante a interação devem resultar de uma regra existente:

- consumir item;
- equipar ou desequipar;
- receber condição;
- desbloquear fase prevista;
- transformação declarada;
- reforço materializado por evento.

Uma revisão estrutural da definição não altera silenciosamente o snapshot em andamento. O encontro usa as versões carregadas, salvo migração explícita e segura.

## 9. Geração determinística

Toda instância materializada deve possuir:

```text
generationSeed
```

A mesma instância deve reproduzir:

- atributos;
- ações;
- equipamentos;
- inventário;
- dinheiro;
- variante;
- resultados previamente resolvidos.

Isso impede que recarregar uma cena gere recompensas diferentes.

## 10. Inventário, dinheiro e saque

### 10.1 Itens carregados

Itens realmente possuídos pela instância.

```json
{
  "equipment": {
    "mainHand": "rusty-sword-instance",
    "body": "worn-clothes-instance"
  },
  "inventory": [
    {
      "itemInstanceId": "minor-healing-potion-instance",
      "quantity": 1,
      "visibility": "HIDDEN"
    }
  ],
  "currencyBalance": 18
}
```

Se a poção for consumida, ela deixa de existir no saque.

### 10.2 Drops naturais e coleta

Animais, monstros e criaturas podem gerar materiais do corpo sem carregá-los no inventário.

```json
{
  "harvestDrops": [
    {
      "itemCode": "wolf-hide",
      "quantity": 1,
      "conditionCode": "BODY_NOT_BURNED"
    },
    {
      "itemCode": "wolf-fang",
      "quantity": 2,
      "conditionCode": "HEAD_NOT_DESTROYED"
    }
  ]
}
```

A forma de derrota pode modificar condição e quantidade quando a regra estiver declarada previamente.

## 11. Grupos e atores anônimos

Um grupo pode ser materializado em lote.

```json
{
  "groupArchetypeCode": "ROAD_BANDIT_AMBUSH",
  "count": 5,
  "averageLevel": 4,
  "context": {
    "locationCode": "OLD_ROAD",
    "encounterRole": "AMBUSH"
  }
}
```

O resultado pode conter indivíduos completos e ainda anônimos:

- Bandido Espadachim A;
- Bandido Espadachim B;
- Bandido Arqueiro;
- Bandido com Escudo;
- Líder Bandido.

Se um indivíduo se tornar relevante, sua mesma instância é promovida e pode receber nome, memória e persistência.

## 12. Companheiros, recrutamento e domesticação

Companheiro não é natureza nem espécie.

Campos separados:

```text
partyMembership
controlMode
loyalty
recruitmentStatus
tamingStatus
factionCode
```

```json
{
  "partyMembership": {
    "partyId": "player-party",
    "status": "ACTIVE"
  },
  "controlMode": "ALLIED_AUTONOMOUS",
  "tamingStatus": "TAMED",
  "loyalty": 62
}
```

Ao entrar no grupo, a ficha-base do ator não é substituída. São atualizados vínculos, facção, controle, progressão e comportamento.

## 13. Relações e memórias

Relações são direcionais. A confiança de A em B não implica o mesmo valor de B em A.

Estrutura resumida:

```json
{
  "sourceActorId": "lysandra",
  "targetActorId": "kael",
  "values": {
    "affinity": 40,
    "trust": 25,
    "respect": 35,
    "fear": 0,
    "hostility": 0,
    "loyalty": 10,
    "romanticInterest": 15
  },
  "impressionTags": [
    "HELPFUL",
    "RELIABLE_IN_COMBAT"
  ]
}
```

Estados diferentes podem coexistir:

```text
partyStatus: COMPANION
friendshipStatus: CLOSE_FRIEND
romanceStatus: PARTNER
legalBondStatus: MARRIED
```

Memórias persistidas devem resumir fatos relevantes, não copiar toda a conversa.

## 14. Avaliação e revelação de informações

Avaliação pode materializar a ficha quando necessário, mas revela apenas o permitido por:

- potência da habilidade;
- perícia aplicável;
- nível do usuário;
- nível e ocultação do alvo;
- conhecimento anterior;
- rolagem;
- regras do mundo.

Possíveis informações:

- espécie;
- nível aproximado;
- estado de saúde;
- equipamentos visíveis;
- ações observadas;
- resistências ou fraquezas;
- ameaça estimada.

## 15. Criação guiada

### 15.1 Definição existente

```text
GPT identifica o conceito
→ solicita materialização
→ backend carrega definição e orçamento
→ GPT ou backend resolve variante e contexto
→ backend valida
→ retorna instância completa
```

### 15.2 Definição inexistente

```text
GPT propõe definição completa
→ backend valida estrutura, orçamento e vínculos
→ GPT corrige eventuais problemas
→ definição é versionada
→ instância é materializada
```

A criação deve incluir todos os campos mecânicos necessários. Campos não aplicáveis são declarados vazios, nulos ou zerados conforme o schema; não são omitidos de forma ambígua.

## 16. Autoridade de revisão do GPT

O GPT pode propor e executar, por meio das Actions adequadas, revisões em qualquer conteúdo:

- definição de ator;
- instância;
- ação;
- habilidade;
- magia;
- passiva;
- equipamento;
- item;
- receita;
- relação;
- memória;
- comportamento;
- regra de saque;
- regra de recrutamento.

O backend não deve rejeitar uma alteração apenas porque o conteúdo já existe. Ele deve avaliar a proposta segundo as regras atuais.

### 16.1 Tipos de alteração

```text
CORRECTION
BALANCE_REVISION
NARRATIVE_EVOLUTION
INSTANCE_STATE_CHANGE
SCOPE_PROMOTION
MIGRATION
```

- `CORRECTION`: corrige erro, incoerência ou campo incompleto.
- `BALANCE_REVISION`: altera números ou comportamento para equilíbrio.
- `NARRATIVE_EVOLUTION`: mudança justificada por acontecimentos do mundo.
- `INSTANCE_STATE_CHANGE`: altera apenas o estado de uma instância.
- `SCOPE_PROMOTION`: promove persistência ou relevância.
- `MIGRATION`: aplica uma nova versão a instâncias existentes de forma explícita.

### 16.2 Sugestões do jogador

Uma sugestão do jogador é uma proposta, não uma ordem mecânica.

O GPT deve:

1. identificar o objetivo da sugestão;
2. consultar regras e conteúdo atual;
3. verificar coerência temática e mecânica;
4. avaliar impacto e escopo;
5. aceitar, ajustar ou recusar com justificativa;
6. enviar uma proposta estruturada ao backend;
7. adaptar a proposta quando o backend retornar erro acionável.

O GPT não deve aplicar uma sugestão incoerente apenas para concordar com o jogador.

### 16.3 Escopo da revisão

Toda revisão declara o alcance:

```text
DEFINITION_FUTURE_INSTANCES
CURRENT_INSTANCE
SELECTED_INSTANCES
ALL_COMPATIBLE_INSTANCES
```

Eventos passados permanecem registrados com as versões usadas.

### 16.4 Versionamento

Preferência:

- manter o mesmo ID e código;
- incrementar versão;
- registrar motivo e autor da revisão;
- preservar vínculos;
- declarar política para instâncias existentes;
- não recalcular eventos históricos.

## 17. Validação de coerência

O backend deve validar estrutura e orçamento. O GPT deve avaliar contexto e significado usando as fontes e instruções.

Exemplo problemático:

```text
Cajado da Ignição
Tags: FIRE, STAFF
Efeito: +10% em habilidades de Água
```

Retorno esperado:

```json
{
  "code": "THEMATIC_EFFECT_MISMATCH",
  "message": "O item possui identidade de Fogo, mas o efeito beneficia habilidades de Água sem justificativa declarada.",
  "recoveryActions": [
    "Altere o efeito para habilidades de Fogo.",
    "Altere a identidade elemental do item.",
    "Declare uma propriedade híbrida ou contraditória coerente."
  ]
}
```

Combinações incomuns não são proibidas quando possuem justificativa e estrutura compatível.

## 18. Responsabilidades

### GPT

- identificar relevância;
- selecionar arquétipo;
- propor novas definições;
- materializar personalidade, aparência e comportamento;
- usar apenas recursos materializados;
- manter separação entre ficha real e conhecimento do jogador;
- propor promoções e revisões;
- avaliar sugestões do jogador;
- adaptar propostas a erros acionáveis.

### Backend

- manter definições e versões;
- fornecer orçamento e regras;
- gerar ou validar instâncias;
- aplicar semente determinística;
- validar atributos, ações, equipamentos, inventário e saque;
- controlar escopo de persistência;
- promover instâncias sem duplicação;
- persistir relações, memórias e vínculos;
- validar revisões e migrações;
- retornar erros acionáveis;
- preservar histórico e versões utilizadas.

## 19. Operações conceituais

```text
createActorDefinition
reviseActorDefinition
materializeActor
materializeActorGroup
loadActor
promoteActorScope
reviseActorInstance
submitActorStateChange
revealActorKnowledge
reviewContentCoherence
```

Os nomes definitivos dependem da API.

## 20. Pendências

- orçamento de criação por nível e categoria de ameaça;
- regras de variantes;
- composição de grupos;
- nível de desafio;
- moral e comportamento;
- relações e memórias completas;
- domesticação e recrutamento;
- loot, inventário e coleta;
- condições de morte, incapacidade e fuga;
- critérios de promoção automática;
- migração de instâncias após revisão de definição;
- validação temática extensível por tags e domínios.
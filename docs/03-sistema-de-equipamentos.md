# Sistema de Equipamentos

**Versão da proposta:** `equipment-v0.5`  
**Status:** em validação

## 1. Objetivo

Definir uma estrutura universal para equipamentos sem duplicar cadastros por qualidade.

Uma Adaga Élfica possui uma única definição reutilizável. Qualidades Inferior, Comum, Rara, Épica e Lendária são resultados de instâncias concretas criadas, encontradas ou compradas.

## 2. Regra central

```text
Definição única com perfil Comum de referência
→ qualidade aplicada à instância
→ orçamento resolvido
→ modificadores finais congelados
```

Não devem existir definições separadas apenas por qualidade:

```text
ERRADO:
elven-dagger-inferior
elven-dagger-common
elven-dagger-rare

CORRETO:
elven-dagger
```

## 3. Definição, variante e instância

### 3.1 Definição de equipamento

Representa o equipamento reutilizável:

```json
{
  "code": "elven-dagger",
  "version": 1,
  "name": "Adaga Élfica",
  "referenceQuality": "COMMON",
  "level": 5,
  "equipmentType": "WEAPON",
  "weaponType": "DAGGER",
  "familyCode": "elven-dagger",
  "variantCode": "BASE",
  "materialCode": "ELVEN_STEEL",
  "weight": 0.7,
  "allowedSlots": ["MAIN_HAND", "OFF_HAND"],
  "handedness": "ONE_HANDED",
  "baseModifiers": {
    "physicalAttack": 10,
    "magicalAttack": 0,
    "physicalDefense": 0,
    "magicalDefense": 0,
    "physicalAccuracy": 2
  },
  "qualityScalingProfileCode": "STANDARD_LIGHT_WEAPON",
  "grantedActionCodes": ["basic-elven-dagger-attack"],
  "grantedPassiveCodes": [],
  "tags": ["ELVEN", "DAGGER", "LIGHT_WEAPON", "FINESSE"]
}
```

A definição usa `COMMON` como referência para atributos, orçamento e preço-base.

### 3.2 Variante

Variante representa diferença real de identidade ou funcionamento, não qualidade.

Exemplos:

```text
Adaga Élfica
Adaga Élfica Lunar
Adaga Élfica Cerimonial
Adaga Élfica Flamejante
```

Uma variante pode alterar:

- material;
- distribuição de poder;
- elemento;
- ações ou passivas;
- receita;
- aparência estrutural;
- requisitos;
- empunhadura ou função mecânica.

Estrutura:

```json
{
  "familyCode": "elven-dagger",
  "variantCode": "MOON_SILVER"
}
```

A variante ainda pode existir em qualquer qualidade permitida.

### 3.3 Instância

Representa uma unidade concreta:

```json
{
  "itemInstanceId": "item-001",
  "definitionCode": "elven-dagger",
  "definitionVersion": 1,
  "quality": "RARE",
  "resolvedPowerBudget": {
    "referenceCommonPoints": 10,
    "qualityMultiplier": 1.4,
    "maximumPoints": 14,
    "usedPoints": 14
  },
  "resolvedModifiers": {
    "physicalAttack": 11,
    "magicalAttack": 0,
    "physicalDefense": 0,
    "magicalDefense": 0,
    "physicalAccuracy": 3
  },
  "condition": "PRISTINE",
  "durability": {
    "current": 100,
    "maximum": 100
  },
  "craftedByActorId": "player-1",
  "generationSeed": "seed-value",
  "ruleVersions": {
    "equipment": "equipment-v0.5",
    "crafting": "crafting-v0.2",
    "inventory": "inventory-v0.2"
  }
}
```

## 4. Qualidades

```text
INFERIOR
COMMON
RARE
EPIC
LEGENDARY
```

Multiplicadores iniciais:

| Qualidade | Poder | Preço |
|---|---:|---:|
| Inferior | `0,60` | `0,50` |
| Comum | `1,00` | `1,00` |
| Raro | `1,40` | `2,00` |
| Épico | `1,80` | `4,00` |
| Lendário | `2,40` | `8,00` |

Os valores ainda precisam de simulações.

## 5. Escalonamento da qualidade

A qualidade multiplica o orçamento de poder da referência Comum.

```text
Orçamento da Instância =
Orçamento Comum de Referência
× Multiplicador de Qualidade
```

Arredondamento deve ser determinístico e versionado.

### Exemplo simples

```text
Adaga Élfica Comum:
ATK Físico +10

Inferior: +6
Comum: +10
Rara: +14
Épica: +18
Lendária: +24
```

### Vários modificadores

Quando existem vários modificadores, não se multiplica cegamente cada número. O sistema:

1. calcula o custo total do perfil Comum;
2. aplica o multiplicador da qualidade;
3. distribui o novo orçamento conforme o perfil da definição;
4. respeita custos diferentes dos modificadores;
5. aplica limites e arredondamento;
6. grava os resultados na instância.

Exemplo:

```text
Perfil Comum:
ATK Físico +8
Precisão Física +2
Custo total: 10

Qualidade Rara:
10 × 1,40 = 14 pontos

Resultado possível:
ATK Físico +11
Precisão Física +3
```

## 6. Perfil de distribuição

A definição deve indicar quais modificadores recebem o orçamento adicional.

```json
{
  "powerDistribution": [
    {
      "attribute": "physicalAttack",
      "priority": 1,
      "minimumPercent": 70,
      "maximumPercent": 85
    },
    {
      "attribute": "physicalAccuracy",
      "priority": 2,
      "minimumPercent": 15,
      "maximumPercent": 30
    }
  ]
}
```

A qualidade não autoriza o GPT a adicionar atributos sem relação com a identidade do item.

## 7. O que escala

Pode escalar quando declarado:

- ATK Físico e Mágico;
- DEF Física e Mágica;
- Precisões;
- Esquiva e resistências;
- recursos máximos;
- atributos primários;
- velocidades;
- outros modificadores estruturados.

Não muda automaticamente:

- peso;
- slots;
- quantidade de mãos;
- tipo da arma;
- material;
- tags;
- aparência-base;
- ações concedidas;
- requisitos conceituais.

## 8. Passivas e efeitos

Passivas e ações não escalam automaticamente com qualidade.

A definição deve declarar:

```json
{
  "passiveCode": "elven-balance",
  "scalesWithQuality": true,
  "qualityValues": {
    "INFERIOR": 1,
    "COMMON": 2,
    "RARE": 3,
    "EPIC": 4,
    "LEGENDARY": 5
  }
}
```

Sem perfil explícito, a passiva permanece igual.

## 9. Slots

```text
HEAD
BODY
CHEST
HANDS
LEGS
FEET
MAIN_HAND
OFF_HAND
NECK
RING_LEFT
RING_RIGHT
BACK
WAIST
ACCESSORY
```

`BODY` representa roupa-base. `CHEST` representa proteção estrutural.

Armas podem ser:

```text
ONE_HANDED
TWO_HANDED
VERSATILE
```

Uma arma de duas mãos ocupa `MAIN_HAND` e `OFF_HAND` pela mesma instância.

## 10. Orçamento-base

Proposta inicial:

```text
Pontos-base do nível = 3 + (nível × 2)
```

```text
Pontos Comuns Máximos =
máximo(
  1,
  arredondar_para_baixo(
    pontos-base
    × multiplicador do slot
  )
)
```

A qualidade é aplicada depois sobre essa referência Comum.

Multiplicadores de slot propostos:

| Slot ou categoria | Multiplicador |
|---|---:|
| Corpo | `0,40` |
| Cabeça | `0,70` |
| Peito | `1,40` |
| Mãos | `0,60` |
| Pernas | `0,90` |
| Pés | `0,60` |
| Costas | `0,70` |
| Cintura | `0,50` |
| Pescoço | `0,50` |
| Anel | `0,40` |
| Acessório | `0,40` |
| Arma de uma mão | `1,00` |
| Arma de duas mãos | `1,70` |
| Escudo | `1,00` |

## 11. Modificadores universais

Todo equipamento declara todos os campos, inclusive os zerados:

```json
{
  "primaryModifiers": {
    "strength": 0,
    "agility": 0,
    "dexterity": 0,
    "vitality": 0,
    "intelligence": 0
  },
  "secondaryModifiers": {
    "maxHealth": 0,
    "maxMana": 0,
    "maxStamina": 0,
    "physicalAttack": 0,
    "magicalAttack": 0,
    "physicalDefense": 0,
    "magicalDefense": 0,
    "physicalAccuracy": 0,
    "magicalAccuracy": 0,
    "evasion": 0,
    "criticalChance": 0,
    "criticalDamage": 0,
    "criticalDefense": 0,
    "physicalResistance": 0,
    "mentalResistance": 0,
    "initiative": 0,
    "movementSpeed": 0,
    "physicalActionSpeed": 0,
    "castingSpeed": 0,
    "carryCapacity": 0,
    "perception": 0,
    "stealth": 0
  }
}
```

## 12. Penalidades

```text
Custo Líquido =
Custo Positivo - Recuperação Negativa
```

Penalidades recuperam orçamento conforme o custo do modificador, mas devem ser coerentes com o item. O limite global de recuperação ainda precisa de validação.

## 13. Preços-base

A definição guarda a referência Comum:

```text
commonBaseBuyPrice
commonBaseSellPrice
currencyCode = CROWN
```

A instância guarda os preços-base resolvidos para sua qualidade e versão:

```text
resolvedBaseBuyPrice
resolvedBaseSellPrice
```

Mercado, condição, comerciante e negociação alteram somente o preço da transação.

## 14. Peso, condição e durabilidade

Qualidade não altera automaticamente o peso.

```text
Peso = material + tamanho + estrutura + efeitos explícitos
```

Qualidade, condição e durabilidade são conceitos diferentes:

```text
Qualidade = potencial da fabricação
Condição = conservação atual
Durabilidade = desgaste mecânico
```

## 15. Congelamento e versionamento

A instância grava:

- definição e versão;
- qualidade;
- orçamento resolvido;
- modificadores resolvidos;
- preços-base resolvidos;
- ações e passivas resolvidas quando escaláveis;
- versões das regras;
- semente de geração.

Mudanças futuras nos multiplicadores não alteram silenciosamente itens existentes. Migrações precisam ser explícitas.

## 16. Quando criar nova definição

Não cria nova definição:

- mudança apenas de qualidade;
- condição ou durabilidade;
- proprietário ou fabricante;
- estado de roubo;
- nome personalizado sem efeito mecânico.

Cria variante ou definição nova quando houver diferença real:

- material estrutural diferente;
- elemento diferente;
- distribuição de poder diferente;
- ações ou passivas diferentes;
- receita diferente;
- empunhadura ou função diferente;
- identidade mecânica própria.

## 17. Responsabilidades

### GPT

- reutilizar definições existentes;
- não duplicar cadastro por qualidade;
- propor variante apenas quando houver diferença real;
- usar perfil Comum como referência;
- avaliar coerência temática;
- enviar criação ou revisão estruturada.

### Backend

- localizar definição e variante;
- impedir duplicação semântica;
- calcular orçamento por qualidade;
- distribuir e validar modificadores;
- congelar os resultados na instância;
- preservar versões e histórico;
- retornar erros acionáveis.

## 18. Pendências

- calibrar multiplicadores de qualidade e slot;
- definir arredondamento definitivo;
- calibrar custos dos modificadores;
- definir limite de recuperação negativa;
- definir perfis de distribuição por categoria;
- definir escalonamento de passivas e ações;
- testar agrupamento visual de instâncias iguais;
- definir migrações de itens já existentes.

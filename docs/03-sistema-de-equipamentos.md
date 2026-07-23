# Sistema de Equipamentos

**Versão da proposta:** `equipment-v0.1`  
**Status:** em validação

## 1. Objetivo

Definir uma estrutura universal para qualquer item equipável. Todo equipamento utiliza os mesmos campos de modificadores, ainda que a maioria esteja em `0`.

O GPT propõe itens coerentes. O backend recalcula orçamento, valida slots, preços, peso e modificadores antes de persistir.

## 2. Princípios

1. Todo item equipável declara todos os modificadores.
2. Campos não utilizados permanecem em `0`.
3. Equipamentos podem alterar atributos primários e secundários.
4. Nível e qualidade determinam o orçamento de poder.
5. Penalidades negativas recuperam pontos para distribuição positiva.
6. O custo líquido não pode ultrapassar o orçamento máximo.
7. Todo item possui preço de compra e venda.
8. A qualidade aumenta o preço, mas não altera automaticamente o peso.
9. O peso depende de material, tamanho, estrutura e efeitos explícitos.
10. Equipamentos de duas mãos ocupam os dois slots de mão.

## 3. Slots

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

### 3.1 Corpo — `BODY`

Representa roupa-base, aparência e proteção leve:

- roupa comum;
- traje de aventureiro;
- túnica;
- uniforme;
- vestido;
- vestes de viagem;
- traje cerimonial.

Pode não conceder bônus ou conceder modificadores mínimos. Permanece equipado sob o item de Peito.

### 3.2 Peito — `CHEST`

Representa proteção estrutural usada sobre a roupa:

- cota de malha;
- couraça;
- peitoral de couro;
- armadura de placas;
- manto reforçado;
- veste de batalha encantada.

`BODY` e `CHEST` são slots independentes.

### 3.3 Mãos — `HANDS`

Luvas, manoplas e braçadeiras. Não deve ser confundido com `MAIN_HAND` e `OFF_HAND`, usados para armas, escudos e focos.

## 4. Ocupação das mãos

### 4.1 Uma mão

```json
{
  "handedness": "ONE_HANDED",
  "allowedSlots": ["MAIN_HAND", "OFF_HAND"],
  "occupiedSlots": 1
}
```

Permite combinações como:

- adaga + adaga;
- espada curta + escudo;
- varinha + adaga;
- cajado de uma mão + livro mágico.

### 4.2 Duas mãos

```json
{
  "handedness": "TWO_HANDED",
  "allowedSlots": ["MAIN_HAND"],
  "occupiedSlots": 2
}
```

Ao equipar, o backend marca as duas mãos como ocupadas pela mesma instância.

Exemplos:

- espada larga;
- arco longo;
- lança pesada;
- machado de guerra;
- cajado longo.

### 4.3 Versátil

Reservado para evolução futura. Um item `VERSATILE` poderá possuir perfis diferentes para uma ou duas mãos.

## 5. Qualidade

Ordem de poder adotada nesta versão:

```text
INFERIOR
COMMON
RARE
EPIC
LEGENDARY
```

Em português:

- Inferior;
- Comum;
- Raro;
- Épico;
- Lendário.

A qualidade representa fabricação, conservação, pureza, refinamento, encantamento ou técnica especial.

## 6. Orçamento-base por nível

Proposta inicial para um item comum de referência:

```text
Pontos-base do nível = 3 + (nível × 2)
```

| Nível | Pontos-base comuns |
|---:|---:|
| 1 | 5 |
| 2 | 7 |
| 3 | 9 |
| 4 | 11 |
| 5 | 13 |
| 10 | 23 |

## 7. Multiplicadores de qualidade

| Qualidade | Multiplicador de poder |
|---|---:|
| Inferior | `0,60` |
| Comum | `1,00` |
| Raro | `1,40` |
| Épico | `1,80` |
| Lendário | `2,40` |

```text
Pontos após qualidade =
arredondar_para_baixo(
  pontos-base do nível × multiplicador da qualidade
)
```

No nível 1:

| Qualidade | Pontos |
|---|---:|
| Inferior | 3 |
| Comum | 5 |
| Raro | 7 |
| Épico | 9 |
| Lendário | 12 |

## 8. Multiplicador de slot

A importância do slot pode ajustar o orçamento.

| Slot ou categoria | Multiplicador proposto |
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

```text
Pontos Máximos =
máximo(
  1,
  arredondar_para_baixo(
    pontos-base
    × multiplicador de qualidade
    × multiplicador de slot
  )
)
```

Os multiplicadores de slot ainda precisam de simulações. Uma arma de duas mãos deve compensar a perda da mão secundária sem equivaler automaticamente a dois itens completos.

## 9. Custos de distribuição

Os modificadores utilizam unidades de orçamento.

| Modificador | Benefício por ponto gasto |
|---|---:|
| ATK Físico | `+1` |
| ATK Mágico | `+1` |
| DEF Física | `+1` |
| DEF Mágica | `+1` |
| Precisão Física | `+1` |
| Precisão Mágica | `+1` |
| Esquiva | `+1` |
| Defesa Crítica | `+1` |
| Resistência Física | `+1` |
| Resistência Mental | `+1` |
| Iniciativa | `+1` |
| Percepção | `+1` |
| Furtividade | `+1` |
| Vida Máxima | `+5` |
| Mana Máxima | `+5` |
| Vigor Máximo | `+5` |
| Capacidade de Carga | `+5` |

Modificadores especiais:

| Modificador | Custo proposto |
|---|---:|
| Atributo primário `+1` | `5` pontos |
| Chance de Crítico `+1%` | `2` pontos |
| Dano Crítico `+5%` | `2` pontos |
| Velocidade `+1` | `3` pontos |

Esses custos são provisórios e devem ser calibrados com simulações.

## 10. Penalidades e recuperação de pontos

Modificadores negativos devolvem orçamento de forma simétrica.

```text
Custo Líquido =
Custo dos bônus positivos
- pontos recuperados pelas penalidades
```

O item é válido quando:

```text
Custo Líquido ≤ Pontos Máximos
```

### Exemplo solicitado

```text
Orçamento-base: 7
DEF Física +7: 7 pontos
DEF Mágica +2: 2 pontos
Esquiva -2: recupera 2 pontos

Pontos positivos: 9
Recuperação negativa: 2
Custo líquido: 7
```

O jogador pode priorizar defesa em troca de mobilidade.

Para modificadores com custo especial, a penalidade recupera o mesmo custo. Exemplo: `Destreza -1` recupera os mesmos `5` pontos que `Destreza +1` consumiria.

### 10.1 Coerência

Penalidades devem ser mecanicamente coerentes com o item:

- armadura pesada pode reduzir Esquiva, Iniciativa, Furtividade ou Velocidade;
- arma pesada pode reduzir Precisão, Iniciativa ou Esquiva;
- foco arcano pode trocar capacidade física por poder mágico;
- item amaldiçoado pode justificar trocas incomuns.

Ainda não foi definido um limite global de recuperação negativa. Esse ponto permanece pendente de testes para evitar abuso com atributos irrelevantes ao usuário do item.

## 11. Estrutura universal de modificadores

Todo equipamento declara todos os campos:

```json
{
  "primaryModifiers": {
    "strength": 0,
    "dexterity": 0,
    "intelligence": 0,
    "vitality": 0
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
    "speed": 0,
    "carryCapacity": 0,
    "perception": 0,
    "stealth": 0
  }
}
```

## 12. Identidade por categoria

### Adaga

Prioriza ATK Físico, Precisão Física, crítico, Destreza, Iniciativa e Furtividade.

### Espada curta

Prioriza ATK Físico, Precisão Física, pequena DEF Física e possível Defesa Crítica.

### Espada larga

Prioriza ATK Físico elevado e Força; pode penalizar Precisão, Iniciativa ou Esquiva.

### Cajado

Prioriza ATK Mágico, Mana, Inteligência, Precisão Mágica e DEF Mágica. Ainda possui ATK Físico, mesmo que baixo ou zero.

### Escudo

Prioriza DEF Física, DEF Mágica e Defesa Crítica.

### Roupa de Corpo

Fornece aparência e bônus mínimos ou nulos. Pode favorecer Furtividade, Percepção, Mana, Vigor ou proteção leve.

### Peitoral pesado

Prioriza DEF Física, Vida, Vitalidade e Defesa Crítica; pode penalizar Esquiva, Iniciativa, Velocidade e Furtividade.

### Botas

Podem favorecer DEFs, Destreza, Esquiva, Iniciativa, Velocidade e Furtividade.

## 13. Peso

A qualidade não altera automaticamente o peso.

```text
Peso = função de material + tamanho + estrutura + efeitos explícitos
```

Exemplo:

```text
Adaga de ferro inferior: 0,8 kg
Adaga de ferro comum: 0,8 kg
Adaga de ferro rara: 0,8 kg
```

O peso só muda por razão concreta, como outro material, tamanho, reforço, estrutura oca ou encantamento de redução de peso.

## 14. Preço

Todo item possui obrigatoriamente:

```text
buyPrice
sellPrice
```

### 14.1 Compra

```text
Preço de Compra =
preço-base da categoria
× fator de nível
× multiplicador de preço da qualidade
× multiplicador de material
× multiplicador de fabricação
× multiplicador de mercado
```

Fator de nível proposto:

```text
1 + ((nível - 1) × 0,20)
```

Multiplicadores de preço propostos:

| Qualidade | Multiplicador de preço |
|---|---:|
| Inferior | `0,50` |
| Comum | `1,00` |
| Raro | `2,00` |
| Épico | `4,00` |
| Lendário | `8,00` |

O preço cresce mais rápido que o poder porque também representa escassez e dificuldade de fabricação.

### 14.2 Venda

```text
sellPrice =
arredondar_para_baixo(buyPrice × 0,40)
```

O valor final de uma transação pode ser alterado por conservação, reputação, negociação, mercado e relação com o comerciante.

## 15. Exemplo completo resumido

```json
{
  "code": "iron-dagger",
  "name": "Adaga de Ferro",
  "equipmentType": "WEAPON",
  "weaponType": "DAGGER",
  "level": 1,
  "quality": "COMMON",
  "allowedSlots": ["MAIN_HAND", "OFF_HAND"],
  "handedness": "ONE_HANDED",
  "occupiedSlots": 1,
  "weight": 0.8,
  "requirements": {
    "level": 1,
    "strength": 0,
    "dexterity": 5,
    "intelligence": 0,
    "vitality": 0
  },
  "primaryModifiers": {
    "strength": 0,
    "dexterity": 0,
    "intelligence": 0,
    "vitality": 0
  },
  "secondaryModifiers": {
    "maxHealth": 0,
    "maxMana": 0,
    "maxStamina": 0,
    "physicalAttack": 3,
    "magicalAttack": 0,
    "physicalDefense": 0,
    "magicalDefense": 0,
    "physicalAccuracy": 1,
    "magicalAccuracy": 0,
    "evasion": 0,
    "criticalChance": 0,
    "criticalDamage": 0,
    "criticalDefense": 0,
    "physicalResistance": 0,
    "mentalResistance": 0,
    "initiative": 1,
    "speed": 0,
    "carryCapacity": 0,
    "perception": 0,
    "stealth": 0
  },
  "powerBudget": {
    "maximumPoints": 5,
    "positiveCost": 5,
    "negativeRefund": 0,
    "netCost": 5
  },
  "buyPrice": 40,
  "sellPrice": 16,
  "effects": [],
  "tags": ["LIGHT_WEAPON", "FINESSE", "DUAL_WIELD_ALLOWED"]
}
```

## 16. Validações do backend

O backend deve validar:

- estrutura completa;
- nível e qualidade;
- slots permitidos e ocupados;
- requisitos;
- orçamento positivo, negativo e líquido;
- coerência de categoria;
- peso;
- compra e venda;
- incompatibilidades de mãos;
- limites dos atributos;
- versão das regras utilizada.

Erros devem informar valor esperado, valor recebido e ações de recuperação.

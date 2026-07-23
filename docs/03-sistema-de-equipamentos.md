# Sistema de Equipamentos

**Versão da proposta:** `equipment-v0.3`  
**Status:** em validação

## 1. Objetivo

Definir uma estrutura universal para qualquer item equipável. Todo equipamento utiliza os mesmos campos de modificadores, ainda que a maioria esteja em `0`.

O GPT propõe itens coerentes. O backend recalcula orçamento, valida slots, preços-base, peso e modificadores antes de persistir.

## 2. Princípios

1. Todo item equipável declara todos os modificadores.
2. Campos não utilizados permanecem em `0`.
3. Equipamentos podem alterar atributos primários e secundários.
4. Nível e qualidade determinam o orçamento de poder.
5. Penalidades negativas recuperam pontos para distribuição positiva.
6. O custo líquido não pode ultrapassar o orçamento máximo.
7. Todo item comercializável possui moeda e preços-base.
8. O preço final pertence ao sistema de economia e comércio.
9. A qualidade aumenta o preço, mas não altera automaticamente o peso.
10. O peso depende de material, tamanho, estrutura e efeitos explícitos.
11. Equipamentos de duas mãos ocupam os dois slots de mão.
12. Armas e focos podem gerar ações completas com alcance e tempo próprios.

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

### Corpo — `BODY`

Representa roupa-base, aparência e proteção leve:

- roupa comum;
- traje de aventureiro;
- túnica;
- uniforme;
- vestido;
- vestes de viagem;
- traje cerimonial.

Pode não conceder bônus ou conceder modificadores mínimos. Permanece equipado sob o item de Peito.

### Peito — `CHEST`

Representa proteção estrutural usada sobre a roupa:

- cota de malha;
- couraça;
- peitoral de couro;
- armadura de placas;
- manto reforçado;
- veste de batalha encantada.

`BODY` e `CHEST` são independentes.

### Mãos — `HANDS`

Luvas, manoplas e braçadeiras. Não deve ser confundido com `MAIN_HAND` e `OFF_HAND`.

## 4. Ocupação das mãos

### Uma mão

```json
{
  "handedness": "ONE_HANDED",
  "allowedSlots": ["MAIN_HAND", "OFF_HAND"],
  "occupiedSlots": 1
}
```

Permite:

- adaga + adaga;
- espada curta + escudo;
- varinha + adaga;
- cajado de uma mão + livro mágico.

### Duas mãos

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

### Versátil

Reservado para evolução futura. Um item `VERSATILE` poderá possuir perfis diferentes para uma ou duas mãos.

## 5. Ações concedidas pelo equipamento

Armas, escudos, focos, grimórios e outros itens podem conceder ações.

```json
{
  "grantedActionCodes": [
    "basic-dagger-attack"
  ]
}
```

A definição completa da ação pertence a `07-sistema-de-acoes-habilidades-e-magias.md`.

O equipamento não deve depender apenas de um valor narrativo de dano. A ação concedida declara:

- alcance em metros;
- preparação e recuperação;
- escala de dano;
- precisão;
- custos;
- reações;
- efeitos;
- política de interrupção.

## 6. Qualidade

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

A qualidade representa fabricação, conservação inicial, pureza, refinamento, encantamento ou técnica especial.

A condição atual do item é um conceito separado.

## 7. Orçamento-base por nível

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

## 8. Multiplicadores de qualidade

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

## 9. Multiplicador de slot

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

Os multiplicadores ainda precisam de simulações.

## 10. Custos de distribuição

### Modificadores básicos

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
| Velocidade de Ação Física | `+1` |
| Velocidade de Conjuração | `+1` |
| Percepção | `+1` |
| Furtividade | `+1` |
| Vida Máxima | `+5` |
| Mana Máxima | `+5` |
| Vigor Máximo | `+5` |
| Capacidade de Carga | `+5` |

### Modificadores especiais

| Modificador | Custo proposto |
|---|---:|
| Atributo primário `+1` | `5` pontos |
| Chance de Crítico `+1%` | `2` pontos |
| Dano Crítico `+5%` | `2` pontos |
| Velocidade de Movimento `+0,1 m/s` | `1` ponto |

Os custos são provisórios.

A Velocidade de Movimento aceita precisão de uma casa decimal. O backend pode usar representação fixa internamente para evitar erros de ponto flutuante.

## 11. Penalidades e recuperação

```text
Custo Líquido =
Custo dos bônus positivos
- pontos recuperados pelas penalidades
```

O item é válido quando:

```text
Custo Líquido ≤ Pontos Máximos
```

Exemplo:

```text
Orçamento-base: 7
DEF Física +7: 7 pontos
DEF Mágica +2: 2 pontos
Esquiva -2: recupera 2 pontos

Pontos positivos: 9
Recuperação negativa: 2
Custo líquido: 7
```

Para modificadores com custo especial, a penalidade recupera o mesmo custo.

Exemplo:

```text
Destreza -1 recupera 5 pontos
```

### Coerência

- armadura pesada pode reduzir Esquiva, Iniciativa, Movimento, Ação Física ou Furtividade;
- arma pesada pode reduzir Precisão, Iniciativa ou Ação Física;
- foco arcano pode trocar capacidade física por ATK Mágico, Mana ou Conjuração;
- item amaldiçoado pode justificar trocas incomuns.

Ainda não existe limite global definitivo de recuperação negativa.

## 12. Estrutura universal de modificadores

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
    "movementSpeed": 0,
    "physicalActionSpeed": 0,
    "castingSpeed": 0,
    "carryCapacity": 0,
    "perception": 0,
    "stealth": 0
  }
}
```

Todos os campos devem existir mesmo quando forem `0`.

## 13. Identidade por categoria

### Adaga

Prioriza ATK Físico, Precisão Física, crítico, Destreza, Iniciativa e Ação Física. Concede ataque de curto alcance e preparação rápida.

### Espada curta

Prioriza ATK Físico, Precisão Física, pequena DEF Física e possível Defesa Crítica.

### Espada larga

Prioriza ATK Físico elevado e Força; pode penalizar Precisão, Iniciativa, Movimento ou Ação Física. Sua ação possui preparação e recuperação maiores.

### Arco

Prioriza Precisão Física e alcance. Seus disparos exigem preparação, munição e linha de visão.

### Cajado

Prioriza ATK Mágico, Mana, Inteligência, Precisão Mágica, Conjuração e DEF Mágica. Ainda pode conceder ataque físico, mesmo que fraco.

### Escudo

Prioriza DEF Física, DEF Mágica, Defesa Crítica e reações defensivas.

### Roupa de Corpo

Fornece aparência e bônus mínimos ou nulos. Pode favorecer Furtividade, Percepção, Mana, Vigor ou Conjuração.

### Peitoral pesado

Prioriza DEF Física, Vida, Vitalidade e Defesa Crítica; pode penalizar Esquiva, Iniciativa, Movimento, Ação Física e Furtividade.

### Botas

Podem favorecer DEFs, Destreza, Esquiva, Iniciativa, Movimento e Furtividade.

## 14. Peso

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

O peso só muda por razão concreta.

O peso total pode reduzir `movementSpeed` e outros valores conforme regras futuras de carga.

## 15. Preços-base

Todo item comercializável possui:

```text
currencyCode
baseBuyPrice
baseSellPrice
```

Código monetário:

```text
CROWN
```

### Preço-base de compra

```text
Preço-base de compra =
preço-base da categoria
× fator de nível
× multiplicador de preço da qualidade
× multiplicador de material
× multiplicador de fabricação
× fator de poder utilizado
```

```text
Fator de nível = 1 + ((nível - 1) × 0,20)
```

| Qualidade | Multiplicador de preço |
|---|---:|
| Inferior | `0,50` |
| Comum | `1,00` |
| Raro | `2,00` |
| Épico | `4,00` |
| Lendário | `8,00` |

```text
fatorPoder = 0,75 + ((pontosUtilizados ÷ pontosMáximos) × 0,25)
```

### Preço-base de venda

```text
baseSellPrice =
arredondar_para_baixo(baseBuyPrice × 0,40)
```

O preço final de transação pertence ao sistema econômico.

## 16. Exemplo resumido

```json
{
  "code": "iron-dagger",
  "name": "Adaga de Ferro",
  "equipmentType": "WEAPON",
  "weaponType": "DAGGER",
  "level": 1,
  "quality": "COMMON",
  "materialCode": "IRON",
  "allowedSlots": ["MAIN_HAND", "OFF_HAND"],
  "handedness": "ONE_HANDED",
  "occupiedSlots": 1,
  "weight": 0.8,
  "grantedActionCodes": ["basic-dagger-attack"],
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
    "movementSpeed": 0,
    "physicalActionSpeed": 1,
    "castingSpeed": 0,
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
  "currencyCode": "CROWN",
  "baseBuyPrice": 40,
  "baseSellPrice": 16,
  "effects": [],
  "tags": ["LIGHT_WEAPON", "FINESSE", "DUAL_WIELD_ALLOWED"]
}
```

## 17. Validações do backend

O backend valida:

- estrutura completa;
- nível e qualidade;
- slots permitidos e ocupados;
- requisitos;
- orçamento positivo, negativo e líquido;
- coerência de categoria;
- modificadores temporais;
- ações concedidas;
- peso;
- preços-base;
- incompatibilidades de mãos;
- limites dos atributos;
- versão das regras.

Erros devem informar valor esperado, recebido e ações de recuperação.

## 18. Pendências

- validar multiplicadores de slot;
- calibrar custos dos modificadores temporais;
- definir limite de recuperação negativa;
- definir carga e penalidade por peso;
- definir equipamentos versáteis;
- calibrar tempos das ações concedidas por cada arma;
- definir durabilidade e reparos;
- validar preços-base junto da economia.
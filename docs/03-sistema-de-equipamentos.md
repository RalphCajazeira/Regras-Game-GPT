# Sistema de Equipamentos

**Versão da proposta:** `equipment-v0.4`  
**Status:** em validação

## 1. Objetivo

Definir uma estrutura universal para qualquer item equipável. Todo equipamento usa os mesmos campos de modificadores, inclusive quando o valor é `0`.

O GPT propõe equipamentos coerentes. O backend valida estrutura, orçamento, slots, atributos, ações concedidas, peso e preços-base.

## 2. Princípios

1. Todo equipamento declara todos os modificadores.
2. Campos não utilizados permanecem em `0`.
3. Equipamentos podem alterar os cinco atributos primários e os secundários.
4. Nível, qualidade e slot determinam o orçamento de poder.
5. Penalidades negativas recuperam orçamento para bônus positivos.
6. O custo líquido não pode ultrapassar o limite.
7. Todo item comercializável possui moeda, preço-base de compra e preço-base de venda.
8. Qualidade aumenta poder e preço, mas não altera automaticamente o peso.
9. Peso depende de material, tamanho, estrutura e efeitos explícitos.
10. Equipamentos podem conceder ações, proficiências ou passivas compatíveis.

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

Roupa-base e aparência: traje de aventureiro, túnica, uniforme, vestido, vestes de viagem ou traje cerimonial. Pode fornecer bônus mínimos ou nulos e permanece sob o item de Peito.

### Peito — `CHEST`

Proteção estrutural: cota de malha, couraça, peitoral de couro, armadura de placas, manto reforçado ou veste de batalha encantada.

`BODY` e `CHEST` são independentes.

### Mãos — `HANDS`

Luvas, manoplas e braçadeiras. Não se confunde com `MAIN_HAND` e `OFF_HAND`.

## 4. Ocupação das mãos

### Uma mão

```json
{
  "handedness": "ONE_HANDED",
  "allowedSlots": ["MAIN_HAND", "OFF_HAND"],
  "occupiedSlots": 1
}
```

Permite adaga + adaga, espada + escudo, varinha + adaga e combinações equivalentes.

### Duas mãos

```json
{
  "handedness": "TWO_HANDED",
  "allowedSlots": ["MAIN_HAND"],
  "occupiedSlots": 2
}
```

O backend marca as duas mãos como ocupadas pela mesma instância.

### Versátil

`VERSATILE` fica reservado para perfis diferentes de uma ou duas mãos.

## 5. Qualidade

```text
INFERIOR
COMMON
RARE
EPIC
LEGENDARY
```

| Qualidade | Multiplicador de poder | Multiplicador de preço proposto |
|---|---:|---:|
| Inferior | `0,60` | `0,50` |
| Comum | `1,00` | `1,00` |
| Raro | `1,40` | `2,00` |
| Épico | `1,80` | `4,00` |
| Lendário | `2,40` | `8,00` |

A qualidade representa fabricação, refinamento, encantamento ou técnica. Condição e conservação são conceitos separados.

## 6. Orçamento-base

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

## 7. Multiplicador de slot

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
    × multiplicador da qualidade
    × multiplicador do slot
  )
)
```

Os multiplicadores ainda precisam de simulação.

## 8. Custos dos modificadores

| Modificador | Benefício por ponto |
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

Custos especiais propostos:

| Modificador | Custo |
|---|---:|
| Atributo primário `+1` | `5` pontos |
| Chance de Crítico `+1%` | `2` pontos |
| Dano Crítico `+5%` | `2` pontos |
| Velocidade de Movimento `+0,5 m/s` | `3` pontos |
| Velocidade de Ação Física `+1` | `2` pontos |
| Velocidade de Conjuração `+1` | `2` pontos |

## 9. Penalidades

```text
Custo Líquido =
custo positivo - recuperação negativa
```

O item é válido quando:

```text
Custo Líquido ≤ Pontos Máximos
```

Exemplo:

```text
Orçamento: 7
DEF Física +7 = 7
DEF Mágica +2 = 2
Esquiva -2 = recupera 2
Custo líquido = 7
```

Custos especiais são simétricos. `Agilidade -1`, por exemplo, recupera o mesmo custo de `Agilidade +1`.

Penalidades precisam ser coerentes:

- armadura pesada pode reduzir Agilidade, Esquiva, movimento, Iniciativa ou Furtividade;
- arma pesada pode reduzir precisão ou velocidade de ação;
- foco arcano pode trocar capacidade física por poder mágico;
- item amaldiçoado pode justificar trocas incomuns.

O limite global de recuperação negativa ainda precisa de testes.

## 10. Estrutura universal de modificadores

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

## 11. Ações, perícias e passivas concedidas

Um equipamento pode fornecer conteúdo mecânico previamente definido:

```json
{
  "grantedActionCodes": ["basic-dagger-attack"],
  "grantedPassiveCodes": ["balanced-grip-1"],
  "grantedProficiencyCodes": []
}
```

A definição do item não cria automaticamente ações arbitrárias. Os códigos precisam existir e ser compatíveis com slots, empunhadura e requisitos.

## 12. Identidade por categoria

### Adaga

Prioriza ATK Físico, Precisão Física, Destreza, velocidade de ação, crítico e Furtividade.

### Espada larga

Prioriza ATK Físico e Força; pode penalizar precisão, Agilidade ou velocidade de ação.

### Arco

Prioriza Destreza, Precisão Física e ações de disparo. Agilidade pode favorecer reposicionamento e frequência por meio dos atributos do ator.

### Cajado

Prioriza ATK Mágico, Mana, Inteligência, Precisão Mágica e DEF Mágica. Pode conceder ações mágicas ou passivas de conjuração.

### Escudo

Prioriza DEF Física, DEF Mágica, Defesa Crítica e ações defensivas.

### Roupa de Corpo

Aparência e bônus mínimos ou nulos. Pode favorecer Percepção, Furtividade, Mana ou Vigor.

### Peitoral pesado

Prioriza DEF Física, Vida, Vitalidade e Defesa Crítica; pode penalizar Agilidade, movimento, velocidade de ação e Furtividade.

### Botas

Podem favorecer DEFs, Agilidade, Esquiva, movimento, Iniciativa e Furtividade.

## 13. Peso

A qualidade não altera automaticamente o peso.

```text
Peso = material + tamanho + estrutura + efeitos explícitos
```

O peso só muda por motivo concreto, como outro material, reforço, estrutura oca ou encantamento.

## 14. Preços-base

Todo item comercializável possui:

```text
currencyCode
baseBuyPrice
baseSellPrice
```

Moeda atual:

```text
CROWN
```

```text
baseBuyPrice =
preço-base da categoria
× fator de nível
× multiplicador de qualidade
× material
× fabricação
× fator de poder utilizado
```

```text
baseSellPrice =
arredondar_para_baixo(baseBuyPrice × 0,40)
```

Mercado, condição, especialidade e negociação pertencem ao sistema de economia.

## 15. Fabricação

A qualidade de um equipamento fabricado deve respeitar o sistema de fabricação:

- receita;
- material;
- ferramenta;
- oficina;
- perícia;
- profissão;
- passivas;
- teto de qualidade.

Uma perícia elevada melhora a chance de qualidade, mas não ignora os limites de receita e materiais.

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
  "primaryModifiers": {
    "strength": 0,
    "agility": 0,
    "dexterity": 0,
    "vitality": 0,
    "intelligence": 0
  },
  "secondaryModifiers": {
    "physicalAttack": 3,
    "physicalAccuracy": 1,
    "physicalActionSpeed": 1
  },
  "grantedActionCodes": ["basic-dagger-attack"],
  "currencyCode": "CROWN",
  "baseBuyPrice": 40,
  "baseSellPrice": 16
}
```

No payload persistido completo, todos os campos de modificadores devem estar presentes, inclusive os zerados.

## 17. Validações do backend

- estrutura completa;
- nível, qualidade e material;
- slots e empunhadura;
- cinco atributos primários;
- orçamento positivo, negativo e líquido;
- ações, passivas e proficiências concedidas;
- coerência de categoria;
- peso;
- preços-base;
- requisitos;
- versão das regras.

Erros informam valor esperado, recebido e ações de recuperação.

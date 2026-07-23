# Sistema de Atributos

**Versão da proposta:** `attributes-v0.2`  
**Status:** em validação  
**Escopo:** personagens, NPCs, criaturas e animais

## 1. Objetivo

Definir uma estrutura única de atributos que possa ser carregada pelo GPT, modificada por equipamentos, habilidades e condições, validada pelo backend e futuramente exibida no frontend.

Este documento define os valores da ficha. A comparação entre eles e as rolagens pertence ao sistema de combate.

## 2. Atributos primários

| Nome | Código | Função principal |
|---|---|---|
| Força | `strength` | poder físico, carga, vigor e resistência estrutural |
| Destreza | `dexterity` | precisão física, esquiva, iniciativa, crítico e furtividade |
| Inteligência | `intelligence` | poder mágico, mana, precisão mágica e resistência mental |
| Vitalidade | `vitality` | vida, vigor e capacidade de suportar dano e condições físicas |

### 2.1 Criação inicial

Proposta atual:

```text
Força: 5
Destreza: 5
Inteligência: 5
Vitalidade: 5
Pontos livres na criação: 16
```

O valor mínimo normal na criação é `5`. Reduções abaixo disso somente podem existir por raça, condição, maldição ou regra explícita.

### 2.2 Progressão

A cada nível adquirido:

```text
+10 pontos de atributos primários
```

Os pontos podem permanecer sem distribuição e devem ser persistidos em:

```text
unspentPrimaryPoints
```

O backend distribui os pontos apenas mediante operação válida. A subida de nível não precisa preencher os atributos automaticamente.

Personagens de jogador seguem esse orçamento. NPCs e criaturas utilizam a mesma estrutura, mas podem receber atributos por arquétipo ou nível de desafio.

## 3. Atributos secundários canônicos

### 3.1 Recursos

| Nome | Código |
|---|---|
| Vida Máxima | `maxHealth` |
| Mana Máxima | `maxMana` |
| Vigor Máximo | `maxStamina` |

Valores atuais persistidos separadamente:

- `currentHealth`
- `currentMana`
- `currentStamina`

### 3.2 Ofensivos

| Nome | Código |
|---|---|
| ATK Físico | `physicalAttack` |
| ATK Mágico | `magicalAttack` |
| Precisão Física | `physicalAccuracy` |
| Precisão Mágica | `magicalAccuracy` |
| Chance de Crítico | `criticalChance` |
| Dano Crítico | `criticalDamage` |

### 3.3 Defensivos

| Nome | Código |
|---|---|
| DEF Física | `physicalDefense` |
| DEF Mágica | `magicalDefense` |
| Esquiva | `evasion` |
| Defesa Crítica | `criticalDefense` |
| Resistência Física | `physicalResistance` |
| Resistência Mental | `mentalResistance` |

A Esquiva é um **rating de oposição**, não uma porcentagem pronta e não gera uma rolagem separada.

A Defesa Crítica é uma base ou modificador para a chance de anulação completa de um ataque que já acertou.

### 3.4 Utilitários

| Nome | Código |
|---|---|
| Iniciativa | `initiative` |
| Velocidade | `speed` |
| Capacidade de Carga | `carryCapacity` |
| Percepção | `perception` |
| Furtividade | `stealth` |

## 4. Fórmulas-base propostas

As fórmulas abaixo devem ser versionadas e ainda precisam de simulações em diferentes níveis.

### 4.1 Recursos

```text
Vida Máxima =
40 + (nível × 6) + (Vitalidade × 5) + modificadores diretos
```

```text
Mana Máxima =
10 + (nível × 2) + (Inteligência × 4) + modificadores diretos
```

```text
Vigor Máximo =
10
+ (nível × 2)
+ (Vitalidade × 2)
+ Força
+ arredondar_para_baixo(Destreza ÷ 2)
+ modificadores diretos
```

### 4.2 Poder ofensivo

```text
ATK Físico Base = Força × 2
```

```text
ATK Mágico Base = Inteligência × 2
```

Ações, armas e habilidades podem declarar escalas adicionais. Uma arma de finesse poderá aproveitar Destreza sem mudar a fórmula universal da ficha.

### 4.3 Precisões e Esquiva

```text
Precisão Física Base =
nível + arredondar_para_baixo(Destreza ÷ 2)
```

```text
Precisão Mágica Base =
nível + arredondar_para_baixo(Inteligência ÷ 2)
```

```text
Esquiva Base =
nível + arredondar_para_baixo(Destreza ÷ 2)
```

Precisão e Esquiva crescem na mesma escala para evitar que todos os ataques cheguem automaticamente ao teto em níveis altos.

### 4.4 Defesas

```text
DEF Física Base =
Vitalidade + arredondar_para_baixo(Força ÷ 2)
```

```text
DEF Mágica Base =
Vitalidade + arredondar_para_baixo(Inteligência ÷ 2)
```

```text
Defesa Crítica Base = 0
```

Defesa Crítica é aumentada principalmente por equipamentos, posturas, barreiras, talentos e efeitos.

### 4.5 Crítico

```text
Chance de Crítico Base =
5 + arredondar_para_baixo(Destreza ÷ 10)
```

```text
Dano Crítico Base = 150%
```

### 4.6 Resistências

```text
Resistência Física Base =
Vitalidade + arredondar_para_baixo(Força ÷ 2)
```

```text
Resistência Mental Base =
Inteligência + arredondar_para_baixo(Vitalidade ÷ 2)
```

As resistências são usadas contra condições e efeitos. Elas não substituem DEF Física ou DEF Mágica na mitigação de dano.

### 4.7 Utilidade

```text
Iniciativa Base = Destreza
```

```text
Velocidade Base =
5 + arredondar_para_baixo(Destreza ÷ 20)
```

```text
Capacidade de Carga Base =
10 + (Força × 5)
```

```text
Percepção Base =
arredondar_para_baixo((Destreza + Inteligência) ÷ 2)
```

```text
Furtividade Base = Destreza
```

## 5. Composição dos valores

### 5.1 Primários

```text
Atributo Primário Final =
base
+ pontos distribuídos
+ bônus permanentes
+ bônus de equipamentos
+ modificadores temporários
```

### 5.2 Secundários

```text
Atributo Secundário Final =
valor derivado dos primários finais
+ modificadores permanentes diretos
+ modificadores de equipamentos
+ modificadores temporários
```

Um bônus em atributo primário recalcula todos os secundários dependentes. Um bônus direto em secundário altera somente aquele valor.

## 6. Limites finais

| Estatística | Limite |
|---|---|
| Vida Máxima | mínimo `1` |
| Mana e Vigor Máximos | mínimo `0` |
| ATKs, DEFs, Precisões e Resistências | mínimo `0` |
| Esquiva e Defesa Crítica | mínimo `0` |
| Velocidade | mínimo `1` |
| Chance de Crítico | `0%` a `95%` |
| Dano Crítico | mínimo `100%` |

Modificadores individuais podem ser negativos, mas o resultado final respeita os limites da estatística.

## 7. Alteração de recursos máximos

Ao aumentar um máximo, o valor atual não é preenchido automaticamente:

```text
50/100 de Vida → máximo aumenta para 120 → 50/120
```

Ao reduzir um máximo:

```text
novoAtual = mínimo(atualAnterior, novoMáximo)
```

## 8. Estrutura canônica resumida

```json
{
  "ruleVersion": "attributes-v0.2",
  "level": 5,
  "unspentPrimaryPoints": 0,
  "primaryAttributes": {
    "strength": { "base": 5, "allocated": 20, "permanent": 0, "equipment": 0, "temporary": 0, "total": 25 },
    "dexterity": { "base": 5, "allocated": 5, "permanent": 0, "equipment": 0, "temporary": 0, "total": 10 },
    "intelligence": { "base": 5, "allocated": 0, "permanent": 0, "equipment": 0, "temporary": 0, "total": 5 },
    "vitality": { "base": 5, "allocated": 21, "permanent": 0, "equipment": 0, "temporary": 0, "total": 26 }
  },
  "resources": {
    "health": { "current": 200, "maximum": 200 },
    "mana": { "current": 40, "maximum": 40 },
    "stamina": { "current": 102, "maximum": 102 }
  },
  "secondaryAttributes": {
    "physicalAttack": 50,
    "magicalAttack": 10,
    "physicalAccuracy": 10,
    "magicalAccuracy": 7,
    "physicalDefense": 38,
    "magicalDefense": 28,
    "evasion": 10,
    "criticalChance": 6,
    "criticalDamage": 150,
    "criticalDefense": 0,
    "physicalResistance": 38,
    "mentalResistance": 18,
    "initiative": 10,
    "speed": 5,
    "carryCapacity": 135,
    "perception": 7,
    "stealth": 10
  }
}
```

## 9. Responsabilidades

O backend deve persistir os primários, recursos atuais, pontos não distribuídos e versões de regra; recalcular os secundários; validar limites; e retornar a composição dos valores.

O GPT deve carregar a ficha antes de resolver ações, utilizar os valores finais retornados, não inventar atributos e registrar qualquer alteração mecânica na resolução enviada ao backend.

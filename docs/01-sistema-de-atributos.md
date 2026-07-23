# Sistema de Atributos

**Versão da proposta:** `attributes-v0.4`  
**Status:** em validação  
**Escopo:** personagens, NPCs, criaturas e animais

## 1. Objetivo

Definir uma estrutura simples e universal de atributos para o GPT utilizar em combate, exploração e outras resoluções.

A ficha possui poucos atributos primários. Especializações específicas pertencem a atributos secundários, perícias, proficiências, profissões, habilidades, passivas, equipamentos e condições.

## 2. Atributos primários

```text
Força
Agilidade
Destreza
Vitalidade
Inteligência
```

| Nome | Código | Função principal |
|---|---|---|
| Força | `strength` | impacto físico, carga, Vigor e uso de equipamentos pesados |
| Agilidade | `agility` | mobilidade, Esquiva, Iniciativa e velocidade física |
| Destreza | `dexterity` | precisão, controle, ataques localizados e conjuração técnica |
| Vitalidade | `vitality` | Vida, Vigor, defesa e resistência corporal |
| Inteligência | `intelligence` | poder mágico, Mana, conhecimento e resistência mental |

### 2.1 Separação essencial

```text
Agilidade = rapidez e mobilidade
Destreza = precisão e controle
```

Essa separação impede que um único atributo conceda simultaneamente alta Precisão, Esquiva, movimento, frequência de ações, crítico e Furtividade.

## 3. Criação e progressão

Proposta atual:

```text
Força: 5
Agilidade: 5
Destreza: 5
Vitalidade: 5
Inteligência: 5
Pontos livres na criação: 16
```

O valor mínimo normal na criação é `5`. Valores menores exigem raça, condição, maldição ou regra explícita.

A cada nível adquirido:

```text
+10 pontos de atributos primários
```

Pontos não distribuídos são persistidos em:

```text
unspentPrimaryPoints
```

Personagens de jogador seguem o orçamento de progressão. NPCs e criaturas usam a mesma estrutura, mas podem receber valores por arquétipo ou nível de desafio.

## 4. Atributos secundários canônicos

### 4.1 Recursos

| Nome | Código |
|---|---|
| Vida Máxima | `maxHealth` |
| Mana Máxima | `maxMana` |
| Vigor Máximo | `maxStamina` |

Valores atuais:

```text
currentHealth
currentMana
currentStamina
```

### 4.2 Ofensivos

| Nome | Código |
|---|---|
| ATK Físico | `physicalAttack` |
| ATK Mágico | `magicalAttack` |
| Precisão Física | `physicalAccuracy` |
| Precisão Mágica | `magicalAccuracy` |
| Chance de Crítico | `criticalChance` |
| Dano Crítico | `criticalDamage` |

### 4.3 Defensivos

| Nome | Código |
|---|---|
| DEF Física | `physicalDefense` |
| DEF Mágica | `magicalDefense` |
| Esquiva | `evasion` |
| Defesa Crítica | `criticalDefense` |
| Resistência Física | `physicalResistance` |
| Resistência Mental | `mentalResistance` |

Esquiva é um rating de oposição usado na chance única de acerto. Não gera uma segunda rolagem.

### 4.4 Tempo, movimento e utilidade

| Nome | Código | Função |
|---|---|---|
| Iniciativa | `initiative` | disponibilidade inicial e desempates |
| Velocidade de Movimento | `movementSpeed` | metros por segundo |
| Velocidade de Ação Física | `physicalActionSpeed` | redução temporal de ações físicas |
| Velocidade de Conjuração | `castingSpeed` | redução temporal de conjuração |
| Capacidade de Carga | `carryCapacity` | peso sem penalidade |
| Percepção | `perception` | detecção e análise |
| Furtividade | `stealth` | ocultação e movimentação discreta |

## 5. Fórmulas-base propostas

As fórmulas são provisórias e devem ser simuladas nos níveis 1, 5, 10, 20 e 50.

### 5.1 Recursos

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
+ arredondar_para_baixo(Agilidade ÷ 2)
+ modificadores diretos
```

### 5.2 Poder ofensivo

```text
ATK Físico Base = Força × 2
```

```text
ATK Mágico Base = Inteligência × 2
```

Armas e ações podem declarar escalas próprias. Armas de finesse e à distância podem usar Destreza na escala de dano sem mudar o ATK Físico universal.

### 5.3 Precisão e Esquiva

```text
Precisão Física Base =
nível + arredondar_para_baixo(Destreza ÷ 2)
```

```text
Precisão Mágica Base =
nível
+ arredondar_para_baixo(Destreza ÷ 3)
+ arredondar_para_baixo(Inteligência ÷ 4)
```

```text
Esquiva Base =
nível + arredondar_para_baixo(Agilidade ÷ 2)
```

### 5.4 Defesas

```text
DEF Física Base =
Vitalidade + arredondar_para_baixo(Força ÷ 2)
```

```text
DEF Mágica Base =
Vitalidade + arredondar_para_baixo(Inteligência ÷ 2)
```

```text
Defesa Crítica Base =
arredondar_para_baixo(Vitalidade ÷ 25)
```

Equipamentos, escudos, posturas, barreiras, talentos e passivas são as principais fontes adicionais de Defesa Crítica.

### 5.5 Crítico

```text
Chance de Crítico Base =
5 + arredondar_para_baixo(Destreza ÷ 20)
```

```text
Dano Crítico Base = 150%
```

O principal aumento de crítico deve vir de ações, equipamentos, perícias e passivas especializadas.

### 5.6 Resistências

```text
Resistência Física Base =
Vitalidade + arredondar_para_baixo(Força ÷ 2)
```

```text
Resistência Mental Base =
Inteligência + arredondar_para_baixo(Vitalidade ÷ 2)
```

### 5.7 Tempo e movimento

```text
Iniciativa Base = Agilidade
```

```text
Velocidade de Movimento Base =
4 + arredondar_para_baixo(Agilidade ÷ 25)
```

Unidade: metros por segundo.

```text
Velocidade de Ação Física Base =
arredondar_para_baixo(Agilidade ÷ 5)
+ arredondar_para_baixo(Destreza ÷ 10)
```

```text
Velocidade de Conjuração Base =
arredondar_para_baixo(Destreza ÷ 5)
+ arredondar_para_baixo(Inteligência ÷ 10)
```

Agilidade faz o ator agir rapidamente. Destreza melhora a execução técnica. Inteligência contribui para domínio e fluidez de magias.

### 5.8 Utilidade

```text
Capacidade de Carga Base =
10 + (Força × 5)
```

```text
Percepção Base =
arredondar_para_baixo((Destreza + Inteligência) ÷ 2)
```

```text
Furtividade Base =
arredondar_para_baixo((Agilidade + Destreza) ÷ 2)
```

## 6. Composição

```text
Atributo Primário Final =
base
+ pontos distribuídos
+ bônus permanentes
+ equipamentos
+ modificadores temporários
```

```text
Atributo Secundário Final =
valor derivado dos primários finais
+ bônus permanentes diretos
+ equipamentos
+ perícias ou passivas aplicáveis
+ modificadores temporários
```

Um bônus primário recalcula todos os secundários dependentes. Um bônus direto altera somente o secundário indicado.

## 7. Limites finais

| Estatística | Limite |
|---|---|
| Vida Máxima | mínimo `1` |
| Mana e Vigor Máximos | mínimo `0` |
| ATKs, DEFs, Precisões e Resistências | mínimo `0` |
| Esquiva e Defesa Crítica | mínimo `0` |
| Velocidade de Movimento | mínimo `0,5 m/s` |
| Velocidades de Ação e Conjuração | mínimo `0` |
| Chance de Crítico | `0%` a `95%` |
| Dano Crítico | mínimo `100%` |

Modificadores individuais podem ser negativos, mas o resultado final respeita os limites.

## 8. Alteração de recursos máximos

Ao aumentar um máximo, o valor atual não é preenchido automaticamente:

```text
50/100 de Vida → máximo 120 → 50/120
```

Ao reduzir:

```text
novoAtual = mínimo(atualAnterior, novoMáximo)
```

## 9. Estrutura canônica resumida

Personagem de nível 5:

```text
25 pontos-base
+ 16 pontos livres da criação
+ 40 pontos das quatro subidas
= 81 pontos primários totais
```

```json
{
  "ruleVersion": "attributes-v0.4",
  "level": 5,
  "unspentPrimaryPoints": 0,
  "primaryAttributes": {
    "strength": { "base": 5, "allocated": 15, "total": 20 },
    "agility": { "base": 5, "allocated": 10, "total": 15 },
    "dexterity": { "base": 5, "allocated": 8, "total": 13 },
    "vitality": { "base": 5, "allocated": 18, "total": 23 },
    "intelligence": { "base": 5, "allocated": 0, "total": 5 }
  },
  "secondaryAttributes": {
    "physicalAttack": 40,
    "magicalAttack": 10,
    "physicalAccuracy": 11,
    "magicalAccuracy": 10,
    "physicalDefense": 33,
    "magicalDefense": 25,
    "evasion": 12,
    "criticalChance": 5,
    "criticalDamage": 150,
    "criticalDefense": 0,
    "physicalResistance": 33,
    "mentalResistance": 16,
    "initiative": 15,
    "movementSpeed": 4,
    "physicalActionSpeed": 4,
    "castingSpeed": 3,
    "carryCapacity": 110,
    "perception": 9,
    "stealth": 14
  }
}
```

## 10. Responsabilidades

O backend persiste primários, pontos não distribuídos e recursos; recalcula secundários; aplica equipamentos, perícias e passivas; valida limites e retorna composição.

O GPT utiliza os valores finais carregados, não inventa atributos e registra qualquer alteração mecânica na resolução enviada ao backend.

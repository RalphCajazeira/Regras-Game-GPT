# Sistema de Atributos

**Versão da proposta:** `attributes-v0.5`  
**Status:** em validação  
**Escopo:** personagens, NPCs, criaturas e animais

## 1. Objetivo

Definir poucos atributos primários universais e permitir crescimento direto pelo uso, sem abandonar um orçamento previsível por nível.

Especializações específicas pertencem a atributos secundários, perícias, proficiências, habilidades, magias, passivas, equipamentos e condições.

A progressão detalhada está em:

`docs/19-progressao-niveis-experiencia-treinamento-e-dominio.md`

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
| Força | `strength` | impacto físico, carga, Vigor e equipamentos pesados |
| Agilidade | `agility` | mobilidade, Esquiva, Iniciativa e velocidade física |
| Destreza | `dexterity` | precisão, controle, ataques localizados e conjuração técnica |
| Vitalidade | `vitality` | Vida, Vigor, defesa, resistência e adaptação ao esforço |
| Inteligência | `intelligence` | poder mágico, Mana, conhecimento e resistência mental |

### Separação essencial

```text
Agilidade = rapidez e mobilidade
Destreza = precisão e controle
```

## 3. Criação

Proposta atual:

```text
Força: 5
Agilidade: 5
Destreza: 5
Vitalidade: 5
Inteligência: 5
Pontos livres na criação: 16
```

O mínimo normal na criação é `5`. Valores menores exigem espécie, condição, maldição ou regra explícita.

Os `16` pontos iniciais permitem definir identidade antes do início do treinamento orgânico.

## 4. Crescimento primário

A cada nível adquirido:

```text
+10 pontos de capacidade de crescimento primário
```

Esses pontos não precisam ser distribuídos manualmente.

```text
subida de nível
→ aumenta primaryGrowthCapacity
→ ações geram attributeTrainingXp
→ atributo que atinge o limiar consome capacidade
→ atributo aumenta
```

Campos:

```text
primaryGrowthCapacityEarned
primaryGrowthCapacitySpent
primaryGrowthCapacityAvailable
attributeTrainingXp
```

XP de treinamento pode permanecer acumulada quando não houver capacidade disponível.

### 4.1 Materialização de NPCs

NPCs e criaturas usam a mesma estrutura.

Ao materializar um ator de nível definido, o backend valida:

- orçamento-base;
- capacidade liberada pelo nível;
- capacidade já consumida;
- atributos;
- reservas intencionais;
- arquétipo;
- espécie;
- nível de desafio.

Um NPC nível 5 não pode receber atributos incompatíveis com a capacidade de seu nível sem exceção explícita.

### 4.2 Crescimento por uso

Ações declaram pesos de treinamento.

Exemplo:

```json
{
  "attributeTrainingProfile": [
    { "attribute": "intelligence", "weight": 0.60 },
    { "attribute": "dexterity", "weight": 0.30 },
    { "attribute": "vitality", "weight": 0.10 }
  ]
}
```

Uso válido, treino livre, alvo estático, sparring e combate real podem gerar XP de atributo.

## 5. Atributos secundários canônicos

### Recursos

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

Cansaço é persistido separadamente e pode reduzir o Vigor máximo efetivo.

### Ofensivos

| Nome | Código |
|---|---|
| ATK Físico | `physicalAttack` |
| ATK Mágico | `magicalAttack` |
| Precisão Física | `physicalAccuracy` |
| Precisão Mágica | `magicalAccuracy` |
| Chance de Crítico | `criticalChance` |
| Dano Crítico | `criticalDamage` |

### Defensivos

| Nome | Código |
|---|---|
| DEF Física | `physicalDefense` |
| DEF Mágica | `magicalDefense` |
| Esquiva | `evasion` |
| Defesa Crítica | `criticalDefense` |
| Resistência Física | `physicalResistance` |
| Resistência Mental | `mentalResistance` |

Esquiva é rating de oposição usado na chance única de acerto. Não gera segunda rolagem.

### Tempo, movimento e utilidade

| Nome | Código | Função |
|---|---|---|
| Iniciativa | `initiative` | disponibilidade inicial e desempates |
| Velocidade de Movimento | `movementSpeed` | metros por segundo |
| Velocidade de Ação Física | `physicalActionSpeed` | redução temporal de ações físicas |
| Velocidade de Conjuração | `castingSpeed` | redução temporal de conjuração |
| Capacidade de Carga | `carryCapacity` | peso sem penalidade |
| Percepção | `perception` | detecção e análise |
| Furtividade | `stealth` | ocultação e movimentação discreta |

## 6. Fórmulas-base propostas

As fórmulas são provisórias e devem ser simuladas nos níveis 1, 5, 10, 20 e 50.

### Recursos

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

```text
Vigor Máximo Efetivo =
Vigor Máximo
× modificador de Cansaço
× condições aplicáveis
```

### Poder ofensivo

```text
ATK Físico Base = Força × 2
```

```text
ATK Mágico Base = Inteligência × 2
```

Armas e ações podem declarar escalas próprias.

### Precisão e Esquiva

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

### Defesas

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

### Crítico

```text
Chance de Crítico Base =
5 + arredondar_para_baixo(Destreza ÷ 20)
```

```text
Dano Crítico Base = 150%
```

### Resistências

```text
Resistência Física Base =
Vitalidade + arredondar_para_baixo(Força ÷ 2)
```

```text
Resistência Mental Base =
Inteligência + arredondar_para_baixo(Vitalidade ÷ 2)
```

### Tempo e movimento

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

### Utilidade

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

## 7. Composição

```text
Atributo Primário Final =
base
+ crescimento permanente por uso
+ bônus permanentes
+ equipamentos
+ modificadores temporários
```

```text
Atributo Secundário Final =
valor derivado dos primários finais
+ bônus permanentes diretos
+ equipamentos
+ perícias ou passivas
+ modificadores temporários
+ penalidades de Cansaço
```

## 8. Limites finais

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

## 9. Alteração de recursos máximos

Ao aumentar um máximo, o valor atual não é preenchido automaticamente:

```text
50/100 de Vida → máximo 120 → 50/120
```

Ao reduzir:

```text
novoAtual = mínimo(atualAnterior, novoMáximo)
```

## 10. Exemplo de nível 5

```text
25 pontos-base
+ 16 pontos iniciais
+ 40 pontos de capacidade pelas quatro subidas
= 81 pontos primários possíveis
```

A capacidade pode estar:

```text
consumida em atributos
+ disponível
+ aguardando XP de treinamento
```

Exemplo:

```json
{
  "ruleVersion": "attributes-v0.5",
  "level": 5,
  "primaryGrowthCapacityEarned": 56,
  "primaryGrowthCapacitySpent": 54,
  "primaryGrowthCapacityAvailable": 2,
  "primaryAttributes": {
    "strength": { "base": 5, "grown": 15, "total": 20, "trainingXp": 120 },
    "agility": { "base": 5, "grown": 10, "total": 15, "trainingXp": 80 },
    "dexterity": { "base": 5, "grown": 8, "total": 13, "trainingXp": 240 },
    "vitality": { "base": 5, "grown": 18, "total": 23, "trainingXp": 60 },
    "intelligence": { "base": 5, "grown": 3, "total": 8, "trainingXp": 310 }
  }
}
```

O backend deve impedir que `spent > earned`.

## 11. Responsabilidades

### Backend

- persistir atributos e treinamento;
- controlar capacidade de crescimento;
- recalcular secundários;
- aplicar Cansaço, equipamentos, perícias e passivas;
- validar materialização de NPCs;
- impedir crescimento duplicado;
- retornar composição e erros acionáveis.

### GPT

- interpretar atividade e contexto;
- usar valores carregados;
- não conceder pontos diretamente;
- narrar crescimento confirmado.

### Widget

- mostrar capacidade, treinamento e atributos;
- exibir prévias;
- nunca alterar o valor oficial sem resposta do backend.

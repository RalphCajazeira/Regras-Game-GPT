# Sistema de Combate

**Versão da proposta:** `combat-v0.1`  
**Status:** em validação

## 1. Objetivo

Permitir que o GPT resolva um encontro com as fichas carregadas, faça as rolagens, aplique as decisões narrativas e envie ao backend uma resolução consolidada, evitando uma chamada remota para cada ataque ou microetapa.

## 2. Princípio: uma única chance de acerto

Não existe uma rolagem de ataque seguida por outra rolagem de esquiva.

A chance única de acerto já compara os dois envolvidos:

```text
Precisão do atacante
contra
Esquiva ou Resistência do defensor
=
Chance final de acerto
```

A Esquiva é um rating defensivo, não uma porcentagem independente.

Os resultados que não acertam podem ser narrados como erro, esquiva, mudança de trajetória, obstáculo, terreno ruim ou outro acontecimento coerente.

## 3. Limites absolutos

```text
Chance mínima de acerto: 1%
Chance máxima de acerto: 99%
```

Nenhum ataque é impossível e nenhum ataque é garantido.

Uma pessoa nível 1 pode atingir um gatuno nível 50 extremamente ágil em uma situação excepcional de 1%. A narrativa explica o acontecimento depois da rolagem; ela não altera o resultado mecânico.

## 4. Fluxo de resolução

```text
1. Validar ação, alcance, alvo, custos e condições.
2. Selecionar Precisão e oposição aplicáveis.
3. Calcular a chance única de acerto.
4. Rolar 1d100.
5. Se falhar, encerrar o ataque sem dano.
6. Se acertar, testar crítico ofensivo.
7. Calcular o dano bruto ou poder do impacto.
8. Calcular e testar Defesa Crítica.
9. Se a Defesa Crítica ocorrer, anular o dano.
10. Caso contrário, aplicar DEF Física ou DEF Mágica.
11. Aplicar dano, custos e efeitos.
12. Verificar derrota, recuo, rendição ou continuação.
```

## 5. Seleção da oposição

| Tipo de ação | Valor ofensivo | Oposição |
|---|---|---|
| Ataque físico | `physicalAccuracy` | `evasion` |
| Projétil mágico | `magicalAccuracy` | `evasion` |
| Magia mental | `magicalAccuracy` | `mentalResistance` |
| Magia corporal/condição física | `magicalAccuracy` | `physicalResistance` |
| Técnica física de condição | `physicalAccuracy` | `physicalResistance` |

Cada ação deve declarar o modo de resolução. O GPT não escolhe a oposição depois de ver a rolagem.

## 6. Fórmula da chance de acerto

```text
Chance de Acerto =
75
+ Precisão aplicável do atacante
+ modificador de precisão da ação
+ modificadores situacionais do atacante
- oposição aplicável do defensor
- modificadores situacionais do defensor
```

Depois:

```text
Chance Final = limitar(Chance de Acerto, 1, 99)
```

### Exemplo comum

```text
Precisão Física: 30
Esquiva do alvo: 25
Modificador da ação: 0

Chance = 75 + 30 - 25 = 80%
```

### Exemplo extremo

```text
Precisão Física: 55
Esquiva do alvo: 220

Chance calculada = 75 + 55 - 220 = -90%
Chance final = 1%
```

## 7. Rolagem de acerto

O GPT gera um número inteiro de `1` a `100` e o registra.

```text
rolagem ≤ chance final → acerto
rolagem > chance final → falha
```

Exemplo:

```text
Chance: 80%
Rolagem: 37
Resultado: acerto
```

## 8. Crítico ofensivo

O crítico é testado somente depois do acerto.

```text
rolagem crítica ≤ criticalChance → crítico
```

Quando ocorrer:

```text
Dano Bruto Crítico =
Dano Bruto Normal × (criticalDamage ÷ 100)
```

O crítico aumenta o potencial do impacto, mas não impede a Defesa Crítica do alvo.

## 9. Dano bruto

Cada ação deve declarar:

- categoria de dano;
- escala de ATK Físico e/ou ATK Mágico;
- poder fixo;
- penetração;
- custos;
- alcance;
- modificadores de acerto e crítico;
- efeitos adicionais.

Fórmula geral:

```text
Dano Bruto =
(ATK Físico × escala física)
+ (ATK Mágico × escala mágica)
+ poder fixo
+ modificadores situacionais
```

Ações híbridas podem gerar componentes físicos e mágicos separados.

## 10. Defesa Crítica

A Defesa Crítica representa uma defesa perfeita após o ataque conectar:

- aparo perfeito;
- escudo que absorve completamente o impacto;
- armadura que faz o golpe ricochetear;
- barreira que anula a magia;
- proteção natural excepcional.

Quando bem-sucedida:

```text
Dano final = 0
```

### 10.1 Chance proposta

```text
Chance de Defesa Crítica =
criticalDefense
+ arredondar_para_baixo(
    (Defesa correspondente - Poder do Impacto) ÷ 2
  )
+ modificadores defensivos da situação
- modificadores de quebra de guarda do ataque
```

Depois:

```text
Chance Final de Defesa Crítica =
limitar(valor calculado, 0, 99)
```

Onde `Poder do Impacto` é o dano bruto após o crítico ofensivo e antes da mitigação passiva.

Essa fórmula é provisória e precisa ser simulada. Equipamentos, escudos, posturas e barreiras serão as principais fontes de `criticalDefense`.

## 11. Defesa passiva e mitigação

Se a Defesa Crítica falhar, aplica-se a defesa correspondente:

```text
Dano físico → physicalDefense
Dano mágico → magicalDefense
```

Primeiro:

```text
Defesa Efetiva =
máximo(0, Defesa correspondente - Penetração do ataque)
```

Depois:

```text
Dano Final =
máximo(
  1,
  arredondar_para_baixo(
    Dano Bruto × 100 ÷ (100 + Defesa Efetiva)
  )
)
```

Essa mitigação possui retorno decrescente e evita que uma defesa alta anule automaticamente todo ataque. A anulação completa pertence à Defesa Crítica ou a efeitos explícitos.

## 12. Especializações extremas

O sistema permite personagens com riscos claros.

### Muito forte e pouco preciso

- ATK Físico alto;
- Vida e DEF Física altas;
- baixa Precisão;
- baixa Esquiva.

Contra um alvo extremamente ágil, pode ter apenas `1%` de chance de acertar. Se acertar, pode causar morte imediata caso o alvo tenha pouca Vitalidade e defesa.

### Muito ágil e pouco resistente

- alta Precisão e Esquiva;
- alta Iniciativa e crítico;
- pouca Vida e defesa se não investir em Vitalidade.

Evita quase todos os golpes, mas corre risco elevado quando um golpe conecta.

## 13. Ataques localizados

O jogador pode trocar precisão por uma recompensa previamente declarada.

| Alvo | Precisão | Recompensa sugerida |
|---|---:|---:|
| região ampla | `0` | nenhuma |
| região pequena | `-15` | `+15%` crítico ou penetração moderada |
| ponto vulnerável extremo | `-30` | `+30%` crítico ou penetração alta |

A recompensa deve ser escolhida antes da rolagem.

Acertar repetidamente a mesma região pode criar uma condição como `EXPOSED_POINT`, com duração e limite de acúmulo definidos pela ação.

## 14. Ataque furtivo

Proposta inicial para um alvo realmente desprevenido:

```text
Precisão: +20
Chance de Crítico: +25%
Defesa Crítica do alvo: -10%
```

Ataque furtivo não garante acerto nem crítico. Ele modifica a mesma resolução normal.

Percepção, iluminação, barulho, posição e condições podem impedir o estado desprevenido.

## 15. Magias

### 15.1 Projétil direcionado

```text
Precisão Mágica contra Esquiva
```

Se acertar, normalmente utiliza DEF Mágica.

### 15.2 Magia de área

A ação deve declarar o resultado de uma falha na chance de acerto:

- `NO_DAMAGE`;
- `HALF_DAMAGE`;
- dano reduzido específico;
- efeito secundário sem dano principal.

### 15.3 Magia mental

```text
Precisão Mágica contra Resistência Mental
```

### 15.4 Magia corporal

```text
Precisão Mágica contra Resistência Física
```

## 16. Iniciativa e rodadas

A ordem inicial utiliza `initiative`, com modificadores de surpresa, terreno, prontidão e habilidades.

Empates podem ser resolvidos por:

1. maior Destreza;
2. maior nível;
3. rolagem d100.

A estrutura de economia de ações ainda será definida em documento próprio ou em uma revisão deste arquivo.

## 17. Registro estruturado de uma ação

```json
{
  "turn": 3,
  "actorId": "warrior",
  "targetId": "rogue",
  "actionId": "heavy-strike",
  "calculation": {
    "accuracy": 55,
    "oppositionType": "EVASION",
    "opposition": 220,
    "actionAccuracyModifier": 0,
    "situationalModifier": 0,
    "hitChance": 1
  },
  "rolls": {
    "hit": 1,
    "critical": 73,
    "criticalDefense": 84
  },
  "result": {
    "hit": true,
    "critical": false,
    "criticalDefense": false,
    "rawDamage": 410,
    "effectiveDefense": 15,
    "finalDamage": 356,
    "targetHealthBefore": 310,
    "targetHealthAfter": 0,
    "targetDefeated": true
  }
}
```

## 18. Encerramento

O combate pode terminar por:

- vitória;
- derrota;
- recuo;
- rendição;
- interrupção narrativa;
- cancelamento seguro.

Ao terminar, o GPT envia o histórico consolidado ao backend para validação e persistência.

## 19. Experiência

Cada participante derrotável pode possuir:

```text
baseXpReward
```

O GPT informa quais entidades foram derrotadas. O backend confirma a derrota, calcula o XP autoritativo, aplica participação e diferença de nível, verifica subidas e adiciona `10` pontos primários não distribuídos por nível adquirido.

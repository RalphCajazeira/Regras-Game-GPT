# Sistema de Combate

**Versão da proposta:** `combat-v0.2`  
**Status:** em validação

## 1. Objetivo

Permitir que o GPT resolva um encontro com fichas, posições e ações carregadas, faça as rolagens, administre uma linha do tempo contínua e envie ao backend uma resolução consolidada.

O backend não precisa ser chamado para cada ataque, deslocamento ou microetapa.

## 2. Princípios

1. O combate usa posições em metros.
2. O tempo usa uma linha do tempo contínua.
3. Não existe uma rodada rígida em que todos agem exatamente uma vez.
4. Atores rápidos podem ficar disponíveis várias vezes antes de ações lentas terminarem.
5. Toda ação possui preparação e recuperação.
6. Declarar uma ação não significa resolvê-la imediatamente.
7. Existe uma única rolagem de acerto.
8. Esquiva ou Resistência participam da chance de acerto, sem segunda rolagem.
9. Depois do acerto entram crítico ofensivo, Defesa Crítica e mitigação passiva.
10. O GPT resolve localmente e registra tudo para validação final.

## 3. Unidades

### 3.1 Espaço

```text
metros
```

Posições recomendadas:

```json
{
  "position": {
    "xMeters": 12,
    "yMeters": 8
  },
  "occupiedRadiusMeters": 0.5
}
```

Ambientes simples podem usar apenas uma distância linear. Ambientes táticos usam coordenadas.

### 3.2 Tempo

```text
10 ticks = 1 segundo
60 ticks = 6 segundos
```

`combatTime` representa o tempo atual do encontro.

Uma referência de 60 ticks pode ser usada para duração de efeitos e interface, mas não obriga cada participante a agir uma vez nesse período.

## 4. Linha do tempo

Cada ator possui:

```text
nextReadyAt
```

A próxima decisão normal pertence ao ator disponível no menor `nextReadyAt`.

Ações declaradas geram eventos futuros:

```text
resolveAt
nextReadyAt
```

Fórmulas:

```text
resolveAt = combatTime + windupFinal
```

```text
nextReadyAt = combatTime + windupFinal + recoveryFinal
```

O relógio avança sempre para o próximo evento relevante.

## 5. Fila de eventos

A fila pode conter:

```text
ACTION_RESOLVE
ACTOR_READY
MOVEMENT_COMPLETE
CONDITION_TICK
CONDITION_EXPIRE
PROJECTILE_ARRIVAL
REACTION_WINDOW
ENCOUNTER_EVENT
```

O evento com menor tempo é processado primeiro.

Empates são resolvidos por:

1. prioridade explícita do evento;
2. maior `initiative`;
3. maior Destreza;
4. rolagem d100 registrada.

A ordem de prioridade dos tipos ainda precisa de teste.

## 6. Disponibilidade inicial

A Iniciativa determina quando cada ator fica disponível no começo do encontro.

Proposta inicial:

```text
initialReadyAt =
máximo(
  0,
  baseInitialDelayTicks - arredondar_para_baixo(Iniciativa ÷ 5)
)
```

`baseInitialDelayTicks` é definido pelo encontro, normalmente `10`.

Surpresa, prontidão, emboscada e preparação anterior podem modificar esse valor.

Depois da primeira ação, a ordem é controlada pelos tempos das ações, não por uma lista fixa de turnos.

## 7. Ajuste temporal

As ações usam `physicalActionSpeed`, `castingSpeed` ou outro rating declarado.

```text
Tempo Ajustado =
arredondar_para_cima(
  Tempo Base × 100 ÷ (100 + Velocidade Aplicável)
)
```

Depois:

```text
Tempo Final = máximo(Tempo Mínimo, Tempo Ajustado)
```

Nenhum atributo pode reduzir uma ação a zero.

## 8. Movimento

`movementSpeed` é expresso em metros por segundo.

```text
Tempo de Movimento =
arredondar_para_cima(
  distânciaEmMetros ÷ movementSpeed × 10
)
```

O movimento pode ser registrado como evento com posição inicial, posição final, início e conclusão.

### 8.1 Revalidação durante o movimento

O GPT deve considerar:

- obstáculos;
- terreno;
- colisão;
- engajamento;
- mudança de destino;
- interrupção;
- queda ou incapacidade.

### 8.2 Corrida

Proposta inicial:

```text
Velocidade de Corrida = movementSpeed × 1,5
```

A corrida consome Vigor e pode reduzir precisão ou aumentar risco quando a manobra ou o terreno justificarem.

### 8.3 Vantagem legítima de mobilidade

Um arqueiro rápido em campo aberto pode disparar e recuar contra um inimigo lento.

O GPT não deve inventar quedas ou escorregões apenas para permitir que o perseguidor alcance o arqueiro.

A resposta do inimigo depende de recursos reais:

- corrida;
- investida;
- ataque à distância;
- cobertura;
- cerco;
- terreno;
- múltiplos participantes;
- consumo de Vigor;
- munição;
- saída limitada da área.

## 9. Movimento e ataque

Movimento e ataque são ações separadas por padrão.

Se o alvo estiver fora do alcance:

```text
1. mover até uma posição válida;
2. ficar disponível novamente;
3. iniciar o ataque.
```

Uma ação composta, como investida, pode combinar movimento e ataque se sua definição declarar isso.

## 10. Engajamento

Atores dentro da faixa de combate próximo podem ficar em estado:

```text
ENGAGED
```

O alcance exato depende das ações e armas envolvidas.

Sair do engajamento pode usar:

### Desengajamento cuidadoso

- maior custo temporal;
- evita reação normal do oponente.

### Recuo imediato

- menor custo temporal;
- pode abrir janela de reação.

As fórmulas exatas ainda precisam ser calibradas.

## 11. Reações

Uma reação pode ocorrer antes de o ator ficar normalmente disponível.

Exemplos:

- ataque de oportunidade;
- aparo;
- contra-ataque;
- interrupção de magia;
- proteção de aliado;
- contra-feitiço.

Reações geram dívida temporal:

```text
nextReadyAt += timeDebtTicks
```

Uma reação não concede uma ação gratuita sem consequência futura.

## 12. Preparação e interrupção

Durante o `windup`, outros eventos podem acontecer.

Uma ação pode falhar ou mudar se:

- o ator for derrotado;
- o ator for incapacitado;
- o alvo sair do alcance;
- a linha de visão for perdida;
- o equipamento necessário deixar de estar disponível;
- a conjuração for interrompida;
- o custo deixar de ser válido;
- o ambiente mudar.

No momento de `ACTION_RESOLVE`, o GPT revalida os requisitos.

Magias e habilidades declaram a política de interrupção no sistema de ações.

## 13. Princípio: uma única chance de acerto

Não existe uma rolagem de ataque seguida por outra rolagem de esquiva.

```text
Precisão do atacante
contra
Esquiva ou Resistência do defensor
=
Chance final de acerto
```

A Esquiva é um rating defensivo, não uma porcentagem independente.

Os resultados que não acertam podem ser narrados como erro, esquiva, mudança de trajetória, obstáculo ou interferência coerente.

## 14. Limites absolutos de acerto

```text
Chance mínima de acerto: 1%
Chance máxima de acerto: 99%
```

Nenhum ataque é impossível e nenhum ataque é garantido.

Uma pessoa nível 1 pode atingir um gatuno nível 50 extremamente ágil em uma situação excepcional de 1%. A narrativa explica o resultado depois da rolagem.

## 15. Fluxo ofensivo

```text
1. A ação é declarada e agendada.
2. Custos de início são aplicados.
3. Eventos intermediários são processados.
4. No resolveAt, revalidar ator, alvo, alcance e linha de visão.
5. Selecionar Precisão e oposição aplicáveis.
6. Calcular a chance única de acerto.
7. Rolar 1d100.
8. Se falhar, resolver a política de falha da ação.
9. Se acertar, testar crítico ofensivo.
10. Calcular dano bruto ou poder do impacto.
11. Calcular e testar Defesa Crítica.
12. Se ocorrer, anular o dano daquele componente.
13. Caso contrário, aplicar defesa passiva.
14. Aplicar dano, cura, custos restantes e efeitos.
15. Verificar derrota, interrupção e novos eventos.
16. Manter a recuperação até nextReadyAt.
```

## 16. Seleção da oposição

| Tipo de ação | Valor ofensivo | Oposição |
|---|---|---|
| Ataque físico | `physicalAccuracy` | `evasion` |
| Projétil mágico | `magicalAccuracy` | `evasion` |
| Magia mental | `magicalAccuracy` | `mentalResistance` |
| Magia corporal ou condição física | `magicalAccuracy` | `physicalResistance` |
| Técnica física de condição | `physicalAccuracy` | `physicalResistance` |

Cada ação declara o modo de resolução. O GPT não escolhe a oposição depois de ver a rolagem.

## 17. Fórmula da chance de acerto

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

Exemplo:

```text
Precisão Física: 30
Esquiva: 25
Chance = 75 + 30 - 25 = 80%
```

Exemplo extremo:

```text
Precisão Física: 55
Esquiva: 220
Chance calculada = -90%
Chance final = 1%
```

## 18. Rolagem de acerto

O GPT gera e registra um inteiro de `1` a `100`.

```text
rolagem ≤ chance final → acerto
rolagem > chance final → falha
```

## 19. Crítico ofensivo

O crítico é testado somente depois do acerto.

```text
rolagem crítica ≤ criticalChance → crítico
```

```text
Dano Bruto Crítico =
Dano Bruto Normal × (criticalDamage ÷ 100)
```

O crítico não impede a Defesa Crítica.

## 20. Dano bruto

Cada perfil da ação declara:

- tipo de dano;
- atributo de escala;
- percentual de escala;
- poder fixo;
- defesa correspondente;
- penetração;
- permissão de crítico.

Fórmula geral:

```text
Dano Bruto =
(ATK Físico × escala física)
+ (ATK Mágico × escala mágica)
+ poder fixo
+ modificadores situacionais
```

Ações híbridas podem possuir componentes separados.

## 21. Defesa Crítica

A Defesa Crítica representa anulação perfeita após o ataque conectar.

Exemplos:

- aparo perfeito;
- escudo que absorve o impacto;
- armadura que faz o golpe ricochetear;
- barreira que anula a magia;
- proteção natural excepcional.

Quando bem-sucedida:

```text
Dano daquele componente = 0
```

### 21.1 Chance proposta

```text
Chance de Defesa Crítica =
criticalDefense
+ arredondar_para_baixo(
    (Defesa correspondente - Poder do Impacto) ÷ 2
  )
+ modificadores defensivos
- modificadores de quebra de guarda
```

```text
Chance Final = limitar(valor calculado, 0, 99)
```

`Poder do Impacto` é o dano bruto depois do crítico e antes da mitigação.

Essa fórmula ainda precisa de simulações.

## 22. Defesa passiva e mitigação

```text
Dano físico → physicalDefense
Dano mágico → magicalDefense
```

```text
Defesa Efetiva =
máximo(0, Defesa correspondente - Penetração)
```

```text
Dano Final =
máximo(
  1,
  arredondar_para_baixo(
    Dano Bruto × 100 ÷ (100 + Defesa Efetiva)
  )
)
```

A anulação completa pertence à Defesa Crítica ou a efeitos explícitos.

## 23. Especializações extremas

### Muito forte e pouco preciso

- ATK Físico alto;
- Vida e DEF Física altas;
- baixa Precisão;
- baixa Esquiva;
- ações físicas podem continuar lentas se Destreza for baixa.

Pode ter apenas 1% de chance contra um alvo ágil, mas causar dano fatal quando acertar.

### Muito ágil e pouco resistente

- alta Precisão e Esquiva;
- alta Iniciativa e `physicalActionSpeed`;
- alta mobilidade;
- pouca Vida e defesa se não investir em Vitalidade.

Pode agir várias vezes antes de um ataque pesado terminar, mas corre risco elevado se for atingido.

### Conjurador especializado

- ATK Mágico, Mana, Precisão Mágica e `castingSpeed` altos;
- pode concluir magias antes de oponentes lentos alcançarem;
- continua vulnerável a interrupção e pressão corporal se não investir em outras áreas.

## 24. Ataques localizados

| Alvo | Precisão | Recompensa sugerida |
|---|---:|---:|
| região ampla | `0` | nenhuma |
| região pequena | `-15` | `+15%` crítico ou penetração moderada |
| ponto vulnerável extremo | `-30` | `+30%` crítico ou penetração alta |

A recompensa é escolhida antes da rolagem.

Acertos repetidos podem criar uma condição como `EXPOSED_POINT`.

## 25. Ataque furtivo

Proposta inicial:

```text
Precisão: +20
Chance de Crítico: +25%
Defesa Crítica do alvo: -10%
```

O ataque furtivo não garante acerto nem crítico.

## 26. Magias

### Projétil direcionado

```text
Precisão Mágica contra Esquiva
```

### Área

A ação declara a política de falha:

```text
NO_DAMAGE
HALF_DAMAGE
REDUCED_DAMAGE
SECONDARY_EFFECT_ONLY
```

### Mental

```text
Precisão Mágica contra Resistência Mental
```

### Corporal

```text
Precisão Mágica contra Resistência Física
```

Magias com `windup` podem ser interrompidas antes de `resolveAt`.

## 27. Múltiplos participantes

Não existe uma sequência obrigatória do tipo jogador, inimigo A, inimigo B.

Exemplo:

| Tempo | Evento |
|---:|---|
| 8 | inimigo rápido conclui aproximação |
| 12 | jogador inicia ataque |
| 17 | inimigo rápido inicia novo golpe |
| 20 | ataque do jogador conecta |
| 23 | golpe do inimigo rápido conecta |
| 30 | inimigo lento conclui aproximação |
| 31 | jogador fica disponível novamente |
| 42 | golpe lento seria concluído |

O jogador pode contra-atacar o inimigo rápido e ainda agir contra o lento antes da conclusão da ação pesada.

## 28. Ações simultâneas

Duas ações podem possuir o mesmo `resolveAt`.

A regra inicial é resolver pelo desempate de eventos. Uma ação concluída primeiro pode interromper a outra.

Uma política de simultaneidade verdadeira poderá ser adicionada depois para casos especiais.

## 29. Registro estruturado

```json
{
  "sequence": 14,
  "combatTime": 120,
  "eventType": "ACTION_RESOLVE",
  "actorId": "warrior",
  "targetId": "rogue",
  "actionId": "heavy-strike",
  "timing": {
    "startedAt": 98,
    "resolveAt": 120,
    "nextReadyAt": 132,
    "windupTicks": 22,
    "recoveryTicks": 12
  },
  "positions": {
    "actor": { "xMeters": 4, "yMeters": 2 },
    "target": { "xMeters": 5.2, "yMeters": 2 }
  },
  "calculation": {
    "distanceMeters": 1.2,
    "maximumRangeMeters": 2,
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

## 30. Estado local obrigatório

O GPT deve manter:

- `combatTime`;
- `nextReadyAt` de cada ator;
- fila de eventos;
- ações em preparação;
- posições em metros;
- engajamentos;
- recursos;
- condições;
- recargas;
- reações disponíveis;
- participantes derrotados, incapacitados ou em fuga.

## 31. Encerramento

O combate pode terminar por:

- vitória;
- derrota;
- recuo;
- rendição;
- interrupção narrativa;
- cancelamento seguro.

Ao terminar, o GPT envia snapshot-base, fila processada, histórico, estado final e versões de regra ao backend.

## 32. Experiência

Cada participante derrotável pode possuir:

```text
baseXpReward
```

O GPT informa entidades derrotadas. O backend confirma, calcula XP autoritativo, aplica participação e diferença de nível, verifica subidas e adiciona `10` pontos primários não distribuídos por nível.

## 33. Dependências

Este documento depende de:

- `01-sistema-de-atributos.md`;
- `07-sistema-de-acoes-habilidades-e-magias.md`;
- futuro sistema de condições e efeitos.

## 34. Pendências

- calibrar `baseInitialDelayTicks`;
- validar tempos de armas, magias e movimento;
- definir prioridade dos eventos;
- definir concentração e interrupção;
- definir custo de Vigor por corrida;
- definir linha de visão, cobertura e terreno;
- definir engajamento e desengajamento;
- validar reações e dívida temporal;
- testar perseguições, arqueiros, conjuradores e múltiplos inimigos;
- simular níveis 1, 5, 10, 20 e 50.
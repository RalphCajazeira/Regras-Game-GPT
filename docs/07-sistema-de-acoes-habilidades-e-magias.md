# Sistema de Ações, Habilidades e Magias

**Versão da proposta:** `actions-v0.1`  
**Status:** em validação

## 1. Objetivo

Definir uma estrutura universal para tudo que um ator pode executar mecanicamente durante um encontro.

O mesmo modelo deve representar:

- ataques básicos;
- habilidades físicas;
- magias;
- movimento;
- investidas;
- defesas e reações;
- uso de consumíveis;
- cura;
- fuga;
- interações mecânicas.

As ações são executadas pelo GPT sobre uma linha do tempo carregada. O backend valida as definições, fornece o snapshot e confere o histórico consolidado.

## 2. Princípios

1. Distâncias e áreas são expressas em metros.
2. Ações não gastam um número fixo universal de pontos de ação; elas consomem tempo.
3. Cada ação possui preparação e recuperação.
4. Declarar uma ação não significa resolvê-la imediatamente.
5. Movimento e ataque são ações separadas, salvo quando uma ação composta declarar o contrário.
6. Toda ação possui tempo mínimo e nunca chega a custo zero.
7. Alcance, custo, escala, alvo, tempo e efeitos devem existir antes da rolagem.
8. Ações em preparação podem ser interrompidas.
9. No momento da resolução, alcance, linha de visão e estado do ator são verificados novamente.
10. O GPT não inventa propriedades ausentes na definição.

## 3. Categorias

```text
BASIC_ATTACK
PHYSICAL_SKILL
SPELL
MOVEMENT
REACTION
DEFENSE
CONSUMABLE
HEALING
UTILITY
ESCAPE
INTERACTION
```

A categoria organiza a ação, mas não determina sozinha sua fórmula. Cada definição declara os próprios perfis.

## 4. Alvos e alcance

### 4.1 Tipos de alvo

```text
SELF
SINGLE_ACTOR
MULTIPLE_ACTORS
POINT
AREA
DIRECTION
OBJECT
```

### 4.2 Alcance numérico

```json
{
  "targeting": {
    "type": "SINGLE_ACTOR",
    "minimumRangeMeters": 0,
    "maximumRangeMeters": 25,
    "requiresLineOfSight": true
  }
}
```

Valores iniciais de referência, ainda sujeitos a teste:

| Ação ou arma | Alcance |
|---|---:|
| Ataque desarmado | `1 m` |
| Adaga | `1,5 m` |
| Espada curta | `1,8 m` |
| Espada longa | `2 m` |
| Lança | `3 m` |
| Arco curto | até `40 m` |
| Projétil mágico inicial | até `25 m` |

O alcance final pertence à ação disponível, que pode ser gerada por arma, magia, habilidade ou efeito.

## 5. Áreas de efeito

### Círculo

```json
{
  "shape": "CIRCLE",
  "radiusMeters": 4
}
```

### Cone

```json
{
  "shape": "CONE",
  "lengthMeters": 8,
  "angleDegrees": 60
}
```

### Linha

```json
{
  "shape": "LINE",
  "lengthMeters": 15,
  "widthMeters": 1
}
```

### Raio ao redor do usuário

```json
{
  "shape": "SELF_RADIUS",
  "radiusMeters": 3
}
```

O GPT determina os alvos pela posição carregada, geometria da área, cobertura e linha de visão. O log deve registrar a posição de origem e os alvos selecionados.

## 6. Tempo da ação

O combate utiliza unidades de tempo chamadas `ticks`.

```text
10 ticks = 1 segundo
60 ticks = 6 segundos
```

Uma ação possui:

```json
{
  "timing": {
    "speedAttribute": "castingSpeed",
    "baseWindupTicks": 20,
    "baseRecoveryTicks": 8,
    "minimumWindupTicks": 10,
    "minimumRecoveryTicks": 4
  }
}
```

### 6.1 Preparação — `windup`

Tempo entre a declaração e a produção do efeito.

Exemplos:

- erguer e golpear uma arma;
- preparar e disparar uma flecha;
- conjurar uma magia;
- beber uma poção;
- iniciar uma corrida.

### 6.2 Recuperação — `recovery`

Tempo depois da execução até o ator voltar a ficar disponível.

### 6.3 Ajuste por velocidade

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

A velocidade aplicável pode ser:

- `physicalActionSpeed`;
- `castingSpeed`;
- uma velocidade específica declarada pela ação.

## 7. Agendamento

Quando uma ação começa no tempo atual:

```text
resolveAt = combatTime + windupFinal
nextReadyAt = combatTime + windupFinal + recoveryFinal
```

A ação produz seu efeito em `resolveAt`. O ator somente declara outra ação normal ao alcançar `nextReadyAt`.

Se a ação for cancelada ou interrompida, a definição informa como recalcular recuperação, custos e recargas.

## 8. Movimento

### 8.1 Velocidade de movimento

`movementSpeed` é expresso em metros por segundo.

```text
Tempo de Movimento =
arredondar_para_cima(
  distânciaEmMetros ÷ movementSpeed × 10
)
```

Movimento normal não consome Vigor por padrão.

### 8.2 Corrida

Proposta inicial:

```text
Velocidade de Corrida = movementSpeed × 1,5
```

A corrida consome Vigor conforme distância, terreno ou perfil da ação.

### 8.3 Movimento arriscado

Movimento normal em terreno estável não exige rolagem.

Pode existir teste quando houver:

- corrida de costas;
- piso molhado;
- obstáculos;
- escadas;
- terreno difícil;
- exaustão;
- salto;
- manobra acrobática;
- tentativa de abandonar engajamento de forma brusca.

O GPT não deve criar acidentes apenas para compensar uma vantagem legítima de velocidade.

## 9. Movimento e ataque

Se o alvo estiver fora do alcance, uma ação corpo a corpo comum exige:

```text
movimento até o alcance
+
ataque
```

Cada ação ocupa seu próprio período da linha do tempo.

Ações compostas podem combinar ambos:

```json
{
  "code": "charge",
  "category": "PHYSICAL_SKILL",
  "movement": {
    "mode": "MOVE_AND_ATTACK",
    "maximumMeters": 10
  },
  "timing": {
    "speedAttribute": "physicalActionSpeed",
    "baseWindupTicks": 18,
    "baseRecoveryTicks": 10,
    "minimumWindupTicks": 12,
    "minimumRecoveryTicks": 6
  },
  "resourceCosts": {
    "stamina": 15
  }
}
```

## 10. Movimento durante preparação

Toda ação declara se permite movimento durante o `windup`:

```json
{
  "movementDuringWindup": {
    "allowed": false,
    "maximumMovementPercent": 0,
    "accuracyPenalty": 0
  }
}
```

Um disparo móvel poderia permitir movimento parcial com penalidade de precisão.

## 11. Custos

```json
{
  "resourceCosts": {
    "health": 0,
    "mana": 12,
    "stamina": 0,
    "ammo": [
      { "itemCode": "arrow", "quantity": 1 }
    ]
  }
}
```

A definição deve declarar quando o custo é consumido:

```text
ON_START
ON_RESOLVE
ON_SUCCESS
```

Magias interrompidas também precisam declarar quanto do custo é perdido.

## 12. Acerto

```json
{
  "accuracy": {
    "attackerAttribute": "magicalAccuracy",
    "defenderAttribute": "evasion",
    "modifier": 0,
    "canMiss": true
  }
}
```

A fórmula de comparação e a rolagem pertencem ao sistema de combate.

A ação define os atributos usados antes da rolagem.

## 13. Dano

Uma ação pode possuir um ou mais perfis de dano:

```json
{
  "damageProfiles": [
    {
      "damageType": "MAGICAL_FIRE",
      "defenseAttribute": "magicalDefense",
      "scalingAttribute": "magicalAttack",
      "scalingPercent": 120,
      "flatPower": 8,
      "penetration": 0,
      "criticalAllowed": true
    }
  ]
}
```

Ações híbridas podem possuir componentes físicos e mágicos separados.

## 14. Cura

```json
{
  "healingProfiles": [
    {
      "resource": "HEALTH",
      "scalingAttribute": "magicalAttack",
      "scalingPercent": 60,
      "flatPower": 10,
      "canCritical": false
    }
  ]
}
```

A cura não pode ultrapassar o máximo do recurso salvo regra explícita de sobrecura.

## 15. Efeitos e condições

```json
{
  "effects": [
    {
      "effectCode": "BURNING",
      "applicationChance": 30,
      "resistanceAttribute": "physicalResistance",
      "durationTicks": 120,
      "stacks": 1
    }
  ]
}
```

A estrutura completa será definida no sistema de condições e efeitos.

## 16. Interrupção

Ações podem declarar:

```json
{
  "interruption": {
    "interruptible": true,
    "mode": "CONCENTRATION_CHECK",
    "refundPercentOnFailure": 50,
    "recoveryOnFailureTicks": 6
  }
}
```

Modos iniciais:

```text
NOT_INTERRUPTIBLE
CANCEL_ON_HIT
CONCENTRATION_CHECK
DELAY_ON_HIT
CUSTOM
```

A fórmula de concentração ainda precisa ser definida.

## 17. Reações

Reações podem ocorrer antes do `nextReadyAt`, mas nunca são gratuitas.

```json
{
  "category": "REACTION",
  "timing": {
    "timeDebtTicks": 8
  }
}
```

Ao usar a reação:

```text
nextReadyAt += timeDebtTicks
```

Exemplos:

- ataque de oportunidade;
- aparo;
- contra-ataque;
- proteção de aliado;
- contra-feitiço;
- interrupção.

## 18. Recargas e usos

```json
{
  "availability": {
    "cooldownTicks": 120,
    "maximumUsesPerEncounter": 0,
    "maximumCharges": 0
  }
}
```

Valor `0` significa ausência daquele limite.

## 19. Exemplo: ataque básico de adaga

```json
{
  "code": "basic-dagger-attack",
  "name": "Ataque de Adaga",
  "category": "BASIC_ATTACK",
  "targeting": {
    "type": "SINGLE_ACTOR",
    "minimumRangeMeters": 0,
    "maximumRangeMeters": 1.5,
    "requiresLineOfSight": true
  },
  "timing": {
    "speedAttribute": "physicalActionSpeed",
    "baseWindupTicks": 8,
    "baseRecoveryTicks": 6,
    "minimumWindupTicks": 4,
    "minimumRecoveryTicks": 3
  },
  "resourceCosts": {
    "health": 0,
    "mana": 0,
    "stamina": 0
  },
  "accuracy": {
    "attackerAttribute": "physicalAccuracy",
    "defenderAttribute": "evasion",
    "modifier": 0,
    "canMiss": true
  },
  "damageProfiles": [
    {
      "damageType": "PHYSICAL_PIERCING",
      "defenseAttribute": "physicalDefense",
      "scalingAttribute": "physicalAttack",
      "scalingPercent": 100,
      "flatPower": 0,
      "penetration": 0,
      "criticalAllowed": true
    }
  ]
}
```

## 20. Exemplo: disparo de arco

```json
{
  "code": "basic-bow-shot",
  "name": "Disparo de Arco",
  "category": "BASIC_ATTACK",
  "targeting": {
    "type": "SINGLE_ACTOR",
    "minimumRangeMeters": 2,
    "maximumRangeMeters": 40,
    "requiresLineOfSight": true
  },
  "timing": {
    "speedAttribute": "physicalActionSpeed",
    "baseWindupTicks": 12,
    "baseRecoveryTicks": 8,
    "minimumWindupTicks": 7,
    "minimumRecoveryTicks": 4
  },
  "resourceCosts": {
    "health": 0,
    "mana": 0,
    "stamina": 0,
    "ammo": [
      { "itemCode": "arrow", "quantity": 1 }
    ]
  },
  "accuracy": {
    "attackerAttribute": "physicalAccuracy",
    "defenderAttribute": "evasion",
    "modifier": 0,
    "canMiss": true
  },
  "damageProfiles": [
    {
      "damageType": "PHYSICAL_PIERCING",
      "defenseAttribute": "physicalDefense",
      "scalingAttribute": "physicalAttack",
      "scalingPercent": 100,
      "flatPower": 0,
      "penetration": 0,
      "criticalAllowed": true
    }
  ]
}
```

## 21. Exemplo: Flecha de Fogo

```json
{
  "code": "fire-arrow",
  "name": "Flecha de Fogo",
  "category": "SPELL",
  "targeting": {
    "type": "SINGLE_ACTOR",
    "minimumRangeMeters": 0,
    "maximumRangeMeters": 25,
    "requiresLineOfSight": true
  },
  "timing": {
    "speedAttribute": "castingSpeed",
    "baseWindupTicks": 20,
    "baseRecoveryTicks": 8,
    "minimumWindupTicks": 10,
    "minimumRecoveryTicks": 4
  },
  "movementDuringWindup": {
    "allowed": false,
    "maximumMovementPercent": 0,
    "accuracyPenalty": 0
  },
  "resourceCosts": {
    "health": 0,
    "mana": 12,
    "stamina": 0
  },
  "costTiming": "ON_START",
  "interruption": {
    "interruptible": true,
    "mode": "CONCENTRATION_CHECK",
    "refundPercentOnFailure": 50,
    "recoveryOnFailureTicks": 6
  },
  "accuracy": {
    "attackerAttribute": "magicalAccuracy",
    "defenderAttribute": "evasion",
    "modifier": 0,
    "canMiss": true
  },
  "damageProfiles": [
    {
      "damageType": "MAGICAL_FIRE",
      "defenseAttribute": "magicalDefense",
      "scalingAttribute": "magicalAttack",
      "scalingPercent": 120,
      "flatPower": 8,
      "penetration": 0,
      "criticalAllowed": true
    }
  ]
}
```

## 22. Validações do backend

O backend deve validar:

- estrutura da ação;
- atributos referenciados;
- alcance e geometria;
- tempos-base e mínimos;
- custo e momento de consumo;
- escalas de dano e cura;
- recargas e usos;
- política de interrupção;
- movimento permitido;
- efeitos aplicados;
- versão das regras;
- coerência com arma, habilidade ou magia de origem.

## 23. Responsabilidade do GPT

O GPT deve:

- carregar ações completas antes do encontro;
- declarar parâmetros antes de rolar;
- agendar `resolveAt` e `nextReadyAt`;
- revalidar alvo, alcance e linha de visão na resolução;
- aplicar custos conforme `costTiming`;
- registrar interrupções e cancelamentos;
- não combinar movimento e ataque sem perfil autorizado;
- não reduzir tempos abaixo dos mínimos;
- manter a fila de eventos ordenada;
- enviar o histórico consolidado ao backend.

## 24. Pendências

- calibrar tempos de armas, magias e consumíveis;
- definir concentração;
- definir linha de visão e cobertura;
- definir terreno difícil;
- definir custo de Vigor por corrida;
- definir ações simultâneas e empates;
- definir reações e desengajamento;
- definir ações com projéteis em deslocamento;
- validar áreas de efeito;
- criar sistema de condições e efeitos.
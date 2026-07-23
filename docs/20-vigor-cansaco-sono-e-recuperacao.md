# Vigor, Cansaço, Sono e Recuperação

**Versão da proposta:** `fatigue-v0.1`  
**Status:** em validação

## 1. Objetivo

Criar limites naturais para combate, treino, viagem e atividades prolongadas, evitando personagens capazes de agir indefinidamente sem descanso.

O sistema separa:

```text
Vigor   → esforço curto e imediato
Cansaço → desgaste acumulado ao longo do tempo
Sono    → necessidade de repouso prolongado
```

Comida, água, temperatura e doenças serão integradas depois, sem substituir estas camadas.

## 2. Princípios

1. Toda atividade avança o tempo.
2. Ações podem consumir Vigor, gerar Cansaço ou ambos.
3. Mana não substitui Cansaço: conjurar repetidamente também desgasta.
4. Vigor pode recuperar rapidamente; Cansaço exige repouso real.
5. Cansaço reduz desempenho e recuperação antes de causar colapso.
6. Sono insuficiente aumenta Cansaço e risco.
7. Treinamento continua concedendo XP enquanto a ação for executável.
8. O backend resolve tempo, custos, penalidades e recuperação.
9. O widget apenas prevê e apresenta o estado.
10. Os valores são regras de jogo, não simulação médica da vida real.

## 3. Estado canônico

```json
{
  "stamina": {
    "current": 42,
    "maximum": 60,
    "effectiveMaximum": 54,
    "regenerationPerMinute": 8
  },
  "fatigue": {
    "value": 27,
    "stage": "TIRED"
  },
  "sleep": {
    "awakeMinutes": 1080,
    "sleepPressure": 22,
    "lastRestQuality": "ADEQUATE"
  }
}
```

## 4. Vigor

Vigor representa energia imediata para esforço intenso.

Usos comuns:

- ataques físicos;
- corrida;
- natação;
- escalada;
- esquiva;
- bloqueio;
- ações compostas;
- técnicas físicas;
- esforço de carga;
- algumas magias ou habilidades híbridas.

### 4.1 Custos

Cada ação declara seu custo:

```json
{
  "resourceCosts": {
    "stamina": 8,
    "mana": 0
  }
}
```

O custo pode variar por:

- peso do equipamento;
- distância;
- terreno;
- velocidade;
- intensidade;
- técnica;
- condição;
- Cansaço;
- passiva;
- ferimento.

### 4.2 Vigor baixo

Faixas iniciais para simulação:

| Percentual efetivo | Estado | Efeito sugerido |
|---:|---|---|
| `26–100%` | `NORMAL` | sem penalidade por Vigor |
| `11–25%` | `LOW_STAMINA` | recuperação mais lenta e pequena penalidade em esforço intenso |
| `1–10%` | `DEPLETED` | penalidade relevante e ações caras indisponíveis |
| `0%` | `EMPTY` | somente ações permitidas sem custo de Vigor |

A ação não pode começar quando o custo obrigatório exceder o Vigor disponível, salvo regra explícita de esforço além do limite com custo em Vida, Cansaço ou ferimento.

## 5. Recuperação de Vigor

Vigor pode recuperar:

- ao parar atividade intensa;
- caminhando lentamente, quando permitido;
- em postura de descanso;
- fora de combate;
- por magia, item ou passiva.

A recuperação depende de:

```text
regeneração-base
× estágio de Cansaço
× condição física
× postura
× ambiente
× equipamento
```

Cansaço alto reduz o máximo efetivo e a regeneração.

## 6. Cansaço

Cansaço é persistido em escala de `0` a `100`.

```text
0   → totalmente descansado
100 → limite crítico
```

Faixas iniciais:

| Cansaço | Estágio |
|---:|---|
| `0–19` | `RESTED` |
| `20–39` | `TIRED` |
| `40–59` | `FATIGUED` |
| `60–79` | `EXHAUSTED` |
| `80–94` | `CRITICAL` |
| `95–100` | `COLLAPSE_RISK` |

## 7. Efeitos do Cansaço

Valores exatos ainda serão simulados.

### `RESTED`

- sem penalidade;
- recuperação normal;
- eficiência de aprendizado normal.

### `TIRED`

- pequena redução de recuperação;
- pequenas penalidades somente em esforços prolongados.

### `FATIGUED`

- redução de Vigor máximo efetivo;
- penalidade moderada em velocidade, precisão ou concentração;
- recuperação mais lenta;
- maior chance de erro em treino longo.

### `EXHAUSTED`

- penalidades relevantes;
- custo efetivo maior em ações intensas;
- risco de falha e interrupção;
- treino ainda possível, mas menos eficiente e mais perigoso.

### `CRITICAL`

- ações intensas podem exigir teste;
- risco de queda, falha crítica ou incapacidade;
- recuperação de Vigor severamente reduzida.

### `COLLAPSE_RISK`

- verificações periódicas de colapso;
- ação pode ser interrompida;
- ator pode desmaiar ou tornar-se incapaz de continuar.

## 8. Geração de Cansaço

Cansaço aumenta por:

- horas acordado;
- combate;
- treino físico;
- corrida;
- natação;
- viagem intensa;
- carga pesada;
- conjuração repetida;
- concentração prolongada;
- ferimentos;
- frio, calor ou ambiente hostil futuramente;
- fome e sede futuramente.

Cada ação pode declarar:

```json
{
  "exertionProfile": {
    "staminaCost": 8,
    "fatigueCost": 1,
    "fatiguePerMinute": 0,
    "intensity": "HIGH"
  }
}
```

## 9. Exemplos de esforço

### Caminhar

- pouco ou nenhum Vigor;
- Cansaço baixo por tempo e carga.

### Correr

- Vigor por distância ou tempo;
- Cansaço maior;
- risco crescente sob carga ou terreno ruim.

### Nadar

- Vigor contínuo;
- Cansaço elevado;
- falha grave pode gerar risco de afogamento no sistema futuro.

### Ataque físico

- Vigor conforme arma e técnica;
- pequena ou moderada geração de Cansaço;
- armas pesadas podem gerar mais desgaste.

### Esquiva

- Vigor imediato;
- custo maior quando repetida em sequência;
- Cansaço proporcional à intensidade.

### Bloqueio

- Vigor pode depender do impacto recebido;
- golpes pesados podem consumir Vigor adicional.

### Magia

- Mana conforme definição;
- Cansaço mental ou geral por repetição;
- magias complexas podem exigir concentração e também Vigor.

## 10. Tempo acordado e pressão de sono

O sistema utiliza um ciclo de jogo provisório.

```text
0–16 horas acordado  → faixa normal
16–20 horas          → aumento leve de pressão de sono
20–24 horas          → aumento relevante
24–36 horas          → penalidades severas e testes
36–48 horas          → estado crítico
48+ horas            → colapso altamente provável sem regra sobrenatural
```

Essas faixas são valores de balanceamento e serão simuladas.

Não significam que permanecer acordado por determinado período seja seguro na vida real.

### 10.1 Acúmulo sugerido

Após uma janela normal de vigília, cada hora adicional aumenta:

- `sleepPressure`;
- Cansaço;
- penalidades de concentração;
- risco de microssono, falha ou colapso.

## 11. Sono

Tipos iniciais:

```text
NAP
LIGHT_SLEEP
ADEQUATE_SLEEP
RESTORATIVE_SLEEP
INTERRUPTED_SLEEP
UNSAFE_SLEEP
MAGICAL_SLEEP
```

O resultado depende de:

- duração;
- segurança do local;
- conforto;
- interrupções;
- condição física;
- efeitos, doenças ou maldições;
- passivas;
- qualidade de abrigo.

### 11.1 Recuperação

Sono pode:

- reduzir Cansaço;
- reduzir pressão de sono;
- restaurar parte de Vida, Mana e Vigor;
- permitir recuperação de condições;
- avançar o tempo do mundo;
- disparar eventos de acampamento.

Vida, Mana e Vigor não precisam recuperar completamente apenas porque o ator dormiu.

## 12. Descanso sem dormir

Descanso pode reduzir esforço e recuperar Vigor, mas não substitui indefinidamente o sono.

Tipos:

```text
SHORT_REST
SEATED_REST
CAMP_REST
BED_REST
MEDITATIVE_REST
```

O descanso pode diminuir Cansaço lentamente até um limite, mas a pressão de sono continua quando o ator permanece acordado.

## 13. Treinamento e Cansaço

Treino sempre avança o tempo e aplica custos reais.

Exemplo:

```text
40 Mana
Bola de Fogo custa 4
→ até 10 conjurações
→ cada conjuração avança tempo
→ cada conjuração gera Cansaço
→ XP é calculada por uso
→ sessão termina por Mana, Cansaço, tempo ou evento
```

Cansaço pode reduzir eficiência de aprendizado, mas uma ação validamente executada ainda pode conceder XP.

Em Cansaço crítico, o ator pode falhar ou colapsar antes de completar a próxima repetição.

## 14. Treino físico e recuperação

Exemplo:

> Fazer musculação por quatro horas e depois entrar em combate.

Consequências possíveis:

- Vigor parcialmente recuperado, mas máximo efetivo reduzido;
- Cansaço alto;
- penalidade de desempenho;
- risco de ferimento;
- menor eficiência de treino posterior;
- necessidade de descanso ou sono.

O backend não bloqueia a escolha. Ele aplica o estado resultante.

## 15. Condições de parada em atividades longas

```text
STAMINA_BELOW_NEXT_COST
MANA_BELOW_NEXT_COST
HEALTH_AT_OR_BELOW
FATIGUE_AT_OR_ABOVE
SLEEP_PRESSURE_AT_OR_ABOVE
WORLD_TIME_REACHED
DURATION_REACHED
REPETITIONS_REACHED
ACTOR_INCAPACITATED
THREAT_DETECTED
PLAYER_DECISION_REQUIRED
```

## 16. Passagem de tempo

Toda atividade longa deve declarar ou calcular duração.

```text
atividade
→ intervalo de tempo
→ consumo e recuperação
→ atualização de Cansaço
→ atualização de sono
→ eventos do mundo
→ resultado final
```

Viagem, treino, descanso, fabricação e estudo reutilizam `advanceWorldTime` ou comandos equivalentes nos serviços de domínio.

## 17. Resolução em lote

Uma sessão longa não exige uma chamada por minuto.

O backend pode resolver em intervalos, mas deve preservar:

- ordem de eventos;
- custos;
- rolagens importantes;
- condições de parada;
- XP por ação;
- mudança de estado;
- eventos inesperados.

Exemplo de resposta:

```json
{
  "activity": {
    "type": "TRAIN_ACTION_REPETITION",
    "attemptedRepetitions": 10,
    "completedRepetitions": 9,
    "stopReason": "FATIGUE_AT_OR_ABOVE",
    "elapsedMinutes": 18
  },
  "resources": {
    "manaBefore": 40,
    "manaAfter": 4,
    "staminaBefore": 60,
    "staminaAfter": 54,
    "fatigueBefore": 55,
    "fatigueAfter": 70
  },
  "progression": {},
  "events": []
}
```

## 18. Widget

O widget deve mostrar:

- Vigor atual e efetivo;
- Cansaço;
- estágio;
- tempo acordado;
- pressão de sono;
- recuperação estimada;
- custo previsto da atividade;
- duração;
- condição de parada;
- risco de colapso;
- resumo após treino, viagem ou descanso.

Prévia é estimativa. O backend confirma o resultado oficial.

## 19. GPT

O GPT pode interpretar:

- intensidade do treino;
- objetivo;
- duração desejada;
- condição de parada;
- local e contexto;
- intenção de descansar ou dormir;
- risco que o jogador aceita.

O GPT não ignora Cansaço, dano ou tempo apenas para continuar a narrativa.

## 20. Integração futura de comida e água

Sistemas futuros poderão acrescentar:

```text
hunger
thirst
nutrition
hydration
foodQuality
waterSafety
```

Eles afetarão recuperação, Cansaço e desempenho, mas não devem ser exigidos para o primeiro recorte jogável.

## 21. Pontos para simulação

- custo de Vigor por ação;
- regeneração por minuto;
- ganho de Cansaço por esforço;
- penalidades por estágio;
- máximo efetivo de Vigor;
- curva de vigília;
- duração e qualidade do sono;
- recuperação de recursos;
- interação com ferimentos;
- magia e Cansaço mental;
- treino prolongado;
- companheiros e acampamento;
- valores diferentes por espécie ou natureza.

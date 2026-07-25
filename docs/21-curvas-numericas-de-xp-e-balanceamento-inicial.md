# Curvas Numéricas de XP e Balanceamento Inicial

**Versão da proposta:** `progression-curves-v0.1`  
**Status:** calibração inicial para implementação e simulação

## 1. Objetivo

Transformar o sistema conceitual de progressão por uso em fórmulas numéricas implementáveis, auditáveis e configuráveis.

Este documento complementa:

```text
docs/19-progressao-niveis-experiencia-treinamento-e-dominio.md
docs/20-vigor-cansaco-sono-e-recuperacao.md
```

Os valores desta versão são o primeiro baseline. Devem ser implementados como políticas versionadas e simulados antes de serem considerados definitivos.

## 2. Princípios de calibração

1. Toda ação mecanicamente executada pode gerar aprendizagem.
2. Comando rejeitado antes da execução gera `0 XP`.
3. Prática livre concede menos XP que desafio real, mas nunca é anulada apenas por repetição.
4. Falha válida mantém XP de execução e perde ou reduz XP de resultado.
5. Cansaço diminui eficiência, mas uma ação concluída ainda pode ensinar.
6. Atributos evoluem muito mais lentamente que domínio individual.
7. Domínio individual evolui rapidamente nos níveis iniciais e desacelera progressivamente.
8. Perícias e proficiências representam prática ampla e evoluem mais lentamente que uma técnica específica.
9. Profissões evoluem por resultados profissionais completos e possuem a curva mais lenta entre as trilhas de domínio.
10. Nível geral cresce por desenvolvimento total, não somente por mortes ou conclusão de missões.
11. Nenhuma fórmula autoriza ignorar Mana, Vigor, Vida, Cansaço, tempo, itens, risco ou consequências.
12. Toda constante precisa ser carregada por `ruleVersion`.

## 3. Unidade de armazenamento

XP deve ser armazenada em milésimos:

```text
1 XP = 1.000 xpMilli
0,45 XP = 450 xpMilli
```

Campos recomendados:

```text
xpMilli
lifetimeXpMilli
xpToNextLevelMilli
```

Motivo:

- contribuições pequenas de atributos não são perdidas;
- não existe arredondamento para zero por evento;
- o frontend pode exibir somente inteiros ou uma casa decimal;
- o backend mantém precisão determinística.

Arredondamento canônico:

```text
cálculos internos → inteiro em xpMilli
exibição → arredondamento visual
mudança de nível → comparação pelo valor interno
```

## 4. Perfil de aprendizagem da ação

Toda ação, magia, habilidade, atividade ou resolução capaz de ensinar declara:

```json
{
  "learningProfile": {
    "executionBaseXp": 6,
    "resultBaseXp": 4,
    "trackFactors": {
      "mastery": 1.0,
      "skill": 0.4,
      "proficiency": 0.5,
      "profession": 0.25,
      "attributePool": 0.25,
      "actorDevelopment": 0.1
    },
    "attributeWeights": [
      { "attribute": "intelligence", "weight": 0.6 },
      { "attribute": "dexterity", "weight": 0.3 },
      { "attribute": "vitality", "weight": 0.1 }
    ]
  }
}
```

Os fatores não precisam somar `1`. Cada trilha possui curva própria.

Uma ação pode omitir trilhas não aplicáveis.

Exemplos:

- ataque básico normalmente não possui domínio individual e usa proficiência;
- magia nomeada normalmente possui domínio individual e perícia mágica;
- corrida usa atributos e talvez perícia atlética, sem domínio individual;
- fabricação usa perícia, profissão, possível domínio de receita e atributos;
- receber dano usa trilhas de resistência, Vitalidade ou tolerância, não domínio da ação ofensiva.

## 5. XP-base por complexidade

Valores iniciais:

| Classe da execução | `executionBaseXp` | `resultBaseXp` | Exemplos |
|---|---:|---:|---|
| Muito simples | `1` | `0–1` | intervalo de caminhada, postura, repetição elementar |
| Simples | `4` | `2` | soco, ataque básico, esquiva comum |
| Padrão | `6` | `4` | Bola de Fogo inicial, cura padrão, técnica física |
| Avançada | `10` | `6` | magia complexa, manobra difícil, fabricação especializada |
| Excepcional | `16` | `10` | ritual, técnica de alto domínio, fabricação rara |

Atividades contínuas não geram um evento por passo ou segundo. O backend agrega em intervalos canônicos.

Exemplos:

```text
caminhada → evento por intervalo relevante
corrida → evento por trecho ou intervalo
natação → evento por intervalo e mudança de risco
estudo → evento por bloco de tempo
fabricação → eventos por etapa e resultado final
```

## 6. Fórmula canônica por evento

### 6.1 Componente de execução

```text
ExecutionLearning =
executionBaseXp
× contextMultiplier
× difficultyMultiplier
× executionOutcomeMultiplier
× repetitionMultiplier
× fatigueMultiplier
× otherLearningModifiers
```

### 6.2 Componente de resultado

```text
ResultLearning =
resultBaseXp
× contextMultiplier
× difficultyMultiplier
× resultQualityMultiplier
× repetitionMultiplier
× fatigueMultiplier
× otherLearningModifiers
```

### 6.3 Valor bruto

```text
RawLearning = ExecutionLearning + ResultLearning
```

### 6.4 Distribuição

```text
TrackXp = RawLearning × trackFactor
```

Para atributos:

```text
AttributePoolXp = RawLearning × attributePoolFactor
AttributeXp = AttributePoolXp × attributeWeight
```

### 6.5 Piso mínimo

Se a ação foi mecanicamente executada e existe uma trilha primária de prática:

```text
mínimo da trilha primária = 0,10 XP
```

Esse piso não se aplica a:

- comando rejeitado;
- ação não iniciada;
- evento duplicado;
- atividade declarada como não treinável;
- trilha não relacionada ao evento.

## 7. Multiplicador de contexto

Valores iniciais:

| Contexto | Multiplicador |
|---|---:|
| `FREE_PRACTICE` | `0.50` |
| `STATIC_TARGET` | `0.65` |
| `TRAINING_DUMMY` | `0.80` |
| `CONTROLLED_EXERCISE` | `0.75` |
| `STRUCTURED_TRAINING` | `1.00` |
| `SPARRING` | `1.10` |
| `REAL_CHALLENGE` | `1.25` |
| `EXTREME_CHALLENGE` | `1.40` |
| `PROFESSIONAL_PRACTICE` | `1.00` |
| `STUDY` | `0.65` |
| `MEDITATION` | `0.70` |

Regras:

- `FREE_PRACTICE` ensina execução, controle e domínio;
- alvo estático pode ensinar precisão e resultado contra objeto;
- boneco adequado melhora feedback técnico;
- sparring adiciona reação e timing;
- desafio real inclui risco e imprevisibilidade;
- contexto extremo não protege contra morte, captura ou incapacidade;
- estudo e meditação só afetam trilhas que aceitam aprendizagem teórica ou mental.

## 8. Multiplicador de dificuldade relativa

Quando não existe oponente ou desafio comparável:

```text
difficultyMultiplier = 1.00
```

Em combate, sparring ou disputa, o backend calcula:

```text
personalChallengeRatio =
effectiveThreatPresentedToActor
÷ actorEffectiveCapability
```

Faixas iniciais:

| `personalChallengeRatio` | Multiplicador |
|---:|---:|
| `0–0.25` | `0.55` |
| `>0.25–0.50` | `0.70` |
| `>0.50–0.80` | `0.85` |
| `>0.80–1.20` | `1.00` |
| `>1.20–1.50` | `1.15` |
| `>1.50–2.00` | `1.35` |
| `>2.00–3.00` | `1.60` |
| `>3.00` | `1.80` |

O cálculo não utiliza somente nível. Deve considerar:

- atributos efetivos;
- Vida, Mana e Vigor atuais;
- equipamentos;
- ações disponíveis;
- condições;
- quantidade de participantes;
- foco e exposição do ator;
- terreno;
- distância;
- cobertura;
- risco real no momento do evento.

Em grupo, cada participante recebe dificuldade personalizada. Apenas atingir uma vez um chefe enfrentado quase totalmente por aliados não concede o mesmo multiplicador de quem assumiu risco e contribuição real.

## 9. Multiplicadores de execução e resultado

### 9.1 Execução

| Resultado da execução | Multiplicador |
|---|---:|
| Execução limpa | `1.00` |
| Execução excepcional | `1.10` |
| Falha ou erro válido | `0.85` |
| Interrompida após progresso relevante | `0.50` |
| Interrompida ainda no início | `0.25` |
| Não começou | `0.00` |

### 9.2 Resultado

| Qualidade do resultado | Multiplicador |
|---|---:|
| Nenhum resultado mecânico | `0.00` |
| Resultado parcial | `0.50` |
| Resultado normal | `1.00` |
| Resultado elevado | `1.15` |
| Resultado crítico | `1.30` |
| Resultado desperdiçado ou excessivo | `0.50` |

Exemplos:

- errar um ataque mantém XP de execução, mas não recebe XP de dano;
- curar alvo com Vida cheia pode ensinar conjuração, mas não cura útil;
- bloquear parcialmente recebe resultado parcial;
- crítico acrescenta XP de resultado, sem multiplicar infinitamente o evento;
- dano excessivo em alvo já derrotado não gera resultado integral.

## 10. Repetição e retorno gradual

A repetição é calculada por `practiceFingerprint`, não apenas por `actionCode`.

O fingerprint pode incluir:

```text
ator
ação
contexto
alvo ou categoria de alvo
postura
intensidade
ambiente
sessão
objetivo de treino
```

Faixas iniciais para repetição semanticamente idêntica:

| Repetições na janela | Multiplicador |
|---:|---:|
| `1–10` | `1.00` |
| `11–25` | `0.90` |
| `26–50` | `0.80` |
| `51–100` | `0.70` |
| `101–200` | `0.60` |
| `201+` | `0.50` |

Regras:

- o multiplicador nunca cai abaixo de `0.50` apenas por repetição válida;
- combate dinâmico normalmente possui piso de `0.85` porque posição, alvo e risco mudam;
- variar técnica, alvo, distância, intensidade ou contexto pode alterar o fingerprint;
- instrutor ou treino estruturado pode elevar o piso;
- repetição continua consumindo tempo e recursos integralmente;
- o contador pode reduzir após descanso prolongado, sono ou mudança material de contexto;
- baseline inicial: reduzir o contador efetivo em `50%` após `8` horas de afastamento da prática e zerar após `24` horas sem repetição equivalente.

## 11. Eficiência por Cansaço

| Estágio | Multiplicador de aprendizagem |
|---|---:|
| `RESTED` | `1.00` |
| `TIRED` | `0.95` |
| `FATIGUED` | `0.85` |
| `EXHAUSTED` | `0.70` |
| `CRITICAL` | `0.50` |
| `COLLAPSE_RISK` | `0.25` |

O multiplicador só é aplicado se a ação for concluída ou tiver progresso suficiente para XP parcial.

Cansaço também pode reduzir desempenho e impedir a próxima ação antes de sua execução.

## 12. Curva do nível geral

O nível geral começa em `1` e não possui limite rígido nesta versão.

XP necessária do nível atual `L` para o próximo:

```text
ActorXpToNext(L) =
100
+ 50 × (L - 1)
+ 10 × (L - 1)²
```

Tabela:

| Nível atual | XP para o próximo | XP total aproximada acumulada antes do nível |
|---:|---:|---:|
| `1` | `100` | `0` |
| `2` | `160` | `100` |
| `5` | `460` | `840` |
| `10` | `1.360` | `4.740` |
| `20` | `4.660` | `31.540` |
| `30` | `9.960` | `100.340` |
| `50` | `26.560` | `443.940` |

O nível geral recebe XP por uso usando o fator da definição e também por marcos.

### 12.1 Fator padrão por uso

```text
actorDevelopmentFactor = 0.10
```

Uma ação com `RawLearning = 3` concede:

```text
0,30 XP geral
```

### 12.2 Bônus de marco

Valores iniciais:

| Marco | XP geral |
|---|---:|
| Nível de domínio individual | `2 + floor(novoNível ÷ 10)` |
| Nível de perícia | `3 + floor(novoNível ÷ 10)` |
| Nível de proficiência | `3 + floor(novoNível ÷ 10)` |
| Nível de profissão | `5 + floor(novoNível ÷ 10)` |
| Aumento de atributo primário | `10` |

### 12.3 Objetivos e descobertas

| Categoria | XP geral inicial |
|---|---:|
| Objetivo menor | `10` |
| Objetivo padrão | `30` |
| Objetivo importante | `75` |
| Objetivo épico | `200` |
| Marco lendário | `500` |

Missões podem distribuir esses valores entre etapas e conclusão para evitar recompensa concentrada somente no final.

## 13. Curva de domínio individual

Domínio de habilidades e magias utiliza níveis de `1` a `100`.

```text
MasteryXpToNext(L) =
20
+ 4 × L
+ floor(L² ÷ 10)
```

Tabela:

| Nível | XP para o próximo |
|---:|---:|
| `1` | `24` |
| `5` | `42` |
| `10` | `70` |
| `20` | `140` |
| `40` | `340` |
| `60` | `620` |
| `80` | `980` |
| `99` | `1.396` |

Características:

- primeiros níveis sobem rapidamente;
- domínio intermediário exige prática consistente;
- níveis altos exigem grande quantidade de uso;
- nível `100` é domínio máximo normal daquela definição;
- evolução além de `100` exige sistema futuro explícito de transcendência, ascensão ou especialização.

Não existe teto rígido por nível geral nesta versão. O nível geral tende a acompanhar o domínio por receber XP de uso e marcos. Divergências extremas deverão ser avaliadas em simulação antes de criar um bloqueio artificial.

## 14. Curva de perícia

Perícias utilizam níveis de `1` a `100`.

```text
SkillXpToNext(L) =
30
+ 6 × L
+ floor(L² ÷ 8)
```

| Nível | XP para o próximo |
|---:|---:|
| `1` | `36` |
| `5` | `63` |
| `10` | `102` |
| `20` | `200` |
| `40` | `470` |
| `60` | `840` |
| `80` | `1.310` |
| `99` | `1.849` |

Perícias evoluem mais lentamente porque recebem XP de várias ações relacionadas.

## 15. Curva de proficiência

Proficiências utilizam níveis de `1` a `100`.

```text
ProficiencyXpToNext(L) =
25
+ 5 × L
+ floor(L² ÷ 9)
```

| Nível | XP para o próximo |
|---:|---:|
| `1` | `30` |
| `5` | `52` |
| `10` | `86` |
| `20` | `169` |
| `40` | `402` |
| `60` | `725` |
| `80` | `1.136` |
| `99` | `1.609` |

Ataques básicos normalmente evoluem proficiência, sem criar domínio individual para cada golpe comum.

## 16. Curva de profissão

Profissões utilizam níveis de `1` a `100`.

```text
ProfessionXpToNext(L) =
50
+ 8 × L
+ floor(L² ÷ 6)
```

| Nível | XP para o próximo |
|---:|---:|
| `1` | `58` |
| `5` | `94` |
| `10` | `146` |
| `20` | `276` |
| `40` | `636` |
| `60` | `1.130` |
| `80` | `1.756` |
| `99` | `2.475` |

Profissão recebe XP por trabalho real, etapas relevantes e conclusão profissional, não por cada microgesto.

## 17. Curva de treinamento de atributos

O custo depende do valor primário permanente atual `A`, antes de equipamentos e modificadores temporários.

```text
AttributeTrainingXpToIncrease(A) =
50
+ 5 × A
+ 2 × A²
```

| Valor atual | XP de treinamento necessária |
|---:|---:|
| `5` | `125` |
| `10` | `300` |
| `15` | `575` |
| `20` | `950` |
| `30` | `2.000` |
| `40` | `3.450` |
| `50` | `5.300` |

Ao atingir o limiar:

```text
se primaryGrowthCapacityAvailable >= 1
→ consome 1 capacidade
→ aumenta atributo em 1
→ preserva XP excedente
```

Quando não existe capacidade disponível:

```text
attributeTrainingXp continua acumulada
atributo não aumenta
widget informa CAPACITY_REQUIRED
```

A criação inicial continua usando os `16` pontos livres. A curva acima se aplica ao crescimento por uso após a criação.

## 18. Escalonamento mecânico por domínio

O domínio não altera automaticamente todos os campos.

Cada definição declara canais e limite no nível `100`:

```json
{
  "masteryScaling": {
    "damagePercent": {
      "maxBonusAt100": 30,
      "curveExponent": 1.0
    },
    "manaCostPercent": {
      "maxBonusAt100": -10,
      "curveExponent": 1.3
    },
    "windupPercent": {
      "maxBonusAt100": -8,
      "curveExponent": 1.2
    },
    "effectChancePercentagePoints": {
      "maxBonusAt100": 10,
      "curveExponent": 1.5
    }
  }
}
```

Fórmula:

```text
normalizedMastery = (masteryLevel - 1) ÷ 99
resolvedMasteryBonus =
maxBonusAt100 × normalizedMastery ^ curveExponent
```

Regras:

- nível `1` concede aproximadamente `0%` do bônus de domínio;
- nível `100` concede `100%` do máximo declarado;
- conteúdo pode usar marcos além da curva gradual;
- reduções de custo e tempo respeitam pisos mecânicos;
- chance de efeito respeita limites;
- toda definição escalável deve declarar seus canais;
- ausência de canal significa que aquele campo não escala por domínio.

## 19. Exemplo: dez Bolas de Fogo ao alto

Definição:

```text
executionBaseXp = 6
resultBaseXp = 4
context = FREE_PRACTICE = 0,50
difficulty = 1,00
execution = 1,00
result = 0,00
repetition dos usos 1–10 = 1,00
Cansaço = RESTED = 1,00
```

Por conjuração:

```text
ExecutionLearning = 6 × 0,50 = 3,00
ResultLearning = 0
RawLearning = 3,00
```

Fatores:

```text
Domínio Bola de Fogo = 1,00
Piromancia = 0,40
AttributePool = 0,25
ActorDevelopment = 0,10
```

Total em dez conjurações:

| Trilha | XP |
|---|---:|
| Domínio de Bola de Fogo | `30,00` |
| Piromancia | `12,00` |
| Pool de atributos | `7,50` |
| XP geral por uso | `3,00` |

Com pesos:

```text
Inteligência 60% → 4,50 XP
Destreza 30% → 2,25 XP
Vitalidade 10% → 0,75 XP
```

No domínio nível `1`, são necessários `24 XP` para o próximo nível. Portanto:

```text
Bola de Fogo 1 → 2
restam 6 XP no novo nível
```

Isso não aumenta Inteligência imediatamente, pois atributos possuem curva muito mais lenta.

Custos e consequências continuam integrais:

```text
40 Mana consumida
10 conjurações de tempo
Cansaço gerado pela definição
possíveis eventos ambientais
```

## 20. Exemplo: Bola de Fogo contra inimigo equivalente

Hipótese:

```text
executionBaseXp = 6
resultBaseXp = 4
context = REAL_CHALLENGE = 1,25
difficulty = 1,00
execução limpa = 1,00
resultado normal = 1,00
```

Acerto com resultado normal:

```text
RawLearning =
(6 × 1,25)
+
(4 × 1,25)
= 12,50 XP
```

Distribuição:

```text
Domínio = 12,50
Piromancia = 5,00
AttributePool = 3,125
XP geral = 1,25
```

## 21. Exemplo: tentativa contra inimigo muito forte

Hipótese:

```text
context = REAL_CHALLENGE = 1,25
personalChallengeRatio entre 1,50 e 2,00
difficulty = 1,35
ataque errou
executionOutcome = 0,85
result = 0
```

```text
RawLearning =
6 × 1,25 × 1,35 × 0,85
= 8,60625 XP
```

O jogador aprende pela tentativa real, mas o inimigo mantém sua ação normal e pode derrotá-lo.

## 22. Exemplo: ataque físico e dano recebido

Ataque físico simples:

```text
executionBaseXp = 4
resultBaseXp = 2
```

Pode gerar:

- proficiência da arma ou combate desarmado;
- perícia relacionada;
- Força, Destreza, Agilidade e Vitalidade conforme pesos;
- XP geral.

Dano recebido cria evento separado.

Perfil inicial de exposição:

```text
DamageExposureFactor =
clamp(
  0,50,
  actualDamage ÷ (0,10 × maxHealth),
  1,50
)
```

Regras:

- dano igual a `10%` da Vida máxima usa fator `1,00`;
- dano muito pequeno usa no mínimo `0,50`;
- dano muito alto usa no máximo `1,50` por evento;
- receber dano pode treinar Vitalidade, resiliência ou tolerância;
- não concede Esquiva, Bloqueio ou Aparo sem tentativa correspondente;
- Vida, ferimentos, condições e morte permanecem reais.

## 23. Intervalos para movimento e condicionamento

Não criar XP por metro ou passo.

Baseline:

| Atividade | Intervalo canônico inicial |
|---|---:|
| Caminhada | `10 minutos` |
| Corrida moderada | `5 minutos` |
| Corrida intensa | `2 minutos` |
| Natação | `2 minutos` |
| Escalada | trecho ou `1 minuto` |
| Musculação | série ou `5 minutos` |
| Meditação | `10 minutos` |
| Estudo | `15 minutos` |

O intervalo pode terminar antes por mudança de terreno, intensidade, risco, recurso ou evento.

## 24. Progressão em lote

Treinamento em lote deve produzir o mesmo resultado que processar os eventos individualmente na mesma ordem.

Requisitos:

1. calcular custo por repetição;
2. calcular tempo por repetição;
3. atualizar Cansaço antes do evento seguinte;
4. recalcular multiplicador de repetição;
5. recalcular disponibilidade de recursos;
6. resolver falhas e interrupções;
7. criar `ProgressionEvent` por evento ou agregado reproduzível;
8. persistir o resultado atomicamente;
9. respeitar idempotência;
10. devolver resumo e trilha auditável.

O backend pode compactar eventos equivalentes para armazenamento, desde que preserve:

```text
quantidade
ordem lógica
faixas de multiplicador
custos
resultados
XP total por trilha
motivo de parada
```

## 25. Limites contra distorção sem proibir treino

O sistema não usa um bloqueio arbitrário de XP diária.

Usa:

- custos reais;
- tempo real;
- Cansaço;
- sono;
- recursos;
- durabilidade;
- dano ambiental;
- risco;
- repetição gradual;
- curvas crescentes;
- limites por evento;
- idempotência;
- necessidade de capacidade de crescimento para atributos.

Não permitido:

- duplicar o mesmo evento;
- dividir artificialmente uma ação em vários eventos de XP;
- conceder XP por texto não resolvido;
- criar alvo ou recurso inexistente sem materialização;
- ignorar custo porque a atividade é treinamento;
- transformar dano recebido em defesa não executada.

## 26. Configuração versionada

Estrutura sugerida:

```json
{
  "ruleVersion": "progression-curves-v0.1",
  "storage": {
    "xpScale": 1000
  },
  "contextMultipliers": {},
  "difficultyBands": [],
  "outcomeMultipliers": {},
  "repetitionBands": [],
  "fatigueMultipliers": {},
  "curves": {
    "actor": {},
    "mastery": {},
    "skill": {},
    "proficiency": {},
    "profession": {},
    "attribute": {}
  },
  "milestoneRewards": {},
  "activityIntervals": {}
}
```

O widget recebe versão e valores necessários para prévia. O backend permanece autoritativo.

## 27. Critérios de aceite para o Codex

### Fórmulas

- calcular corretamente todas as curvas nos níveis de referência;
- usar aritmética determinística em `xpMilli`;
- não perder frações de atributo;
- não conceder XP a comando rejeitado;
- conceder XP parcial a falha válida;
- aplicar contexto, dificuldade, repetição e Cansaço na ordem definida.

### Idempotência

- repetir `sourceEventId` não duplica XP;
- repetir comando de treinamento com mesma chave retorna o mesmo resultado;
- payload diferente com mesma chave retorna conflito.

### Treinamento

- dez Bolas de Fogo com `40` de Mana e custo `4` resolvem no máximo dez usos;
- cada uso produz progressão e Cansaço;
- a sessão para no primeiro limite;
- repetir prática além de dez usos começa a aplicar retorno gradual;
- prática válida nunca zera somente por repetição.

### Combate

- ataque contra inimigo forte aplica dificuldade maior;
- inimigo continua agindo e aplicando consequências;
- errar mantém XP de execução;
- acerto acrescenta XP de resultado;
- receber dano não concede Esquiva sem tentativa.

### Atributos

- XP fracionária é preservada;
- atributo só aumenta com limiar e capacidade disponível;
- aumento consome exatamente uma capacidade;
- XP excedente é preservada;
- equipamentos não reduzem nem aumentam o custo de treinamento permanente.

## 28. Simulações obrigatórias

Antes de alterar os valores:

1. dez Bolas de Fogo em prática livre;
2. cem Bolas de Fogo em prática livre com Cansaço crescente;
3. dez conjurações contra inimigo equivalente;
4. ataque contra alvo com metade da ameaça;
5. ataque contra alvo com o dobro da ameaça;
6. treino com boneco por uma hora;
7. sparring por trinta minutos;
8. corrida por duas horas;
9. evolução de domínio `1 → 20`;
10. evolução de perícia `1 → 20`;
11. aumento de atributo `5 → 10`;
12. nível geral `1 → 10` com mistura de treino, combate e objetivos;
13. sessão repetida com mesma idempotencyKey;
14. interrupção por Mana, Vigor, Cansaço, horário e ameaça.

## 29. Pendências de calibração futura

- validar velocidade real de progressão com sessões simuladas;
- definir perfis-padrão por categoria de ação;
- calibrar XP de cura, suporte e controle sem dano;
- calibrar atividades sociais;
- calibrar fabricação por etapa e qualidade;
- calibrar estudo com e sem instrutor;
- calibrar progressão de companheiros;
- avaliar bônus ou penalidades por afinidade;
- avaliar soft caps somente após evidência de desequilíbrio;
- integrar comida, água, ferimentos e condições;
- definir reset ou migração quando uma curva for revisada.

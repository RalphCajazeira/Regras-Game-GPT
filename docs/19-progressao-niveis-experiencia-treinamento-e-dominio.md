# Progressão, Níveis, Experiência, Treinamento e Domínio

**Versão da proposta:** `progression-v0.1`  
**Status:** em validação

## 1. Objetivo

Definir uma progressão em que aquilo que o ator pratica melhora diretamente, sem limitar evolução a vitórias, mortes ou missões.

A arquitetura deve permitir que o jogador evolua por:

- combate real;
- treino com alvo, árvore, boneco ou equipamento apropriado;
- repetição de magia, habilidade ou movimento;
- sparring com outro ator;
- estudo, meditação, fabricação e prática profissional;
- tentativa válida, mesmo quando o resultado falhar;
- ações criativas interpretadas pelo GPT e resolvidas pelo backend.

A consequência mecânica é o limitador principal. Mana, Vigor, Cansaço, Vida, ferimentos, tempo, risco, sono e recursos impedem repetição infinita sem custo.

```text
uso válido
→ custos e consequências reais
→ eventos de aprendizagem
→ XP nas progressões aplicáveis
→ níveis, domínio e atributos podem evoluir
```

## 2. Princípios canônicos

1. Toda ação mecanicamente executada pode gerar aprendizagem.
2. Uma ação não precisa ocorrer em combate para conceder XP.
3. Repetição não perde toda a XP apenas por ser repetição.
4. Contextos diferentes alteram quantidade e tipo de XP.
5. Falha pode ensinar; comando rejeitado antes da execução não ensina.
6. Custos, tempo, dano e Cansaço nunca são ignorados durante treinamento.
7. Receber dano não equivale automaticamente a defender.
8. O backend resolve e persiste toda progressão oficial.
9. O widget mostra prévias, barras e custos, mas não concede XP.
10. O GPT interpreta objetivos de treino, cria contexto e narra marcos.
11. Experiência é concedida por evento resolvido, não por texto declarado.
12. Progressão deve ser auditável, idempotente e vinculada ao evento que a originou.

## 3. Camadas de progressão

```text
Nível geral do ator
Atributos primários
Perícias
Proficiências
Domínio de habilidades e magias
Profissões
Passivas, especializações e desbloqueios
```

Cada camada possui XP, curva e função próprias.

Uma mesma ação pode gerar progresso em várias camadas.

Exemplo:

```text
Bola de Fogo
→ domínio de Bola de Fogo
→ perícia Piromancia
→ treinamento de Inteligência
→ treinamento de Destreza
→ experiência geral de desenvolvimento
```

## 4. Evento canônico de aprendizagem

Toda aprendizagem nasce de um evento mecânico confirmado:

```json
{
  "progressionEventId": "uuid",
  "sourceEventId": "event-uuid",
  "actorId": "actor-uuid",
  "sourceType": "ACTION_RESOLVED",
  "sourceCode": "fireball",
  "contextType": "FREE_PRACTICE",
  "outcome": "SUCCESS",
  "costsPaid": {
    "mana": 4,
    "stamina": 0,
    "health": 0,
    "timeTicks": 20,
    "fatigue": 1
  },
  "tracks": [],
  "ruleVersion": "progression-v0.1"
}
```

Fontes iniciais:

```text
ACTION_STARTED
ACTION_RESOLVED
ACTION_INTERRUPTED
ATTACK_ATTEMPTED
ATTACK_HIT
DAMAGE_DEALT
DAMAGE_RECEIVED
DODGE_ATTEMPTED
DODGE_SUCCEEDED
BLOCK_ATTEMPTED
BLOCK_SUCCEEDED
SPELL_CAST
SKILL_CHECK_RESOLVED
ITEM_CRAFTED
ITEM_EVALUATED
TRADE_RESOLVED
TRAVEL_COMPLETED
TRAINING_INTERVAL_COMPLETED
OBJECTIVE_COMPLETED
DISCOVERY_CONFIRMED
```

## 5. Quando existe XP

### 5.1 Ação executada

Se a ação começou validamente e produziu custo, esforço ou execução técnica, pode gerar XP de prática.

### 5.2 Ação bem-sucedida

Sucesso pode acrescentar XP de resultado, precisão, eficiência ou efeito.

### 5.3 Ação que falhou

Uma falha válida pode conceder:

- XP de execução;
- XP de tentativa;
- aprendizado técnico;
- experiência de dificuldade;
- menor XP de resultado, porque o efeito não ocorreu.

### 5.4 Ação interrompida

Pode conceder XP parcial quando houve preparação real, custo ou esforço antes da interrupção.

### 5.5 Comando rejeitado

Não concede XP quando nenhuma ação ocorreu, por exemplo:

- alvo inexistente;
- magia não conhecida;
- recurso insuficiente antes de começar;
- comando estruturalmente inválido;
- ação proibida pela definição.

## 6. Contextos de prática

```text
FREE_PRACTICE
STATIC_TARGET
TRAINING_DUMMY
CONTROLLED_EXERCISE
STRUCTURED_TRAINING
SPARRING
REAL_CHALLENGE
EXTREME_CHALLENGE
PROFESSIONAL_PRACTICE
STUDY
MEDITATION
```

### 6.1 Prática livre

Exemplo:

> Antes de dormir, Kael lança Bola de Fogo ao alto até esgotar a Mana.

A conjuração concede XP porque houve:

- execução real;
- consumo real de Mana;
- tempo;
- Cansaço;
- controle da magia.

Não existe bônus de acerto em alvo, dano útil ou pressão de combate.

### 6.2 Alvo estático

Árvore, parede ou alvo comum permitem treinar:

- execução;
- mira;
- alcance;
- controle de força;
- domínio da ação.

Também podem produzir consequências no mundo, como dano ao objeto, incêndio, barulho ou descoberta por terceiros.

### 6.3 Boneco de treino

Pode possuir perfil próprio:

```text
resistência
material
área de acerto
feedback
limite de durabilidade
tipo de treino suportado
```

Um boneco adequado pode fornecer melhor XP técnica que um objeto improvisado.

### 6.4 Sparring

Treino contra ator reativo pode desenvolver:

- ataque;
- defesa;
- esquiva;
- bloqueio;
- timing;
- leitura do oponente;
- controle de força;
- habilidades específicas.

As regras do sparring definem dano, proteção, interrupção e condição de encerramento.

### 6.5 Combate real

Combate real acrescenta fatores como:

- ameaça;
- pressão;
- imprevisibilidade;
- reação do inimigo;
- risco de ferimento, captura ou morte;
- uso real do efeito.

Um inimigo mais forte pode conceder mais XP por ação, mas o risco não é removido. O inimigo continua agindo normalmente e pode derrotar o jogador antes que ele escape.

## 7. Fórmula conceitual de XP por evento

```text
XP Final da Trilha =
XP Base de Execução
× Qualidade do Contexto
× Dificuldade Relativa
× Qualidade da Execução
× Relevância do Resultado
× Eficiência de Aprendizagem
+ Bônus de Marco
```

Todos os multiplicadores e limites são versionados.

### 7.1 XP Base de Execução

Pertence à definição da ação, habilidade, magia, perícia ou atividade.

### 7.2 Qualidade do contexto

Uma prática livre pode ensinar menos que um duelo real, mas continua concedendo XP.

Faixas iniciais apenas para simulação:

| Contexto | Multiplicador sugerido |
|---|---:|
| `FREE_PRACTICE` | `0,35–0,60` |
| `STATIC_TARGET` | `0,45–0,75` |
| `TRAINING_DUMMY` | `0,55–0,90` |
| `STRUCTURED_TRAINING` | `0,70–1,10` |
| `SPARRING` | `0,85–1,30` |
| `REAL_CHALLENGE` | `0,75–2,00` |
| `EXTREME_CHALLENGE` | até `2,50`, sujeito a risco e limite por evento |

Esses valores não são finais.

### 7.3 Dificuldade relativa

Não depende somente do nível nominal. Considera:

- nível e orçamento do alvo;
- atributos efetivos;
- equipamentos;
- ações disponíveis;
- condições;
- número de participantes;
- terreno;
- posição;
- risco real apresentado ao ator.

### 7.4 Resultado

Uma tentativa pode gerar XP de execução mesmo errando.

Acerto, dano, cura útil, condição aplicada ou defesa efetiva podem acrescentar XP de resultado.

### 7.5 Eficiência de aprendizagem

Pode variar por:

- Cansaço;
- qualidade do instrutor;
- equipamento de treino;
- concentração;
- ferimentos;
- ambiente;
- afinidade;
- repetição idêntica;
- passivas.

Uma ação validamente executada não recebe `0 XP` somente porque foi repetida. A eficiência pode diminuir gradualmente, mas os custos e o tempo continuam reais.

## 8. Nível geral do ator

O nível geral representa desenvolvimento total, não somente quantidade de inimigos mortos.

Pode receber experiência por:

- uso de habilidades, magias, perícias e profissões;
- crescimento de atributos;
- objetivos e missões;
- descobertas;
- desafios sociais;
- fabricação relevante;
- marcos de domínio;
- treino consistente;
- combate.

```text
actorDevelopmentXp
actorLevel
xpToNextActorLevel
```

### 8.1 Função do nível

O nível geral pode determinar:

- capacidade de crescimento primário;
- escala de recursos;
- referências de desafio;
- requisitos de conteúdo;
- pontos de talento ou especialização futuros;
- validação de materialização de NPCs.

### 8.2 Capacidade de crescimento primário

A proposta atual de `+10` pontos primários por nível é preservada como capacidade máxima de crescimento, não como distribuição manual obrigatória.

```text
subida de nível
→ +10 primaryGrowthCapacity
→ uso decide quais atributos acumulam treinamento
→ atributo aumenta quando possui XP e capacidade disponível
```

Campos:

```text
primaryGrowthCapacityEarned
primaryGrowthCapacitySpent
primaryGrowthCapacityAvailable
```

## 9. Atributos primários por uso

Cada ação declara contribuições de treinamento:

```json
{
  "attributeTrainingProfile": [
    { "attribute": "intelligence", "weight": 0.60 },
    { "attribute": "dexterity", "weight": 0.30 },
    { "attribute": "vitality", "weight": 0.10 }
  ]
}
```

Exemplos:

### Bola de Fogo

```text
Inteligência → formação e potência mágica
Destreza → controle técnico e precisão
Vitalidade → pequena adaptação ao esforço, quando aplicável
```

### Soco

```text
Força → impacto
Destreza → execução técnica
Agilidade → timing e deslocamento
Vitalidade → esforço físico
```

### Corrida

```text
Agilidade
Vitalidade
pequena contribuição de Força conforme inclinação e carga
```

### Receber dano

Pode treinar:

```text
Vitalidade
Tolerância à Dor
Resiliência
Proficiência de armadura, quando houve absorção relevante
```

Não treina automaticamente Esquiva ou Bloqueio.

## 10. Ataque, defesa e dano recebido

### 10.1 Ataque

Uma tentativa válida pode conceder:

- domínio da ação;
- proficiência da arma;
- perícia relacionada;
- atributos de execução;
- XP de acerto e dano quando houver resultado.

### 10.2 Esquiva

Somente uma tentativa real de esquiva gera XP de Esquiva.

Receber dano sem tentar esquivar não conta como treino de Esquiva.

### 10.3 Bloqueio e aparo

Somente tentativa válida de bloqueio ou aparo gera XP dessas progressões.

### 10.4 Dano recebido

Dano real pode desenvolver resistência, tolerância, Vitalidade ou adaptação relevante, mas também reduz Vida e pode causar condição, ferimento, incapacidade ou morte.

## 11. Treino contra si mesmo

O jogador pode propor treino autoinfligido quando a ação for fisicamente e mecanicamente possível.

Exemplo:

> Vou me atingir com socos controlados até minha Vida chegar perto de 10.

O backend deve:

1. validar que a ação pode atingir o próprio ator;
2. aplicar custos e tempo;
3. resolver cada ataque;
4. aplicar dano real;
5. conceder XP às trilhas realmente praticadas;
6. não conceder Esquiva ou Bloqueio sem tentativa correspondente;
7. interromper conforme condição de parada ou incapacidade;
8. preservar risco de ferimento e morte.

Não existe proteção narrativa automática contra consequências.

## 12. Sessões de treinamento em lote

O GPT e o widget não precisam enviar uma chamada para cada repetição.

Comandos conceituais:

```text
TRAIN_ACTION_REPETITION
TRAIN_UNTIL_RESOURCE_LIMIT
TRAIN_FOR_DURATION
TRAIN_UNTIL_FATIGUE
TRAIN_UNTIL_HEALTH_THRESHOLD
SPARRING_SESSION
PHYSICAL_CONDITIONING
MEDITATION_SESSION
STUDY_SESSION
```

Exemplo:

```json
{
  "command": "TRAIN_UNTIL_RESOURCE_LIMIT",
  "actorId": "actor-uuid",
  "actionCode": "fireball",
  "targetMode": "AIR_POINT",
  "stopConditions": {
    "stopWhenManaBelowNextCost": true,
    "maximumDurationMinutes": 30,
    "maximumFatigue": 70
  },
  "baseStateVersion": 15
}
```

Se o ator possui `40` de Mana e cada conjuração custa `4`, poderá executar até `10` conjurações quando nenhum outro modificador alterar custo ou interromper a sessão.

O backend ainda resolve:

- custo por repetição;
- tempo;
- rolagens aplicáveis;
- falhas;
- interrupções;
- Cansaço;
- eventos no ambiente;
- XP individual de cada uso;
- condição final.

## 13. Condições de parada

```text
MANA_BELOW_NEXT_COST
STAMINA_BELOW_NEXT_COST
HEALTH_AT_OR_BELOW
FATIGUE_AT_OR_ABOVE
WORLD_TIME_REACHED
DURATION_REACHED
REPETITIONS_REACHED
ACTION_FAILED_CRITICALLY
ACTOR_INCAPACITATED
TARGET_DESTROYED
THREAT_DETECTED
PLAYER_DECISION_REQUIRED
```

A condição de parada faz parte do comando e do log.

## 14. Perícias e proficiências

### 14.1 Perícia

Representa uma área ampla de prática:

```text
Piromancia
Ferraria
Furtividade
Comércio
Sobrevivência
Primeiros Socorros
```

### 14.2 Proficiência

Representa domínio de categoria específica:

```text
Adagas
Espadas
Arcos
Cajados
Armaduras Pesadas
Ferramentas de Ferreiro
```

Ataques universais normalmente evoluem a proficiência, sem exigir uma barra individual para cada ataque básico.

## 15. Domínio individual de habilidade e magia

Cada ator possui domínio próprio da definição conhecida:

```json
{
  "actorId": "actor-uuid",
  "contentDefinitionId": "definition-uuid",
  "contentCode": "fireball",
  "contentVersion": 3,
  "masteryLevel": 24,
  "masteryXp": 340,
  "xpToNextMasteryLevel": 410,
  "timesStarted": 90,
  "timesResolved": 82,
  "successfulUses": 61
}
```

A definição declara o que pode melhorar:

```text
dano
cura
precisão
alcance
área
duração
custo
preparação
recuperação
penetração
chance de efeito
número de alvos
estabilidade
```

Nem toda técnica melhora todos os campos.

## 16. Escalonamento e marcos

### 16.1 Crescimento gradual

Exemplo:

```text
+0,4% de dano-base por nível de domínio
```

### 16.2 Marcos

Exemplo:

```text
nível 20 → maior estabilidade
nível 40 → redução de custo
nível 60 → novo efeito disponível
nível 80 → escolha de especialização
nível 100 → domínio máximo
```

Valores são definidos por conteúdo e versionados.

## 17. Fórmula mecânica final

Uma técnica não depende apenas do próprio domínio.

```text
Resultado Final =
base da definição
+ atributos
+ perícia
+ proficiência
+ domínio individual
+ equipamento
+ passivas
+ condições
+ ambiente
```

Dois atores com a mesma Bola de Fogo no mesmo nível de domínio podem produzir resultados diferentes.

## 18. Profissões

Profissões recebem XP por atividades reais do domínio.

Exemplo:

```text
forjar Adaga Élfica
→ Ferraria
→ proficiência do material
→ profissão Ferreiro
→ domínio da receita, quando existir
→ Força, Destreza e Inteligência conforme perfil
```

A profissão organiza:

- receitas;
- passivas;
- especializações;
- identidade profissional;
- requisitos;
- marcos.

## 19. Cansaço e eficiência

Cansaço não bloqueia artificialmente toda aprendizagem.

Ele afeta:

- capacidade de continuar;
- velocidade;
- precisão;
- risco de falha;
- recuperação de Vigor;
- eficiência de aprendizagem;
- possibilidade de colapso.

Uma ação que ainda foi validamente executada pode gerar XP mesmo em Cansaço alto.

O sistema completo está em:

`docs/20-vigor-cansaco-sono-e-recuperacao.md`

## 20. Persistência e atomicidade

Progressão gerada por comando ou sessão deve ser consolidada em lote:

```text
eventos resolvidos
→ progressionEvents
→ validação
→ agregação por trilha
→ níveis e marcos
→ persistência atômica
→ snapshotDelta
```

Toda mutação usa:

```text
requestId
idempotencyKey
baseStateVersion
ruleVersions
```

Repetir a mesma resolução não concede XP novamente.

## 21. Resposta do backend

```json
{
  "progression": {
    "eventsProcessed": 10,
    "gains": [
      {
        "trackType": "ABILITY_MASTERY",
        "code": "fireball",
        "xpGained": 42,
        "previousLevel": 24,
        "newLevel": 25
      },
      {
        "trackType": "SKILL",
        "code": "pyromancy",
        "xpGained": 18
      },
      {
        "trackType": "ATTRIBUTE_TRAINING",
        "code": "intelligence",
        "xpGained": 8
      }
    ],
    "actorDevelopmentXpGained": 9,
    "milestones": ["ABILITY_MASTERY_LEVEL_UP"]
  },
  "snapshotDelta": {},
  "narrationDirective": {
    "level": "IMPORTANT",
    "reasonCodes": ["ABILITY_MASTERY_LEVEL_UP"]
  }
}
```

## 22. Widget

O widget deve mostrar:

- nível geral e XP;
- capacidade de crescimento primário;
- treinamento de atributos;
- perícias;
- proficiências;
- domínio de habilidades e magias;
- profissões;
- marcos próximos;
- custo de treino;
- duração;
- Cansaço previsto;
- condições de parada;
- resumo da sessão.

A prévia não garante XP exata porque falhas, resultados, Cansaço e eventos podem alterar a resolução.

## 23. Responsabilidades

### GPT

- interpretar o objetivo de treino;
- estruturar ação ou sessão;
- propor alvo, duração e condição de parada;
- criar contexto narrativo;
- não conceder XP diretamente;
- narrar marcos relevantes.

### Widget

- exibir progressões e barras;
- montar comandos comuns;
- mostrar prévias de custo, tempo e risco;
- atualizar o estado confirmado.

### Backend

- resolver cada uso ou lote;
- aplicar custos e consequências;
- criar eventos de progressão;
- calcular XP oficial;
- elevar níveis e domínio;
- aplicar marcos;
- impedir duplicação;
- persistir e auditar.

## 24. Pontos para simulação

- curvas de XP de cada trilha;
- peso de XP geral produzido pelo uso;
- multiplicadores por contexto;
- limites máximos por evento;
- crescimento gradual de magias e habilidades;
- custo de atributos por valor atual;
- relação entre nível geral e capacidade primária;
- aprendizagem sob Cansaço;
- treinamento com instrutor;
- treinamento offline ou por passagem de tempo;
- progressão de companheiros;
- perda, retenção ou transformação de XP após morte e rollback.

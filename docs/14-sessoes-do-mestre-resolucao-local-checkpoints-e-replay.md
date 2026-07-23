# Sessões do Mestre, Resolução Local, Checkpoints e Replay

**Versão da proposta:** `master-runtime-v0.1`  
**Status:** em validação

## 1. Objetivo

Definir como o GPT atua como Mestre do jogo com liberdade para conduzir várias microações localmente, enquanto o backend funciona como livro de regras, árbitro, validador e persistência autoritativa.

O sistema deve evitar chamadas ao backend para cada ataque, movimento, rolagem, reação, fala ou tick, sem permitir que o GPT invente recursos, ignore regras ou produza consequências impossíveis de validar.

```text
Backend prepara e valida a mesa
→ GPT conduz a sessão localmente
→ GPT registra eventos e consequências
→ checkpoint quando necessário
→ backend reproduz, ajusta e persiste
```

## 2. Papéis

### GPT — Mestre

O GPT pode:

- interpretar intenções livres do jogador;
- escolher ações válidas para NPCs e criaturas;
- conduzir diálogos e decisões táticas;
- avançar o tempo local autorizado;
- realizar e registrar rolagens;
- aplicar custos, danos, curas e condições carregadas;
- controlar a fila de eventos;
- manter consequências pendentes;
- narrar livremente detalhes sem impacto mecânico;
- solicitar checkpoint, revisão, materialização ou persistência quando necessário.

### Backend — Livro, árbitro e persistência

O backend deve:

- fornecer regras e definições versionadas;
- fechar dependências necessárias;
- materializar atores, itens e ambiente;
- calcular valores derivados e limites;
- declarar a autoridade local concedida;
- validar prontidão;
- reproduzir logs e rolagens;
- aceitar, ajustar ou rejeitar resoluções;
- persistir consequências autoritativas;
- indicar replay ou recuperação quando houver divergência;
- preservar histórico, versões e resumos.

## 3. Camadas do estado

```text
Livro de regras
+ estado autoritativo persistido
+ snapshot temporário da sessão
+ log local do Mestre
+ consequências pendentes
```

### 3.1 Livro de regras

Contém fórmulas, definições, ações, itens, condições, perícias, economia, combate e demais versões aplicáveis.

### 3.2 Estado autoritativo

Contém Vida, Mana, Vigor, inventário, dinheiro, equipamentos, relações, missões, localização, sessões, atores e demais dados persistidos.

### 3.3 Snapshot temporário

É a mesa preparada para a sessão. Deve conter somente os dados necessários, mas com fechamento completo das dependências mecânicas aplicáveis.

### 3.4 Log local

Registra ações, tempos, rolagens, custos, efeitos e estados antes/depois.

### 3.5 Consequências pendentes

Armazena alterações resolvidas pelo GPT que ainda aguardam validação ou persistência.

## 4. Fechamento de dependências

Uma sessão só pode ficar pronta quando todas as dependências mecânicas necessárias estiverem disponíveis.

```text
ator
→ atributos e recursos
→ ações e passivas
→ equipamento
→ ações concedidas pelo equipamento
→ consumíveis e munições
→ condições aplicáveis
→ regras e versões
```

Resposta esperada:

```json
{
  "dependencyClosure": {
    "status": "COMPLETE",
    "loadedReferences": [],
    "missingReferences": [],
    "invalidReferences": []
  }
}
```

Estados:

```text
COMPLETE
INCOMPLETE
INVALID
REQUIRES_MATERIALIZATION
REQUIRES_REVIEW
```

Uma sessão de combate não pode ser `READY` quando faltar perfil de ataque, custo, condição referenciada, munição necessária ou outra dependência obrigatória.

## 5. Perfis de sessão

```text
COMBAT
EVALUATION
NEGOTIATION
CHASE
INFILTRATION
HARVEST
RECRUITMENT
TAMING
EXPLORATION
MIXED_ENCOUNTER
```

Um perfil define:

- dados obrigatórios do snapshot;
- autoridade local;
- eventos permitidos;
- política de checkpoint;
- transições de fase;
- consequências que exigem backend imediato.

## 6. Autoridade local

Toda sessão deve informar explicitamente o que o GPT pode resolver sem nova chamada.

Níveis:

```text
LOCAL
BUFFERED
IMMEDIATE_BACKEND
FORBIDDEN
```

### 6.1 `LOCAL`

Pode ser resolvido no snapshot e submetido depois:

- movimento;
- linha do tempo;
- dano e cura;
- Mana e Vigor;
- condições já carregadas;
- consumo de munição;
- uso de consumível materializado;
- reações;
- decisões táticas;
- mudanças temporárias de posição e estado.

### 6.2 `BUFFERED`

Pode ser decidido localmente, mas permanece como consequência pendente:

- impressão ou memória relevante;
- mudança proposta de relação;
- progresso de objetivo;
- XP estimado;
- reputação proposta;
- descoberta de informação;
- saque formado;
- mudança de fase.

### 6.3 `IMMEDIATE_BACKEND`

Exige checkpoint ou operação autoritativa antes da continuidade incompatível:

- concluir compra ou venda;
- criar item persistente;
- concluir fabricação;
- conceder XP ou nível definitivo;
- concluir missão e recompensa;
- promover ator temporário;
- migrar ou revisar definição;
- ressuscitar ator;
- transferir propriedade crítica;
- alterar permanentemente o mundo fora do escopo autorizado.

### 6.4 Exemplo

```json
{
  "localAuthority": {
    "advanceSessionTime": "LOCAL",
    "modifyTemporaryResources": "LOCAL",
    "applyLoadedConditions": "LOCAL",
    "consumeLoadedItems": "LOCAL",
    "updateRelationship": "BUFFERED",
    "grantExperience": "BUFFERED",
    "createPermanentItem": "IMMEDIATE_BACKEND",
    "changeRuleDefinition": "IMMEDIATE_BACKEND"
  }
}
```

## 7. Snapshot da sessão

Estrutura resumida:

```json
{
  "sessionId": "uuid",
  "sessionType": "ENCOUNTER",
  "profile": "COMBAT",
  "phase": "COMBAT",
  "lifecycleStatus": "ACTIVE",
  "baseStateVersion": 15,
  "snapshotVersion": 1,
  "ruleVersions": {},
  "dependencyClosure": {},
  "readiness": {},
  "localAuthority": {},
  "participants": [],
  "environment": {},
  "eventQueue": [],
  "pendingActions": [],
  "pendingConsequences": [],
  "randomness": {},
  "visibilityPolicy": {},
  "checkpointPolicy": {},
  "allowedPhaseTransitions": [],
  "lease": {}
}
```

## 8. Log de resolução

O GPT não envia apenas um resultado narrativo. Cada evento mecânico relevante deve possuir registro reproduzível.

```json
{
  "eventSequence": 14,
  "sessionTime": 82,
  "actorRef": "player",
  "actionCode": "fire-bolt",
  "actionVersion": 2,
  "targetRefs": ["alpha-wolf"],
  "stateBefore": {},
  "rolls": [
    {
      "type": "HIT",
      "die": "D100",
      "result": 42,
      "requiredMaximum": 78,
      "rollSequence": 7
    }
  ],
  "costs": [],
  "resolvedEffects": [],
  "stateAfter": {},
  "generatedConsequences": []
}
```

Campos essenciais:

- sequência;
- tempo da sessão;
- ator e ação;
- versão da ação;
- alvos;
- estado anterior;
- rolagens;
- custos;
- efeitos;
- estado posterior;
- consequências geradas.

A narração completa não precisa ser persistida em cada evento.

## 9. Buffer de consequências

```json
{
  "pendingConsequences": [
    {
      "consequenceRef": "consequence-1",
      "type": "RESOURCE_CHANGED",
      "authority": "LOCAL",
      "actorRef": "player",
      "before": 50,
      "after": 38,
      "causeEventSequence": 14
    },
    {
      "consequenceRef": "consequence-2",
      "type": "RELATIONSHIP_IMPRESSION",
      "authority": "BUFFERED",
      "sourceActorRef": "lysandra",
      "targetActorRef": "player",
      "impressionTag": "PROTECTED_ME",
      "proposedImportance": 60
    }
  ]
}
```

Consequências podem ser:

```text
RESOURCE_CHANGED
ITEM_CONSUMED
ITEM_DAMAGED
CONDITION_APPLIED
CONDITION_REMOVED
ACTOR_DEFEATED
ACTOR_KILLED
ACTOR_ESCAPED
LOOT_FORMED
INFORMATION_DISCOVERED
RELATIONSHIP_IMPRESSION
MISSION_PROGRESS_PROPOSED
REPUTATION_CHANGE_PROPOSED
PHASE_TRANSITION
WORLD_EVENT_PROPOSED
```

## 10. Rolagens locais

Modos:

```text
GPT_RECORDED
SEEDED_SEQUENCE
PREGENERATED_ROLL_POOL
BACKEND_IMMEDIATE
```

### 10.1 Primeira versão

O modo inicial pode ser `GPT_RECORDED`:

```text
GPT gera a rolagem
→ registra imediatamente no log
→ associa sequência e finalidade
→ não rerrola o mesmo evento
→ backend valida faixa, quantidade, ordem e coerência
```

### 10.2 Sequência auditável

Em evolução futura:

```json
{
  "randomness": {
    "mode": "SEEDED_SEQUENCE",
    "seed": "session-seed",
    "startingCursor": 0,
    "algorithmVersion": "rng-v1"
  }
}
```

O backend deve conseguir reproduzir a sequência sem ser chamado em cada rolagem.

### 10.3 Pacote de rolagens

O backend também pode entregar valores previamente gerados. O GPT os consome na ordem declarada e registra o cursor.

## 11. Checkpoints

Checkpoints não ocorrem por microação. Devem ocorrer quando:

- uma fase termina;
- um ator persistente morre ou é capturado;
- um item importante é criado, transferido ou consumido;
- um inimigo foge e pode reaparecer;
- uma missão muda de estado;
- uma relação importante é proposta;
- a sessão fica extensa;
- o jogador interrompe;
- há risco de perda de contexto;
- a política do snapshot exigir;
- uma consequência `IMMEDIATE_BACKEND` for necessária.

O checkpoint envia:

```text
eventos desde o último checkpoint
+ consequências pendentes
+ snapshot local final
+ resumo mecânico
+ resumo narrativo proposto
```

O backend valida, persiste o trecho aceito e retorna nova `stateVersion`, novo snapshot ou delta.

## 12. Fases e transições

Uma mesma sessão pode mudar de fase:

```text
COMBAT
→ SURRENDER
→ NEGOTIATION
→ RECRUITMENT_ATTEMPT
→ COMPANION_PROPOSAL
```

Campos:

```text
currentPhase
allowedPhaseTransitions
transitionRequirements
nextProfile
requiredCheckpoint
```

Transições comuns:

```text
EVALUATION → COMBAT
CONVERSATION → TRADE
CONVERSATION → MISSION
COMBAT → LOOT
COMBAT → CAPTURE
COMBAT → NEGOTIATION
COMBAT → ESCAPE
TAMING → COMPANION
```

O backend deve informar quais transições são válidas. O GPT não altera livremente para uma fase incompatível.

## 13. Lease e concorrência

Uma sessão pode possuir:

```json
{
  "lease": {
    "leaseId": "uuid",
    "baseStateVersion": 15,
    "status": "ACTIVE",
    "conflictPolicy": "REVALIDATE_ON_SUBMIT"
  }
}
```

Políticas:

```text
REVALIDATE_ON_SUBMIT
REJECT_ON_CONFLICT
MERGE_NON_OVERLAPPING
REPLAY_REQUIRED
```

Se o estado autoritativo mudar:

- mudança irrelevante pode ser reaplicada;
- mudança compatível pode gerar ajuste;
- conflito direto exige recarga ou replay;
- o backend deve informar o ponto exato afetado.

## 14. Submissão e validação

Resultados possíveis:

```text
PERSISTED
VALIDATED
VALIDATED_WITH_ADJUSTMENTS
PARTIALLY_ACCEPTED
REQUIRES_REPLAY
REQUIRES_REVIEW
STATE_CONFLICT
INVALID_RESOLUTION
```

Exemplo de ajuste não bloqueante:

```text
GPT reportou 27 de dano
backend reproduziu 26 por arredondamento
→ VALIDATED_WITH_ADJUSTMENTS
```

Exemplo bloqueante:

```text
ator utilizou magia ausente do snapshot
→ INVALID_RESOLUTION
```

## 15. Replay parcial

Quando possível, o backend preserva os eventos válidos anteriores ao erro.

```json
{
  "status": "REQUIRES_REPLAY",
  "acceptedThroughSequence": 12,
  "invalidFromSequence": 13,
  "issues": [],
  "recovery": {
    "type": "REPLAY_FROM_EVENT",
    "eventSequence": 13
  }
}
```

Eventos posteriores que dependam do evento inválido devem ser descartados ou recalculados.

## 16. Retomada e recuperação

`loadGame` e as Actions de sessão devem informar:

```text
activeSessions
recoverableSessions
lastCheckpoint
lastAcceptedEventSequence
pendingConsequences
requiredAction
continuationSummary
```

Exemplo:

```json
{
  "activeSession": {
    "type": "ENCOUNTER",
    "status": "PROCESSING_PAUSED",
    "stateVersion": 14,
    "lastAcceptedEventSequence": 23,
    "recoveryAction": {
      "operationId": "startOrLoadEncounter",
      "operation": "LOAD"
    }
  }
}
```

A retomada deve carregar o último estado validado, não depender exclusivamente do histórico do chat.

## 17. Resumos

### 17.1 Resumo mecânico

Contém fatos autoritativos e compactos:

```text
Kael gastou 24 de Mana.
Lobo Alfa sofreu 83 de dano.
Poção de Cura Menor foi consumida.
Lobo Alfa foi derrotado.
A pele foi danificada por Fogo.
```

### 17.2 Resumo narrativo

Preserva continuidade sem substituir o log:

```text
Kael enfrentou o Lobo Alfa perto das ruínas e o derrotou usando magia de fogo, danificando parte da pele.
```

O backend pode armazenar ambos. O resumo narrativo pode ser proposto pelo GPT e validado contra os fatos mecânicos.

## 18. Visibilidade das informações

```text
MASTER_ONLY
PLAYER_KNOWN
PLAYER_VISIBLE
DISCOVERABLE
HIDDEN_UNTIL_TRIGGER
```

O GPT pode receber informações ocultas necessárias para atuar como Mestre, mas deve revelar somente o permitido.

Exemplos `MASTER_ONLY`:

- inventário escondido;
- objetivos secretos;
- armadilha ainda não descoberta;
- fraqueza desconhecida;
- resultado de drop ainda não revelado.

## 19. Detalhe narrativo versus entidade mecânica

```text
NARRATIVE_DETAIL
MECHANICAL_ENTITY
```

O GPT pode criar livremente detalhes sem consequência mecânica:

- clima descritivo;
- postura;
- expressão;
- som;
- objetos cenográficos sem valor ou uso.

Quando um detalhe passa a poder ser coletado, vendido, usado, equipado, destruído ou produzir efeito, ele deve ser materializado como entidade mecânica antes da resolução.

Exemplo:

```text
“Há garrafas vazias no chão.”
→ detalhe narrativo.

Jogador: “Vou recolhê-las para vender.”
→ materializar itens ou informar que não possuem valor mecânico.
```

## 20. Integração com as Actions

Este sistema não consome nova vaga no catálogo.

```text
startOrLoadEncounter
→ criar, carregar ou retomar sessão

checkpointEncounter
→ validar e persistir trecho intermediário

submitEncounterResolution
→ validar e concluir resolução consolidada

cancelSession
→ cancelar conforme política

loadGame
→ descobrir sessão ativa ou recuperável
```

Operações internas possíveis:

```text
CREATE
LOAD
RESUME
CHECKPOINT
SUBMIT
TRANSITION_PHASE
REPLAY_FROM_EVENT
CANCEL
```

Sessões de perícia, comércio, fabricação e outros domínios podem reutilizar os mesmos princípios de autoridade local, log, consequência, checkpoint e recuperação com seus contratos específicos.

## 21. Requisitos para o Codex

A implementação deve possuir:

1. agregados de sessão versionados;
2. schema do snapshot e da autoridade local;
3. fechamento automático de dependências;
4. validadores de prontidão;
5. log de eventos ordenado;
6. buffer de consequências;
7. idempotência por submissão e checkpoint;
8. validação de `stateVersion` e lease;
9. reprodução de cálculos;
10. suporte a ajustes não bloqueantes;
11. replay parcial;
12. retomada pelo último checkpoint;
13. resumo mecânico autoritativo;
14. resumo narrativo vinculado;
15. políticas de visibilidade;
16. testes que provem ausência de chamada por microação.

## 22. Testes de aceitação

1. Carregar combate com fechamento completo de dependências.
2. Rejeitar `COMBAT_READY` quando uma ação estiver incompleta.
3. Resolver dez eventos locais sem chamada intermediária.
4. Registrar rolagens d100 em ordem.
5. Consumir poção e impedir reaparecimento no saque.
6. Enviar checkpoint após rendição.
7. Transicionar de combate para negociação.
8. Ajustar diferença de arredondamento sem invalidar a sessão.
9. Rejeitar ação inexistente e solicitar replay do evento inválido.
10. Aceitar eventos válidos anteriores ao ponto de replay.
11. Pausar e retomar em outro chat pelo último checkpoint.
12. Preservar informação `MASTER_ONLY` sem revelar ao jogador.
13. Transformar detalhe narrativo em entidade mecânica somente quando necessário.
14. Detectar conflito de estado e oferecer recuperação.
15. Confirmar que nenhuma nova Action foi adicionada.

## 23. Pendências

- escolher modo inicial de aleatoriedade;
- definir limites máximos de eventos por submissão;
- definir tamanho máximo do log e política de compactação;
- definir frequência padrão de checkpoints por perfil;
- definir duração e renovação de lease;
- definir regras de merge para conflitos não sobrepostos;
- definir quais consequências exigem checkpoint imediato;
- definir retenção dos logs completos;
- definir formato final dos resumos;
- testar custo de tokens dos snapshots e deltas.

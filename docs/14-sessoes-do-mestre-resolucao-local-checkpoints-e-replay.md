# Orquestração do Mestre, Sessões, Checkpoints e Replay

**Versão da proposta:** `master-runtime-v0.2`  
**Status:** em validação

## 1. Objetivo

Definir como o GPT atua como Mestre narrativo enquanto widget e backend mantêm a experiência mecânica rápida e autoritativa.

A arquitetura principal não exige que o GPT calcule oficialmente cada ataque. O widget pode enviar comandos estruturados diretamente, e o backend resolve até a próxima decisão relevante.

O GPT continua responsável por intenção aberta, personalidade, relações, criação, improvisação e narração.

## 2. Fluxo principal

```text
backend prepara e valida a mesa
→ widget ou GPT envia intenção
→ backend resolve oficialmente
→ widget atualiza
→ GPT narra quando necessário
→ checkpoint e continuidade
```

## 3. Modos de resolução

### 3.1 `BACKEND_AUTHORITATIVE`

Modo principal:

- widget envia comando comum;
- GPT envia intenção criativa estruturada;
- backend calcula, rola, resolve e persiste;
- resposta pode conter vários eventos;
- resolução para na próxima decisão relevante.

### 3.2 `GPT_ORCHESTRATED`

O GPT decide intenções de alto nível, especialmente para:

- NPCs importantes;
- diálogo;
- negociação narrativa;
- relações;
- ações criativas;
- decisões morais;
- criação e revisão de conteúdo.

O backend permanece autoridade mecânica.

### 3.3 `LOCAL_AUTHORIZED_FALLBACK`

Modo opcional para experiência textual, simulação, teste ou contingência.

O GPT pode resolver localmente quando o snapshot declarar autoridade explícita e fornecer regras, rolagens e limites reproduzíveis.

Esse modo não é o caminho principal do widget.

## 4. Camadas do estado

```text
livro de regras
+ estado autoritativo
+ snapshot da sessão
+ estado visual do widget
+ log mecânico
+ resumo narrativo
+ consequências pendentes
```

## 5. Fechamento de dependências

Uma sessão pronta possui todas as dependências mecânicas necessárias:

```text
ator
→ atributos e recursos
→ ações e passivas
→ equipamento
→ ações concedidas
→ inventário relevante
→ condições
→ ambiente
→ regras e versões
```

Estados:

```text
COMPLETE
INCOMPLETE
INVALID
REQUIRES_MATERIALIZATION
REQUIRES_REVIEW
```

## 6. Perfis de sessão

```text
EXPLORATION
COMBAT
EVALUATION
NEGOTIATION
CHASE
INFILTRATION
HARVEST
RECRUITMENT
TAMING
TRADE
CRAFT
MIXED_ENCOUNTER
```

Cada perfil define:

- dados do snapshot;
- ferramentas e comandos permitidos;
- fronteira de resolução;
- transições;
- checkpoint;
- necessidade de narração;
- visibilidade.

## 7. Snapshot

```json
{
  "sessionId": "uuid",
  "profile": "COMBAT",
  "phase": "COMBAT",
  "lifecycleStatus": "ACTIVE",
  "baseStateVersion": 15,
  "snapshotVersion": 1,
  "ruleVersions": {},
  "dependencyClosure": {},
  "readiness": {},
  "participants": [],
  "environment": {},
  "eventQueue": [],
  "nextDecision": {},
  "pendingConsequences": [],
  "visibilityPolicy": {},
  "checkpointPolicy": {},
  "allowedPhaseTransitions": [],
  "lease": {}
}
```

## 8. Janela de decisão

O backend resolve eventos até uma fronteira:

```text
IMMEDIATE_RESULT
NEXT_ACTOR_READY
NEXT_PLAYER_DECISION
UNEXPECTED_EVENT
PHASE_TRANSITION
SESSION_COMPLETE
```

Para o widget, `NEXT_PLAYER_DECISION` é o padrão recomendado em combate.

## 9. Log mecânico

Cada evento oficial registra:

```text
sequence
time
actor
action and version
targets
rolls
costs
effects
state before and after
consequences
```

O backend produz ou confirma o log no fluxo principal.

O GPT recebe resumo suficiente para narrar sem precisar carregar todos os eventos detalhados.

## 10. Rolagens

Modos:

```text
BACKEND_OFFICIAL
SEEDED_SEQUENCE
PREGENERATED_ROLL_POOL
GPT_RECORDED_FALLBACK
```

### `BACKEND_OFFICIAL`

Padrão para widget e produção.

### `SEEDED_SEQUENCE`

Útil para reprodução, simulação e resolução em lote.

### `GPT_RECORDED_FALLBACK`

Somente quando o snapshot autorizar. O GPT registra sequência e não rerrola o mesmo evento.

## 11. Consequências

Tipos comuns:

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

O backend persiste consequências oficiais. Resultados narrativos propostos podem permanecer em buffer até validação.

## 12. Diretiva narrativa

```json
{
  "narrationDirective": {
    "level": "IMPORTANT",
    "reasonCodes": ["BATTLE_COMPLETE"],
    "publicFacts": [],
    "masterFacts": [],
    "followUpSuggested": true
  }
}
```

O GPT não precisa narrar `NONE` e pode resumir `COMPACT`.

## 13. Checkpoints

Ocorrer quando:

- fase termina;
- ator persistente morre, é capturado ou foge;
- missão ou relação muda de forma importante;
- jogador interrompe;
- sessão fica extensa;
- ocorre conflito;
- política exige;
- há transição para outro sistema.

O checkpoint persiste trecho aceito e retorna novo `stateVersion` e delta.

## 14. Transições

```text
EXPLORATION → COMBAT
EVALUATION → COMBAT
COMBAT → LOOT
COMBAT → CAPTURE
COMBAT → NEGOTIATION
COMBAT → ESCAPE
CONVERSATION → TRADE
CONVERSATION → MISSION
TAMING → COMPANION
```

O backend informa transições permitidas e requisitos.

## 15. Replay parcial

Quando um erro ocorrer em uma sequência:

```json
{
  "status": "REQUIRES_REPLAY",
  "acceptedThroughSequence": 12,
  "invalidFromSequence": 13,
  "recovery": {
    "tool": "loadGameContext",
    "command": "RELOAD_SESSION_FROM_CHECKPOINT"
  }
}
```

Eventos válidos anteriores devem ser preservados quando seguro.

## 16. Concorrência e lease

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

## 17. Resumo para continuidade

Persistir:

### Mecânico

```text
recursos alterados
itens consumidos
dano e condições
atores derrotados
loot formado
estado da sessão
```

### Narrativo

Resumo breve dos acontecimentos relevantes para retomada em outro chat.

## 18. Informação do Mestre

```text
MASTER_ONLY
PLAYER_KNOWN
PLAYER_VISIBLE
DISCOVERABLE
HIDDEN_UNTIL_TRIGGER
```

O widget não recebe dados secretos necessários apenas ao GPT/backend.

## 19. Detalhe narrativo versus entidade mecânica

```text
NARRATIVE_DETAIL
MECHANICAL_ENTITY
```

O GPT pode narrar decoração livremente. Quando o jogador tentar usar, coletar ou vender o detalhe, ele deve ser materializado ou declarado sem valor mecânico.

## 20. Estado visual

O widget pode manter:

- zoom;
- aba;
- filtro;
- alvo destacado;
- animação atual;
- seleção ainda não confirmada.

O backend mantém estado oficial.

## 21. Ferramentas reutilizadas

```text
loadGameContext
loadWidgetView
previewGameCommand
resolveGameCommand
resolveCreativeIntent
checkpointSession
finalizeSession
cancelSession
```

Nenhuma ferramenta exclusiva de microação é necessária.

## 22. Critérios de aceitação

1. Ataques comuns não exigem nova resposta do GPT.
2. Ação criativa continua possível pelo chat.
3. Backend produz resultado oficial reproduzível.
4. Widget atualiza antes da narração quando permitido.
5. GPT recebe fatos públicos e do Mestre separados.
6. Sessão pode ser retomada após remontagem.
7. Replay parcial preserva eventos válidos.
8. Modo local do GPT é opcional e explicitamente autorizado.

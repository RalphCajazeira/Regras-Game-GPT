# Integração entre ChatGPT App, Widget, MCP, Backend e Frontend

**Versão da proposta:** `integration-v1.0`  
**Status:** arquitetura canônica em validação

## 1. Objetivo

Permitir que o GPT conduza narrativa e decisões abertas, que o widget ofereça uma experiência rápida e visual e que o backend permaneça autoridade mecânica e persistente.

## 2. Arquitetura canônica

```text
Jogador
├── conversa com o GPT
└── interage com o Widget JS
          ↓
ChatGPT App/Plugin + Apps SDK
          ↓
MCP Tools e recursos de UI
          ↓
Serviços de aplicação e domínio
          ↓
Backend + PostgreSQL
```

O caminho antigo de Custom GPT + Actions REST/OpenAPI é legado/fallback. REST pode continuar existindo e deve reutilizar os mesmos serviços de aplicação.

## 3. Responsabilidades

### GPT

- compreender linguagem natural;
- estruturar intenções criativas;
- narrar cenas, relações e consequências;
- controlar personalidade e intenção de NPCs importantes;
- criar e revisar conteúdo por ferramentas válidas;
- separar fatos públicos e fatos do Mestre;
- solicitar materialização mecânica quando necessário.

### Widget

- exibir ficha, mapa, combate, inventário, loot, comércio e missões;
- coletar cliques e seleções;
- mostrar prévias versionadas;
- chamar ferramentas MCP para comandos comuns;
- animar eventos oficiais;
- solicitar narração apenas quando indicada;
- manter somente estado efêmero de interface.

### MCP Server

- publicar ferramentas tipadas;
- publicar recursos de widget;
- autenticar e autorizar;
- adaptar ferramentas aos serviços de aplicação;
- controlar `structuredContent`, `content` e `_meta`;
- impedir vazamento de informações secretas.

### Backend

- validar regras, estado, custos, alvos, alcance e permissões;
- calcular resultados oficiais;
- executar rolagens oficiais no fluxo principal;
- resolver NPCs genéricos e eventos automáticos;
- persistir estado e histórico;
- fornecer erros acionáveis;
- classificar necessidade de narração;
- controlar idempotência, versões e concorrência.

## 4. Dois fluxos de entrada

### 4.1 Widget

```text
clique
→ previewGameCommand
→ confirmação
→ resolveGameCommand
→ backend resolve oficialmente
→ widget atualiza
```

### 4.2 Chat

```text
mensagem livre
→ GPT interpreta
→ resolveCreativeIntent
→ backend valida e resolve
→ GPT narra
```

Ambos chegam ao mesmo domínio.

## 5. Versões de regras

```json
{
  "ruleVersions": {
    "attributes": "attributes-v0.4",
    "combat": "combat-v0.2",
    "equipment": "equipment-v0.5",
    "actions": "actions-v0.1",
    "economy": "economy-v0.3",
    "skills": "skills-v0.1",
    "crafting": "crafting-v0.2",
    "actors": "actors-v0.1",
    "inventory": "inventory-v0.2",
    "mcpContracts": "mcp-contracts-v0.1",
    "bundles": "bundles-v0.2",
    "masterRuntime": "master-runtime-v0.2",
    "appArchitecture": "app-architecture-v0.1",
    "widgetGameplay": "widget-gameplay-v0.1",
    "security": "security-v0.1",
    "integration": "integration-v1.0"
  }
}
```

Toda resolução, criação, materialização ou revisão informa as versões utilizadas.

## 6. Estado e identidade

Toda escrita persistente usa:

```text
requestId
idempotencyKey
baseStateVersion
ruleVersions
userContext derivado do token
```

UUID identifica entidades. Definições também possuem `code` e `version`. `clientRef` relaciona nós dentro de bundles.

## 7. Conteúdo e materialização em lote

O GPT pode enviar pacote completo contendo conteúdo novo, reutilizável e temporário.

Modos:

```text
VALIDATE_ONLY
MATERIALIZE_TEMPORARY
PERSIST_IF_VALID
PERSIST_REQUIRED
PROMOTE_TEMPORARY
```

Fluxo:

```text
montar bundle
→ resolver referências
→ validar todos os nós e vínculos
→ devolver todos os issues detectáveis
→ materializar temporariamente ou persistir atomicamente
```

## 8. Ferramentas MCP canônicas

Catálogo inicial:

```text
loadGameContext
searchGameContent
getGameContent
loadWidgetView
previewGameCommand
resolveGameCommand
resolveCreativeIntent
manageContentBundle
materializeActors
manageActor
manageRelationships
manageInventory
manageTrade
manageCraft
manageMission
advanceWorldTime
manageWorldState
checkpointSession
finalizeSession
cancelSession
```

O catálogo deve permanecer pequeno e semanticamente claro. Não adotar ferramenta genérica para alterar qualquer registro.

## 9. Resposta de ferramenta

```json
{
  "status": "SUCCESS",
  "requestId": "uuid",
  "previousStateVersion": 15,
  "newStateVersion": 16,
  "result": {},
  "structuredContent": {},
  "snapshotDelta": {},
  "narrationDirective": {},
  "warnings": [],
  "issues": [],
  "recoveryTools": [],
  "availableNextCommands": []
}
```

## 10. Prévia

O widget pode calcular ou solicitar prévia, mas:

- não altera estado;
- informa regras e stateVersion;
- declara hipóteses;
- é recalculada pelo backend na confirmação;
- pode divergir do resultado por rolagem, reação ou mudança de estado.

## 11. Resolução por janela de decisão

Uma chamada pode resolver múltiplos eventos até uma fronteira:

```text
IMMEDIATE_RESULT
NEXT_ACTOR_READY
NEXT_PLAYER_DECISION
UNEXPECTED_EVENT
PHASE_TRANSITION
SESSION_COMPLETE
```

Isso evita chamadas por ataque, reação, NPC ou tick.

## 12. Sessões e checkpoints

Sessões devem suportar:

- fechamento de dependências;
- prontidão;
- fases;
- log reproduzível;
- consequências pendentes;
- checkpoint;
- replay parcial;
- recuperação após remontagem;
- conclusão e cancelamento idempotentes.

## 13. Narração seletiva

O backend retorna:

```text
NONE
COMPACT
STANDARD
IMPORTANT
CRITICAL
PLAYER_DECISION_REQUIRED
```

O widget pode atualizar imediatamente e solicitar mensagem de acompanhamento apenas quando indicada.

## 14. Informação do Mestre

Classificação:

```text
MASTER_ONLY
PLAYER_KNOWN
PLAYER_VISIBLE
DISCOVERABLE
HIDDEN_UNTIL_TRIGGER
ADMIN_ONLY
```

Informações secretas não podem ser incluídas em dados renderizados ao jogador.

## 15. Mapas e combate

O widget oferece:

- mapa de exploração;
- passagem do tempo;
- mapa tático;
- clique para mover;
- planos curtos de ação;
- janela de decisão;
- animação de eventos;
- loot clicável;
- retorno ao mapa.

O motor mantém tempo contínuo em ticks; a interface apresenta turnos aparentes por decisão.

## 16. Autenticação

```text
OAuth/token
→ usuário interno
→ campanhas e atores autorizados
```

IDs enviados no payload não substituem autorização.

## 17. Recuperação

`loadGameContext` deve retornar:

```text
activeSessions
recoverableSessions
continuationSummary
requiredTool
requiredCommand
widgetView
```

## 18. Compatibilidade

O backend REST atual pode permanecer durante a migração e para clientes externos. O MCP Server deve usar os mesmos serviços, não chamar lógica duplicada.

## 19. Requisitos para o Codex

1. Auditar o projeto existente antes de migrar.
2. Separar domínio do transporte.
3. Implementar MCP e widgets incrementalmente.
4. Manter compatibilidade até o novo fluxo estar validado.
5. Proteger dados do Mestre.
6. Implementar autenticação e autorização.
7. Testar widget e GPT usando o mesmo domínio.
8. Implementar métricas e tracing.
9. Registrar commit e versões de regras implementados.
10. Seguir `docs/18-roadmap-de-implementacao-para-o-codex.md`.

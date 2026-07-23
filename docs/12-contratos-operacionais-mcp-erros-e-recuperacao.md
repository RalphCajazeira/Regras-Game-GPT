# Contratos Operacionais, Ferramentas MCP, Erros e Recuperação

**Versão da proposta:** `mcp-contracts-v0.1`  
**Status:** em validação

## 1. Objetivo

Definir como o ChatGPT, os widgets e o backend se comunicam por ferramentas MCP tipadas, sem duplicar regras de domínio e sem obrigar o modelo a adivinhar como continuar após erros.

Este documento substitui a fachada canônica anterior baseada em Custom GPT Actions/OpenAPI. REST continua permitido como interface interna ou pública do backend, mas a integração principal dentro do ChatGPT usa um App construído com Apps SDK e MCP.

## 2. Arquitetura canônica

```text
ChatGPT
├── GPT: intenção, criatividade, diálogo e narração
└── Widget JS: interface, prévias e comandos estruturados
            ↓
MCP Server / Apps SDK
            ↓
Serviços de aplicação e domínio
            ↓
Backend e PostgreSQL
```

Regras:

1. GPT e widget usam os mesmos serviços de aplicação.
2. Regras mecânicas não ficam duplicadas no widget, no prompt ou no servidor MCP.
3. O widget pode calcular prévias versionadas, mas o backend recalcula o resultado oficial.
4. O GPT não atualiza registros diretamente; ele chama ferramentas de domínio.
5. O MCP Server autentica o usuário e propaga sua identidade aos serviços.
6. Ferramentas devem possuir entradas e saídas explícitas e previsíveis.
7. Ferramentas de escrita devem preservar idempotência, concorrência e auditoria.
8. O catálogo permanece pequeno e focado mesmo sem adotar o antigo limite rígido de 30 Actions.

## 3. Duas entradas, um único domínio

### 3.1 Ação estruturada do widget

```text
jogador clica
→ widget monta comando tipado
→ ferramenta MCP
→ backend resolve oficialmente
→ widget atualiza
```

### 3.2 Intenção criativa pelo GPT

```text
jogador escreve livremente
→ GPT interpreta
→ GPT monta intenção estruturada
→ ferramenta MCP
→ backend valida e resolve
→ GPT narra
```

As duas entradas não possuem motores mecânicos diferentes.

## 4. Catálogo inicial de ferramentas MCP

O alvo inicial é manter entre 18 e 24 ferramentas focadas. Mudanças exigem revisão versionada, mas não existe a antiga regra canônica de 30 `operationId`s do OpenAPI.

### 4.1 Contexto e leitura

| Ferramenta | Finalidade |
|---|---|
| `loadGameContext` | Carregar campanha, personagem, sessões ativas e resumo de continuidade |
| `searchGameContent` | Pesquisar definições, versões, instâncias e conteúdos semelhantes |
| `getGameContent` | Obter conteúdo completo por UUID, código e versão |
| `loadWidgetView` | Carregar dados e recurso visual de ficha, mapa, inventário, combate, missão ou comércio |

### 4.2 Prévia e resolução

| Ferramenta | Finalidade |
|---|---|
| `previewGameCommand` | Produzir prévia sem alterar o estado oficial |
| `resolveGameCommand` | Resolver comando estruturado do widget até a fronteira solicitada |
| `resolveCreativeIntent` | Resolver intenção aberta estruturada pelo GPT |

### 4.3 Conteúdo e materialização

| Ferramenta | Finalidade |
|---|---|
| `manageContentBundle` | Validar, persistir, revisar ou promover pacote de conteúdo |
| `materializeActors` | Materializar um ou vários atores completos ou temporários |
| `manageActor` | Alterar estado, controle, facção, comportamento ou escopo |
| `manageRelationships` | Relações, memórias, grupo, recrutamento, romance e domesticação |

### 4.4 Sistemas de jogo

| Ferramenta | Finalidade |
|---|---|
| `manageInventory` | Equipar, transferir, consumir, saquear, coletar, reservar ou destruir |
| `manageTrade` | Carregar, prever e concluir compra, venda e negociação |
| `manageCraft` | Carregar, prever e concluir fabricação, reparo ou refinamento |
| `manageMission` | Criar, aceitar, atualizar, concluir, falhar ou abandonar missão |
| `advanceWorldTime` | Viagem, descanso, recuperação e atividades demoradas |
| `manageWorldState` | Locais, facções, reputação, crimes, eventos e consequências |

### 4.5 Sessões

| Ferramenta | Finalidade |
|---|---|
| `checkpointSession` | Persistir trecho validado de sessão longa |
| `finalizeSession` | Validar e concluir encontro ou outro fluxo complexo |
| `cancelSession` | Cancelar ou encerrar de forma segura e idempotente |

## 5. Ferramentas e widgets

Uma ferramenta pode retornar:

```text
structuredContent → dados para o modelo e para o widget
content           → mensagem textual resumida
_meta             → metadados exclusivos da interface
```

O widget deve receber apenas dados necessários à experiência visual. Informações secretas do Mestre não podem ser incluídas em campos expostos ao jogador.

Ferramentas que renderizam interface devem declarar o recurso de UI correspondente, por exemplo:

```text
ui://game/character-sheet
ui://game/world-map
ui://game/battle-map
ui://game/inventory
ui://game/trade
ui://game/crafting
ui://game/mission-log
```

## 6. Chamada direta pelo widget

O widget pode chamar ferramentas autorizadas para interações rápidas, como:

```text
selecionar destino
mover no mapa
usar ataque básico
usar consumível
selecionar loot
comparar equipamento
comprar ou vender
```

A chamada direta não elimina validação. Ela apenas evita uma nova rodada de interpretação pelo GPT para cada clique.

O widget não deve possuir ferramenta genérica capaz de alterar qualquer entidade.

## 7. Envelope comum de solicitação

Toda mutação persistente aceita, quando aplicável:

```json
{
  "command": "ENUM_EXPLICITO",
  "requestId": "uuid",
  "idempotencyKey": "uuid-ou-chave-estavel",
  "baseStateVersion": 15,
  "ruleVersions": {},
  "context": {},
  "payload": {}
}
```

`payload` deve usar union discriminada por `command`; JSON arbitrário não é permitido.

## 8. Envelope comum de resposta

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

Status canônicos:

```text
SUCCESS
PARTIAL_SUCCESS
REQUIRES_ACTION
REQUIRES_MATERIALIZATION
REQUIRES_REVIEW
REQUIRES_REPLAY
CONFLICT
REJECTED
CANCELLED
```

## 9. Diretiva de narração

O backend classifica o resultado:

```text
NONE
COMPACT
STANDARD
IMPORTANT
CRITICAL
PLAYER_DECISION_REQUIRED
```

Exemplo:

```json
{
  "narrationDirective": {
    "level": "IMPORTANT",
    "reasonCodes": ["BATTLE_COMPLETE", "BOSS_DEFEATED"],
    "publicFacts": [],
    "masterFacts": [],
    "followUpSuggested": true
  }
}
```

- `publicFacts` podem ser narrados ao jogador.
- `masterFacts` orientam o GPT, mas não devem ser revelados automaticamente.
- O widget pode atualizar a interface antes da narração.
- Quando necessário, o widget solicita uma mensagem de acompanhamento ao ChatGPT.

## 10. Erros acionáveis

Um erro precisa informar:

```text
code
severity
blocking
domain
path
ruleCode
message
expected
received
recoveryTools
```

Exemplo:

```json
{
  "code": "TARGET_OUT_OF_RANGE",
  "domain": "COMBAT",
  "path": "/payload/targetActorId",
  "message": "O alvo está a 5,4 m e a ação alcança 2 m.",
  "expected": { "maximumRangeMeters": 2 },
  "received": { "distanceMeters": 5.4 },
  "recoveryTools": [
    {
      "tool": "previewGameCommand",
      "command": "MOVE_INTO_RANGE"
    }
  ]
}
```

Não retornar somente `invalid request` ou `operation failed`.

## 11. Prévia versus resultado oficial

```text
prévia do widget
→ usa estado e regra versionados
→ não altera o estado
→ pode ser aproximada

confirmação
→ backend recarrega ou valida stateVersion
→ recalcula
→ executa rolagens oficiais
→ persiste
```

Toda prévia deve informar:

```text
previewOnly: true
ruleVersions
baseStateVersion
assumptions
estimatedRange
```

## 12. Idempotência e concorrência

Toda operação que cria ou transfere valor deve ser idempotente:

- itens;
- dinheiro;
- XP;
- fabricação;
- comércio;
- loot;
- progressão;
- missão;
- relações persistentes.

Quando `baseStateVersion` estiver desatualizada, retornar `CONFLICT` com recarga, reaplicação segura, merge compatível ou cancelamento.

## 13. Autenticação

A identidade vem do token autenticado.

```text
OAuth / token
→ subject autenticado
→ usuário interno
→ campanhas e personagens autorizados
```

`playerId`, `actorId` ou `campaignId` enviados no payload são seletores de recurso, não prova de identidade.

Cada ferramenta valida autorização no backend.

## 14. Permissões de escrita

Ferramentas devem ser classificadas como leitura, prévia ou escrita.

A interface deve deixar claras ações com consequência relevante, especialmente:

- compra e venda;
- descarte ou destruição;
- conclusão de missão;
- migração de conteúdo;
- promoção de ator;
- decisões irreversíveis.

## 15. Compatibilidade REST

O backend pode continuar expondo REST para:

- frontend externo;
- painel administrativo;
- integrações internas;
- testes;
- compatibilidade com o protótipo antigo.

MCP e REST chamam os mesmos serviços de aplicação. Não duplicar regras em controladores e ferramentas.

## 16. Requisitos para o Codex

1. Criar módulo separado para o MCP Server/Apps SDK.
2. Preservar serviços de domínio independentes do transporte.
3. Implementar ferramentas com schemas tipados e descrições claras.
4. Separar ferramentas de leitura e escrita.
5. Implementar recursos de widget versionados.
6. Implementar `structuredContent`, `content` e `_meta` sem vazar segredos.
7. Implementar OAuth e autorização por recurso.
8. Implementar `stateVersion`, idempotência e erros acionáveis.
9. Criar testes de seleção de ferramentas pelo GPT.
10. Criar testes de chamadas diretas do widget.
11. Medir tamanho dos resultados e uso de contexto.
12. Manter catálogo pequeno e revisar sobreposição semântica.

## 17. Referências oficiais

- https://developers.openai.com/apps-sdk/
- https://developers.openai.com/apps-sdk/plan/tools/
- https://developers.openai.com/apps-sdk/build/chatgpt-ui/
- https://developers.openai.com/apps-sdk/build/state-management/
- https://developers.openai.com/apps-sdk/build/auth/

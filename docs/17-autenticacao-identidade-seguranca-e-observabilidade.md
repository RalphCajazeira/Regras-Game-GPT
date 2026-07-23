# Autenticação, Identidade, Segurança e Observabilidade

**Versão da proposta:** `security-v0.1`  
**Status:** em validação

## 1. Objetivo

Definir como cada jogador é identificado, autorizado e auditado na arquitetura ChatGPT App + Widget + MCP + Backend.

## 2. Regra central de identidade

```text
OAuth/token autenticado
→ subject do provedor
→ usuário interno
→ campanhas, personagens e recursos autorizados
```

O backend nunca confia em um `playerId`, `actorId` ou `campaignId` enviado isoladamente pelo GPT ou widget como prova de identidade.

## 3. Identificadores

```text
userId        → UUID interno do usuário
externalSub   → subject do provedor autenticado
actorId       → personagem ou ator selecionado
campaignId    → campanha selecionada
sessionId     → sessão de jogo
```

Todos os recursos devem possuir escopo de acesso explícito.

## 4. Autorização

Antes de qualquer leitura ou escrita, validar:

- usuário autenticado;
- posse ou participação na campanha;
- permissão sobre o ator;
- função no grupo;
- política do recurso;
- escopo da ferramenta;
- estado da sessão;
- restrições administrativas.

## 5. Categorias de ferramentas

```text
READ_ONLY
PREVIEW
WRITE_REVERSIBLE
WRITE_IMPORTANT
WRITE_IRREVERSIBLE
ADMINISTRATIVE
```

Exemplos:

| Ferramenta/comando | Categoria |
|---|---|
| carregar ficha | `READ_ONLY` |
| prévia de ataque | `PREVIEW` |
| mover no combate | `WRITE_REVERSIBLE` dentro da sessão |
| comprar item | `WRITE_IMPORTANT` |
| destruir item único | `WRITE_IRREVERSIBLE` |
| migrar definição | `ADMINISTRATIVE` |

## 6. Dados secretos

Informações como as seguintes não devem chegar ao widget do jogador sem autorização:

- inventário oculto de NPC;
- objetivos secretos;
- armadilhas não descobertas;
- identidade real desconhecida;
- rolagens secretas;
- resultados futuros de eventos;
- dados administrativos;
- segredos de autenticação;
- tokens e credenciais.

Usar classificação:

```text
MASTER_ONLY
PLAYER_KNOWN
PLAYER_VISIBLE
DISCOVERABLE
HIDDEN_UNTIL_TRIGGER
ADMIN_ONLY
```

## 7. Separação de resultados MCP

```text
structuredContent → somente dados necessários ao modelo e widget
content           → resumo textual seguro
_meta             → metadados de UI que não precisam entrar no contexto do modelo
```

Mesmo `_meta` não deve carregar credenciais ou segredos reutilizáveis.

## 8. Estado e concorrência

Toda mutação crítica inclui:

```text
requestId
idempotencyKey
baseStateVersion
ruleVersions
userContext derivado do token
```

Conflitos retornam:

```text
CONFLICT
RELOAD_REQUIRED
SAFE_RETRY
REPLAY_REQUIRED
CANCEL_AVAILABLE
```

## 9. Auditoria

Registrar:

- usuário autenticado;
- ferramenta e comando;
- origem: GPT, widget, frontend ou administração;
- campanha e sessão;
- requestId;
- idempotencyKey;
- versão anterior e posterior;
- regras usadas;
- conteúdo criado, reutilizado ou alterado;
- resultado;
- issues e recovery;
- duração;
- classificação narrativa;
- confirmação do usuário quando aplicável.

## 10. Observabilidade

Métricas mínimas:

```text
tool_call_count
tool_success_rate
tool_error_rate
recovery_success_rate
average_tool_latency
average_payload_size
average_structured_content_size
widget_render_failures
state_conflicts
idempotency_replays
session_recovery_rate
narration_requests
```

Logs devem permitir rastrear um fluxo completo por `requestId`, `sessionId` e `traceId`.

## 11. Privacidade

- armazenar apenas dados necessários;
- separar logs técnicos de conteúdo narrativo sensível;
- definir retenção;
- permitir exclusão de conta e campanhas quando aplicável;
- não registrar tokens;
- mascarar dados sensíveis em logs;
- documentar política de privacidade do App.

## 12. Falhas de autenticação

Erros canônicos:

```text
AUTHENTICATION_REQUIRED
TOKEN_EXPIRED
INSUFFICIENT_SCOPE
RESOURCE_FORBIDDEN
ACTOR_NOT_OWNED
CAMPAIGN_ACCESS_DENIED
ADMIN_PERMISSION_REQUIRED
```

A resposta deve indicar como reconectar ou selecionar um recurso permitido sem revelar dados de terceiros.

## 13. Continuidade entre dispositivos e conversas

O estado oficial pertence ao backend. Um usuário autenticado pode abrir outra conversa ou dispositivo e carregar:

- campanhas disponíveis;
- personagem ativo;
- sessão interrompida;
- último checkpoint;
- resumo de continuidade;
- widget correspondente.

## 14. Ambientes

Separar:

```text
local
staging
production
```

Tokens, usuários, bancos, URLs e dados de teste não devem ser compartilhados indevidamente entre ambientes.

## 15. Segurança de conteúdo criado pelo GPT

Conteúdo proposto pelo GPT passa por:

- schema;
- orçamento;
- referências;
- permissões;
- coerência temática;
- impacto;
- duplicação semântica;
- política do mundo;
- escopo de persistência.

O GPT não recebe acesso direto ao banco.

## 16. Requisitos para o Codex

1. Implementar OAuth/token conforme contrato atual da plataforma.
2. Derivar usuário do token no MCP Server.
3. Propagar contexto autenticado sem aceitar spoofing de IDs.
4. Criar middleware de autorização por recurso.
5. Classificar ferramentas e comandos por risco.
6. Proteger dados `MASTER_ONLY` e `ADMIN_ONLY`.
7. Criar auditoria estruturada.
8. Criar tracing ponta a ponta.
9. Mascarar segredos e dados sensíveis.
10. Testar acesso cruzado entre usuários.
11. Testar retomada em outro dispositivo/conversa.
12. Documentar política de privacidade e retenção.

## 17. Critérios de aceitação

- usuário A não acessa recursos do usuário B;
- trocar `actorId` no payload não contorna autorização;
- ferramenta de leitura não executa escrita;
- repetição idempotente não duplica valor;
- dados secretos não aparecem no widget;
- logs permitem reconstruir o fluxo sem expor tokens;
- sessão pode ser retomada após nova autenticação válida.

## 18. Referência oficial

- https://developers.openai.com/apps-sdk/build/auth/

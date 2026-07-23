# Arquitetura do ChatGPT App, Widget, MCP e Backend

**Versão da proposta:** `app-architecture-v0.1`  
**Status:** arquitetura canônica em validação

## 1. Objetivo

Definir a experiência principal do jogo dentro do ChatGPT, combinando liberdade narrativa, interface interativa e resolução mecânica autoritativa.

```text
GPT: compreensão da intenção, criatividade, diálogo e narração
Widget JS: interface visual, prévias e interações rápidas
MCP Server: ponte tipada entre ChatGPT e os serviços do jogo
Backend: regras oficiais, validação, resolução e persistência
Banco: estado durável, histórico, definições e instâncias
```

## 2. Decisão arquitetural

A arquitetura principal passa a ser um ChatGPT App construído com Apps SDK e MCP.

O caminho antigo de Custom GPT com Actions REST/OpenAPI deixa de ser a fachada canônica. Ele pode permanecer como fallback textual, compatibilidade temporária ou ferramenta de administração, desde que use os mesmos serviços de domínio.

## 3. Componentes

### 3.1 GPT — Mestre narrativo

Responsabilidades:

- compreender linguagem natural;
- transformar ações criativas em intenções estruturadas;
- narrar cenas, diálogos, relações e consequências;
- interpretar personalidade e objetivos de NPCs importantes;
- criar ou revisar conteúdo por ferramentas válidas;
- apresentar escolhas narrativas;
- separar informação do Mestre e informação do jogador;
- solicitar materialização quando um detalhe narrativo se tornar mecânico.

O GPT pode raciocinar e apresentar estimativas, mas não altera sozinho o estado oficial.

### 3.2 Widget JS — mesa visual

Responsabilidades:

- exibir ficha, recursos, inventário e equipamentos;
- exibir mapa do mundo e mapa tático;
- permitir seleção de destino, alvo, ação e itens;
- mostrar alcance, caminho, área de efeito e cobertura;
- calcular ou exibir prévias instantâneas versionadas;
- chamar ferramentas MCP para ações comuns;
- atualizar a interface com o resultado oficial;
- solicitar narração ao ChatGPT quando o backend classificar o evento como relevante;
- manter somente estado efêmero de interface.

Estado visual permitido:

```text
aba selecionada
zoom
painel aberto
filtro
alvo destacado
caminho em prévia
item comparado
```

Estado oficial proibido no widget:

```text
Vida oficial
saldo oficial
inventário oficial
resultado de rolagem
estado de missão
posição autoritativa
```

O widget pode manter cópia de visualização, mas o backend permanece autoridade.

### 3.3 MCP Server — ponte do App

Responsabilidades:

- publicar ferramentas tipadas;
- fornecer recursos de UI;
- validar autenticação;
- adaptar chamadas do GPT e widget aos serviços de aplicação;
- retornar `structuredContent`, conteúdo textual e metadados de UI;
- não conter regras de negócio duplicadas;
- controlar exposição de dados secretos.

### 3.4 Backend — autoridade mecânica

Responsabilidades:

- validar regras, permissões, custos, alvos, alcance e estado;
- calcular atributos, dano, cura, tempo e qualidade;
- executar rolagens oficiais no fluxo principal;
- resolver ações de NPCs genéricos conforme comportamento;
- resolver comandos até a próxima decisão relevante;
- persistir estado, eventos e histórico;
- fornecer erros acionáveis;
- classificar necessidade de narração;
- garantir idempotência e concorrência.

### 3.5 Banco de dados

Responsabilidades:

- persistir definições, versões e instâncias;
- persistir personagens, campanhas e mundo;
- persistir inventário, comércio e fabricação;
- persistir encontros, eventos e checkpoints;
- persistir relações, memórias, missões e reputação;
- manter rastreabilidade e auditoria.

## 4. Fluxos principais

### 4.1 Ação comum pelo widget

```text
jogador clica
→ widget mostra prévia
→ jogador confirma
→ widget chama ferramenta MCP
→ backend valida, rola, resolve e persiste
→ widget anima e atualiza
→ narração somente quando necessária
```

Exemplos:

- ataque básico;
- movimento;
- uso de poção;
- equipar item;
- coletar loot;
- compra ou venda simples;
- selecionar destino de viagem.

### 4.2 Ação criativa pelo chat

```text
jogador escreve
→ GPT interpreta
→ GPT consulta estado e regras necessários
→ GPT cria intenção estruturada
→ ferramenta MCP
→ backend valida e resolve
→ widget representa o que for visualizável
→ GPT narra
```

### 4.3 Evento importante

```text
backend resolve
→ devolve narrationDirective
→ widget atualiza imediatamente
→ ChatGPT recebe fatos públicos e fatos do Mestre
→ GPT narra ou pede decisão
```

## 5. Um único motor de domínio

As entradas abaixo devem chegar ao mesmo serviço de aplicação:

```text
clique em Ataque Básico
mensagem “ataco o bandido”
mensagem criativa com movimento e ataque
frontend externo futuro
painel administrativo
```

Não implementar regras separadas para widget e GPT.

## 6. Fronteira de resolução

Uma chamada pode resolver mais que uma microação.

Fronteiras canônicas:

```text
IMMEDIATE_RESULT
NEXT_ACTOR_READY
NEXT_PLAYER_DECISION
UNEXPECTED_EVENT
PHASE_TRANSITION
SESSION_COMPLETE
```

Fluxo recomendado para combate:

```text
jogador confirma plano
→ backend agenda e resolve eventos
→ NPCs e reações agem quando aplicável
→ backend para na próxima decisão relevante
→ widget apresenta o novo estado
```

## 7. NPCs

### 7.1 NPCs genéricos

O backend pode selecionar ações por perfis:

```text
AGGRESSIVE_MELEE
RANGED_KITER
PACK_HUNTER
PROTECT_LEADER
COWARDLY
DEFENSIVE
HEALER_SUPPORT
```

### 7.2 NPCs importantes

O GPT define a intenção narrativa de alto nível:

```text
proteger o aliado
fugir sem abandonar o artefato
negociar antes de matar
manter segredo
```

O backend escolhe ou valida ações mecânicas compatíveis.

```text
GPT decide por que o NPC age.
Backend confirma como ele pode agir mecanicamente.
```

## 8. Classificação narrativa

```text
NONE
COMPACT
STANDARD
IMPORTANT
CRITICAL
PLAYER_DECISION_REQUIRED
```

Eventos típicos:

| Evento | Classificação sugerida |
|---|---|
| movimento comum | `NONE` |
| ataque básico sem destaque | `COMPACT` |
| uso normal de consumível | `COMPACT` |
| crítico | `IMPORTANT` |
| inimigo derrotado | `IMPORTANT` |
| mudança de fase | `CRITICAL` |
| rendição | `PLAYER_DECISION_REQUIRED` |
| escolha irreversível | `PLAYER_DECISION_REQUIRED` |
| fim de combate | `IMPORTANT` ou `CRITICAL` |

## 9. Estado e sincronização

Toda escrita deve carregar:

```text
baseStateVersion
idempotencyKey
ruleVersions
```

O widget pode animar com otimismo apenas quando existir política explícita de rollback. O padrão é atualizar oficialmente depois da resposta do backend.

## 10. Autenticação

- autenticação por OAuth/token;
- identidade derivada do token;
- autorização por campanha, ator e recurso;
- nunca confiar em `playerId` isolado do payload;
- ferramentas de escrita validam posse e permissão;
- segredos não entram em `structuredContent` visível ao modelo ou widget.

## 11. Continuidade

Ao abrir ou remontar o widget:

```text
loadGameContext
→ identifica sessão ativa
→ loadWidgetView
→ restaura modo EXPLORATION, COMBAT, TRADE ou outro
```

O widget nunca é a única cópia do estado.

## 12. Frontend externo

O mesmo backend poderá atender futuramente:

- site próprio;
- aplicativo móvel;
- painel administrativo;
- ferramentas de criação de conteúdo;
- observabilidade e suporte.

O frontend externo reutiliza os mesmos serviços e contratos de domínio.

## 13. Migração da arquitetura anterior

Preservar:

- regras de domínio;
- serviços existentes;
- schemas Zod reutilizáveis;
- idempotência;
- `stateVersion`;
- banco e migrations compatíveis;
- eventos, snapshots e checkpoints;
- REST interno útil.

Substituir como fachada principal:

```text
Custom GPT Actions/OpenAPI
→ ChatGPT App/Plugin + MCP Tools + Widget
```

## 14. Critérios de aceitação

1. Uma ação comum pode ser concluída pelo widget sem uma nova interpretação do GPT.
2. Uma ação criativa pode ser interpretada pelo GPT e resolvida pelo mesmo backend.
3. O widget nunca confirma resultado oficial usando apenas cálculo local.
4. O backend retorna diretiva narrativa.
5. O GPT recebe apenas fatos permitidos para narração.
6. Uma sessão pode ser recarregada após remontagem do widget.
7. O mesmo comando repetido não duplica consequência.
8. O frontend externo futuro não exige reescrever regras.

## 15. Referências oficiais

- https://developers.openai.com/apps-sdk/
- https://developers.openai.com/apps-sdk/build/chatgpt-ui/
- https://developers.openai.com/apps-sdk/build/state-management/
- https://developers.openai.com/apps-sdk/build/auth/

# Arquitetura GPT, Extensão e Backend

**Versão:** `extension-architecture-v0.1`  
**Status:** canônica

## 1. Objetivo

Definir a arquitetura principal de **Crônicas de Outro Mundo** após a validação do ChatGPT App/widget e a decisão de adotar uma extensão Chromium como frontend persistente do jogador.

A mudança não descarta o trabalho realizado no widget. Ela altera a superfície principal de experiência, preservando backend, OAuth, MCP, projeções, sessões, idempotência e eventos já implementados.

## 2. Arquitetura canônica

```text
GPT personalizado
├─ Instructions
├─ Knowledge
├─ App MCP
├─ interpretação criativa
├─ diálogos
└─ narração

Extensão Chromium Manifest V3
├─ botão flutuante no ChatGPT web
├─ overlay/full-page
├─ página própria em aba
├─ ficha
├─ inventário
├─ equipamentos
├─ habilidades
├─ mapa
├─ combate
├─ comércio
├─ treinamento
└─ demais controles do jogador

Backend
├─ identidade
├─ autorização
├─ regras oficiais
├─ resolução
├─ idempotência
├─ stateVersion
├─ persistência
├─ REST para a extensão
├─ MCP para o GPT
└─ notificações de atualização

PostgreSQL
├─ estado oficial
├─ eventos
├─ histórico
├─ sessões
├─ definições
└─ instâncias

Widget do ChatGPT
├─ fallback leve
├─ bootstrap
├─ diagnóstico
└─ compatibilidade
```

## 3. GPT personalizado definitivo

O GPT definitivo deve usar:

- Instructions;
- Knowledge;
- App MCP.

Ele não deve manter Actions/OpenAPI configuradas simultaneamente. As Actions podem permanecer no backend e no repositório como legado durante a migração, mas serão removidas da configuração do GPT final.

Responsabilidades do GPT:

- compreender intenção aberta;
- criar e adaptar conteúdo narrativo;
- conduzir NPCs importantes;
- narrar resultados oficiais;
- escolher a tool MCP correta;
- pedir esclarecimento somente quando necessário;
- nunca substituir o backend como autoridade mecânica.

## 4. Extensão como frontend principal

A extensão deve oferecer a experiência persistente que o widget não consegue garantir de forma satisfatória.

### 4.1 Modos

```text
MINIMIZED
→ botão flutuante

OVERLAY
→ interface ocupa toda a área visível da página do ChatGPT

PAGE
→ mesma interface em página própria da extensão
```

Um Side Panel pode ser adicionado futuramente como modo opcional, mas não é a base obrigatória.

### 4.2 Regras de integração com o ChatGPT

A extensão não deve:

- copiar o histórico da conversa;
- observar continuamente o transcript;
- capturar cookies ou tokens;
- chamar APIs internas não documentadas;
- reconstruir um segundo chat;
- automatizar o composer como dependência arquitetural.

O ChatGPT continua sendo a superfície oficial da conversa. A extensão apresenta e opera o jogo.

### 4.3 Estado visual

A extensão pode manter localmente:

- modo aberto;
- aba ativa;
- posição do botão;
- filtros;
- seleção visual;
- preferências de tema e animação;
- cache efêmero de consultas.

Ela não pode considerar localmente como oficial:

- HP;
- Mana;
- SP;
- inventário;
- equipamentos;
- XP;
- localização;
- encontro;
- tempo do mundo;
- qualquer consequência mecânica.

## 5. Backend como autoridade

Toda operação oficial deve passar pelo backend.

Requisitos mínimos de mutação:

- identidade derivada do token;
- autorização por campanha e ator;
- `stateVersion`;
- `idempotencyKey`;
- transação;
- projeção pública allowlisted;
- auditoria sanitizada;
- proteção de dados `MASTER_ONLY`.

A extensão e o GPT reutilizam os mesmos serviços de domínio. Não devem existir regras divergentes por transporte.

## 6. REST e MCP

```text
Extensão → REST autenticado → serviços de aplicação
GPT      → MCP autenticado  → mesmos serviços de aplicação
```

REST não é legado quando utilizado como transporte oficial da extensão. O legado é a dependência do Custom GPT em Actions/OpenAPI.

As fachadas devem compartilhar:

- DTOs públicos;
- schemas;
- reason codes;
- idempotência;
- autorização;
- observabilidade;
- testes de contrato.

## 7. Autenticação da extensão

A extensão terá uma sessão própria, vinculada ao mesmo usuário interno utilizado pelo App MCP.

Ela não reutiliza:

- cookie do ChatGPT;
- token do App MCP;
- token capturado da página;
- credencial armazenada em código.

Fluxo-alvo:

```text
extensão inicia OAuth
→ provedor confirma identidade
→ backend vincula ExternalIdentity ao User
→ extensão recebe sessão própria
→ REST e realtime usam essa identidade
```

Segredos não entram no manifest.

## 8. Atualização automática

O realtime é um canal de invalidação, não a fonte de verdade.

```text
backend confirma transação
→ publica evento pequeno
→ extensão recebe tipo + versão + áreas alteradas
→ extensão invalida consultas
→ extensão carrega novamente projeções oficiais
```

Exemplo conceitual:

```json
{
  "type": "game.session.updated",
  "sessionVersion": 18,
  "changed": ["resources", "inventory", "activeEncounter"]
}
```

O canal não deve enviar payload completo quando uma invalidação for suficiente.

### 8.1 Estratégia incremental

1. recarga após mutação;
2. polling controlado como fallback;
3. SSE quando comunicação servidor → cliente for suficiente;
4. WebSocket quando mapa, combate ou múltiplos participantes exigirem comunicação mais frequente.

Após reconexão, a extensão sempre recarrega o estado oficial.

## 9. Fluxos operacionais

### 9.1 Ação pela extensão

```text
jogador clica
→ extensão valida input visual
→ backend autoriza e resolve
→ PostgreSQL confirma
→ resposta oficial retorna
→ extensão atualiza
→ realtime notifica outras superfícies
→ GPT narra somente quando necessário
```

### 9.2 Ação pelo GPT

```text
jogador escreve ao GPT
→ GPT interpreta
→ App MCP chama backend
→ backend persiste
→ realtime notifica extensão
→ extensão atualiza automaticamente
→ GPT narra o resultado oficial
```

### 9.3 Conflito

```text
baseVersion obsoleta
→ backend retorna conflito
→ extensão recarrega
→ jogador revisa antes de reenviar
```

## 10. Widget como fallback

O widget já validou:

- OAuth;
- identidade;
- autorização;
- GameSession;
- ficha;
- inventário;
- equipamentos;
- habilidades;
- ação persistente;
- idempotência;
- remontagem.

Ele permanece útil para:

- fallback sem extensão;
- diagnóstico;
- bootstrap;
- testes do App MCP;
- compatibilidade temporária.

Não deve receber novas evoluções grandes, salvo correção de segurança ou regressão compartilhada com o backend.

## 11. Segurança da extensão

Requisitos:

- Manifest V3;
- CSP sem código remoto;
- permissões mínimas;
- host permissions restritas;
- Shadow DOM para isolamento do overlay;
- mensagens internas tipadas;
- nenhuma leitura do chat;
- nenhuma captura de token;
- nenhuma regra mecânica no cliente;
- sanitização de conteúdo;
- proteção contra duplicação de injeção;
- tratamento de foco e acessibilidade.

## 12. Testes

Cada fase deve cobrir:

- contratos públicos;
- autenticação;
- autorização cruzada;
- expiração e revogação;
- estado vazio;
- conflito de versão;
- idempotência;
- reconexão;
- atualização por invalidação;
- ausência de dados `MASTER_ONLY`;
- overlay e página própria;
- largura reduzida e tela ampla;
- teclado e foco;
- regressão do backend/MCP.

## 13. Migração

A migração é incremental:

```text
widget validado
→ fundação da extensão
→ OAuth da extensão
→ leitura real
→ atualização automática
→ primeira ação real
→ demais sistemas jogáveis
→ GPT definitivo sem Actions
```

Não reescrever serviços de domínio apenas para atender a nova interface.

## 14. Decisões finais

- extensão overlay/full-page é o frontend principal;
- GPT personalizado é Mestre, intérprete e narrador;
- App MCP é a ponte oficial do GPT;
- REST autenticado é a ponte oficial da extensão;
- backend e PostgreSQL são autoridade;
- realtime invalida, não substitui leitura oficial;
- widget é fallback;
- Actions/OpenAPI não farão parte do GPT definitivo;
- a conversa do ChatGPT não será espelhada pela extensão.
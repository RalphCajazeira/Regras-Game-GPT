# Roadmap de Implementação para o Codex

**Versão da proposta:** `implementation-roadmap-v0.1`  
**Status:** pronto para auditoria técnica

## 1. Objetivo

Orientar o Codex a transformar as regras do repositório em uma ChatGPT App com widget, MCP e backend autoritativo, reaproveitando o projeto `cronicas-de-outro-mundo` existente.

O Codex não deve assumir que o sistema começa do zero.

## 2. Fontes canônicas

Antes de qualquer alteração:

1. ler este repositório;
2. registrar commit SHA das regras;
3. listar versões implementadas;
4. auditar o repositório do jogo;
5. comparar regra, código, banco, OpenAPI, testes e ambientes;
6. preservar funcionalidades já corretas.

## 3. Princípio de migração

```text
serviços de domínio existentes
→ adaptar e ampliar
→ criar MCP Server e widgets sobre os mesmos serviços
→ manter REST útil internamente
→ retirar dependência da fachada Custom GPT Actions
```

Evitar reescrita ampla sem evidência técnica.

## 4. Fase 0 — Auditoria e plano

Entregáveis:

- mapa do repositório atual;
- modelos Prisma e relações existentes;
- serviços reaproveitáveis;
- endpoints e Actions atuais;
- lacunas contra as regras;
- riscos de migrations;
- plano expand/contract;
- roadmap em PRs pequenos;
- primeiro recorte vertical;
- matriz de testes.

Não executar migração destrutiva antes de concluir a auditoria.

## 5. Fase 1 — Fronteira de domínio e contratos

Objetivos:

- separar serviços de aplicação dos controllers;
- criar schemas compartilhados;
- consolidar UUID, `code`, `version`, `clientRef`;
- implementar issues estruturadas;
- implementar `stateVersion` e idempotência;
- implementar readiness e dependency closure;
- implementar bundles transacionais.

Critérios:

- ator nível inválido retorna todos os problemas detectáveis;
- pacote válido cria ou materializa tudo sem chamadas individuais;
- rollback total em erro bloqueante;
- conteúdo existente é reutilizado.

## 6. Fase 2 — MCP Server e Apps SDK

Objetivos:

- criar pacote ou aplicação `mcp-server`;
- publicar ferramentas canônicas;
- implementar schemas tipados;
- mapear ferramentas para serviços de aplicação;
- publicar recursos de UI iniciais;
- implementar respostas com `structuredContent`, `content` e `_meta`;
- criar harness de teste de ferramentas.

Primeiras ferramentas:

```text
loadGameContext
searchGameContent
getGameContent
loadWidgetView
previewGameCommand
resolveGameCommand
manageContentBundle
materializeActors
checkpointSession
finalizeSession
cancelSession
```

## 7. Fase 3 — Autenticação e isolamento

Objetivos:

- OAuth/token;
- vínculo do subject com usuário interno;
- autorização por campanha e ator;
- proteção de dados do Mestre;
- auditoria e tracing;
- ambientes local, staging e produção.

Testes:

- acesso cruzado negado;
- spoofing de actorId negado;
- token expirado recuperável;
- logs sem credenciais.

## 8. Fase 4 — Widget somente leitura

Primeira UI:

- ficha;
- recursos;
- inventário;
- equipamentos;
- missões resumidas;
- sessão ativa;
- recarregamento após remontagem.

O widget ainda não executa escrita nesta fase.

## 9. Fase 5 — Mapa e passagem do tempo

Objetivos:

- mapa simples com dois ou mais pontos;
- rota e prévia de viagem;
- animação de posição;
- `worldTime` oficial;
- evento durante deslocamento;
- retorno de diretiva narrativa.

Critérios:

- widget não altera localização sozinho;
- conflito de stateVersion é recuperável;
- viagem repetida com mesma idempotencyKey não duplica tempo.

## 10. Fase 6 — Combate visual mínimo

Objetivos:

- mapa tático 2D simples;
- jogador e um inimigo;
- posições em metros;
- clique para mover;
- ataque básico;
- janela de decisão;
- resolução até `NEXT_PLAYER_DECISION`;
- animação por eventos;
- fim de combate;
- recuperação da sessão.

Não implementar inicialmente todos os tipos de magia ou condição.

## 11. Fase 7 — Loot e inventário interativo

Objetivos:

- corpo/contêiner clicável;
- itens reais do inimigo;
- `TAKE_ALL` e seleção;
- transferência atômica;
- comparação e equipar;
- peso e slots;
- consumidos não reaparecem.

## 12. Fase 8 — Fluxo criativo pelo GPT

Objetivos:

- GPT transforma linguagem natural em intenção estruturada;
- `resolveCreativeIntent` usa o mesmo domínio;
- materialização de objetos necessários;
- narração de resultados;
- separação entre fatos públicos e do Mestre;
- ações criativas podem interromper o fluxo visual padrão.

## 13. Fase 9 — Narração seletiva

Objetivos:

- `narrationDirective`;
- níveis `NONE` a `PLAYER_DECISION_REQUIRED`;
- resumos desde a última narração;
- solicitação de mensagem de acompanhamento;
- não poluir contexto com cada clique.

## 14. Fase 10 — Comércio e fabricação no widget

Objetivos:

- estoque e saldo;
- prévias de compra e venda;
- confirmação oficial;
- receita e materiais;
- chance de sucesso e qualidade;
- criação de instância;
- narração apenas em resultados relevantes.

## 15. Fase 11 — Relações, companheiros e missões

Objetivos:

- painel de relações;
- memórias e impressões;
- recrutamento e domesticação;
- missão e objetivos;
- progressão;
- escolhas narrativas.

Implementar após os respectivos documentos de regras estarem completos.

## 16. Fase 12 — Administração e frontend externo

Objetivos:

- revisar definições e instâncias;
- histórico e versões;
- migrações explícitas;
- dashboard de sessões e erros;
- reutilizar serviços do App;
- avaliar GraphQL somente para consultas flexíveis do frontend.

## 17. Estratégia de branches e PRs

- trabalhar sobre `develop` do jogo;
- uma fase ou subfase por branch curta;
- PRs pequenos e testáveis;
- migrations expand/contract;
- não remover compatibilidade antes do novo fluxo estar validado;
- registrar regra/commit implementado em cada PR.

## 18. Testes obrigatórios

### Unidade

- fórmulas;
- validadores;
- autorização;
- idempotência;
- resolução de comandos;
- classificação narrativa.

### Integração

- MCP → serviço → banco;
- bundle atômico;
- mapa e viagem;
- combate e loot;
- recuperação de sessão;
- widget chamando ferramenta.

### E2E

```text
login
→ carregar campanha
→ abrir mapa
→ viajar
→ iniciar combate
→ mover
→ atacar
→ derrotar
→ coletar loot
→ receber narração
→ reabrir sessão
```

## 19. Critérios para não avançar

Não avançar de fase quando:

- testes críticos falharem;
- migrations não tiverem rollback ou plano de recuperação;
- widget e backend divergirem em stateVersion;
- dados do Mestre vazarem;
- domínio estiver duplicado no widget;
- uma operação não for idempotente;
- sessão não puder ser retomada.

## 20. Primeiro prompt recomendado ao Codex

```text
Audite o repositório RalphCajazeira/cronicas-de-outro-mundo na branch develop contra o commit atual de RalphCajazeira/Regras-Game-GPT.

Considere como arquitetura canônica ChatGPT App/Plugin + Apps SDK + MCP Tools + Widget JS + serviços de domínio + backend/PostgreSQL. O caminho antigo de Custom GPT Actions/OpenAPI é compatibilidade, não a fachada principal.

Mapeie o que já existe e pode ser reaproveitado, o que conflita, o que falta e quais migrations seriam necessárias. Proponha um roadmap expand/contract em PRs pequenos, começando por contratos de domínio, bundles, MCP Server, autenticação e widget somente leitura.

Não faça reescrita ampla nem migration destrutiva antes de apresentar a auditoria e o plano com evidências, testes e riscos.
```

## 21. Resultado esperado da auditoria

- baseline da branch e commits;
- árvore limpa ou alterações explicadas;
- inventário técnico;
- matriz regra × implementação;
- plano de dados;
- plano de APIs/MCP;
- plano de widget;
- plano de autenticação;
- ordem de PRs;
- critérios de aceite;
- riscos e reversão;
- primeira task implementável.

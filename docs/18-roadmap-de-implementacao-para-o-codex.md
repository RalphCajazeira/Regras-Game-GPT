# Roadmap de Implementação para o Codex

**Versão da proposta:** `implementation-roadmap-v0.2`  
**Status:** pronto para auditoria técnica

## 1. Objetivo

Orientar o Codex a transformar as regras em uma ChatGPT App com widget, MCP e backend autoritativo, reaproveitando o projeto `cronicas-de-outro-mundo` existente.

O Codex não deve assumir que o sistema começa do zero.

## 2. Fontes canônicas

Antes de qualquer alteração:

1. ler este repositório;
2. registrar commit SHA das regras;
3. listar versões implementadas;
4. auditar o repositório do jogo;
5. comparar regra, código, banco, APIs, testes e ambientes;
6. preservar funcionalidades corretas;
7. identificar incompatibilidades com progressão por uso e Cansaço.

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
- modelos Prisma e relações;
- serviços reaproveitáveis;
- endpoints e Actions atuais;
- lacunas contra as regras;
- riscos de migrations;
- plano expand/contract;
- roadmap em PRs pequenos;
- primeiro recorte vertical;
- matriz de testes;
- diagnóstico de atributos, XP, Vigor, tempo e encontros atuais.

Não executar migration destrutiva antes de concluir a auditoria.

## 5. Fase 1 — Fronteira de domínio e contratos

Objetivos:

- separar serviços de aplicação dos controllers;
- criar schemas compartilhados;
- consolidar UUID, `code`, `version`, `clientRef`;
- implementar issues estruturadas;
- implementar `stateVersion` e idempotência;
- implementar readiness e dependency closure;
- implementar bundles transacionais;
- registrar `ruleVersions` em resoluções e progressão.

Critérios:

- ator inválido retorna todos os problemas detectáveis;
- pacote válido cria ou materializa tudo sem chamadas individuais;
- rollback total em erro bloqueante;
- conteúdo existente é reutilizado.

## 6. Fase 2 — Núcleo de progressão

Objetivos:

- criar entidades de progressão por ator;
- criar `ProgressionEvent` auditável;
- implementar XP de nível geral;
- implementar `primaryGrowthCapacity`;
- implementar `attributeTrainingXp`;
- implementar perícias e proficiências;
- implementar domínio individual de habilidade e magia;
- implementar profissão;
- impedir duplicação por idempotência.

Entidades conceituais:

```text
ActorProgression
AttributeTraining
SkillProgression
ProficiencyProgression
ContentMastery
ProfessionProgression
ProgressionEvent
ProgressionMilestone
```

Critérios:

- ação válida pode produzir múltiplas trilhas de XP;
- falha válida pode gerar XP parcial;
- comando rejeitado não gera XP;
- receber dano não concede Esquiva sem tentativa;
- repetição idempotente não duplica XP;
- nível libera capacidade primária;
- atributos consomem capacidade conforme treinamento.

## 7. Fase 3 — Vigor, Cansaço, tempo e sono

Objetivos:

- separar `currentStamina` de `fatigue`;
- calcular Vigor máximo efetivo;
- aplicar custo de Vigor por ação;
- gerar Cansaço por esforço e vigília;
- implementar estágios de Cansaço;
- implementar `awakeMinutes` e `sleepPressure`;
- implementar descanso e sono;
- integrar passagem de tempo;
- criar condições de parada em atividades longas.

Critérios:

- treino avança o tempo;
- Mana não elimina Cansaço de conjuração;
- Cansaço reduz recuperação e desempenho;
- descanso recupera Vigor sem zerar pressão de sono indevidamente;
- sono reduz pressão e Cansaço conforme duração e qualidade;
- atividade repetida encerra por recurso, risco ou condição de parada.

## 8. Fase 4 — Atividades e treinamento em lote

Objetivos:

- resolver repetições sem uma chamada por ação;
- suportar treino livre;
- suportar alvo estático e boneco;
- suportar sparring;
- suportar estudo e meditação;
- aplicar custos e XP por repetição;
- interromper por eventos e limites.

Comandos conceituais:

```text
TRAIN_ACTION_REPETITION
TRAIN_UNTIL_RESOURCE_LIMIT
TRAIN_FOR_DURATION
TRAIN_UNTIL_FATIGUE
SPARRING_SESSION
PHYSICAL_CONDITIONING
STUDY_SESSION
MEDITATION_SESSION
```

Cenário obrigatório:

```text
40 Mana
Bola de Fogo custa 4
→ backend resolve até 10 conjurações quando possível
→ aplica tempo e Cansaço
→ concede XP em todas as trilhas aplicáveis
→ encerra pelo primeiro limite atingido
```

## 9. Fase 5 — MCP Server e Apps SDK

Objetivos:

- criar pacote ou aplicação `mcp-server`;
- publicar ferramentas canônicas;
- implementar schemas tipados;
- mapear ferramentas para serviços;
- publicar recursos de UI;
- implementar `structuredContent`, `content` e `_meta`;
- criar harness de teste.

Primeiras ferramentas:

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
advanceWorldTime
checkpointSession
finalizeSession
cancelSession
```

Treinamento usa ferramentas existentes, não uma ferramenta por modalidade.

## 10. Fase 6 — Autenticação e isolamento

Objetivos:

- OAuth/token;
- vínculo do subject com usuário interno;
- autorização por campanha e ator;
- proteção de dados do Mestre;
- auditoria e tracing;
- ambientes local, staging e produção.

Testes:

- acesso cruzado negado;
- spoofing de ator negado;
- token expirado recuperável;
- logs sem credenciais.

## 11. Fase 7 — Widget somente leitura

Primeira UI:

- ficha;
- recursos;
- atributos;
- nível e XP;
- domínio de habilidades e magias;
- perícias e proficiências;
- Vigor, Cansaço e sono;
- inventário;
- equipamentos;
- missões resumidas;
- sessão ativa;
- recarregamento após remontagem.

O widget ainda não executa escrita nesta fase.

## 12. Fase 8 — Widget de treino

Objetivos:

- selecionar ação ou atividade;
- escolher alvo de treino;
- escolher duração ou repetições;
- mostrar Mana, Vigor, Vida, Cansaço e tempo previstos;
- configurar condições de parada;
- confirmar atividade;
- exibir resumo de progressão;
- solicitar narração somente para marcos.

## 13. Fase 9 — Mapa e passagem do tempo

Objetivos:

- mapa simples com dois ou mais pontos;
- rota e prévia de viagem;
- animação de posição;
- `worldTime` oficial;
- Cansaço e pressão de sono durante viagem;
- evento durante deslocamento;
- diretiva narrativa.

Critérios:

- widget não altera localização sozinho;
- conflito de versão é recuperável;
- idempotência não duplica tempo;
- viagem afeta recursos e Cansaço.

## 14. Fase 10 — Combate visual mínimo

Objetivos:

- mapa tático 2D simples;
- jogador e um inimigo;
- posições em metros;
- clique para mover;
- ataque básico;
- custo de Vigor;
- progressão por evento;
- janela de decisão;
- resolução até `NEXT_PLAYER_DECISION`;
- animação por eventos;
- fim de combate;
- recuperação da sessão.

Não implementar inicialmente todos os tipos de magia ou condição.

## 15. Fase 11 — Loot e inventário interativo

Objetivos:

- corpo ou contêiner clicável;
- itens reais do inimigo;
- `TAKE_ALL` e seleção;
- transferência atômica;
- comparação e equipar;
- peso e slots;
- consumidos não reaparecem.

## 16. Fase 12 — Fluxo criativo pelo GPT

Objetivos:

- GPT transforma linguagem natural em intenção estruturada;
- `resolveCreativeIntent` usa o mesmo domínio;
- treino criativo usa o mesmo motor de atividades;
- materialização de objetos necessários;
- narração de resultados;
- separação entre fatos públicos e do Mestre;
- ações criativas podem interromper o fluxo visual.

Cenários:

```text
lançar magia ao alto até acabar Mana
socar a si mesmo até limite declarado
treinar com árvore
sparring com companheiro
correr até Cansaço definido
meditar antes de dormir
```

## 17. Fase 13 — Narração seletiva

Objetivos:

- `narrationDirective`;
- níveis `NONE` a `PLAYER_DECISION_REQUIRED`;
- resumos desde a última narração;
- mensagem de acompanhamento;
- não poluir contexto com cada clique;
- narrar nível, atributo, domínio e colapso quando relevante.

## 18. Fases posteriores

### Comércio e fabricação

- estoque e saldo;
- prévias;
- confirmação oficial;
- receitas e materiais;
- progressão profissional;
- qualidade e instâncias.

### Relações, companheiros e missões

- painel de relações;
- memórias e impressões;
- recrutamento e domesticação;
- missões e objetivos;
- progressão de companheiros;
- escolhas narrativas.

### Administração e frontend externo

- revisar definições e instâncias;
- histórico e versões;
- migrações explícitas;
- dashboard de sessões e progressão;
- reutilizar serviços do App;
- avaliar GraphQL para consultas flexíveis.

## 19. Estratégia de branches e PRs

- trabalhar sobre `develop` do jogo;
- uma fase ou subfase por branch curta;
- PRs pequenos e testáveis;
- migrations expand/contract;
- não remover compatibilidade antes do novo fluxo estar validado;
- registrar regra e commit implementado em cada PR.

## 20. Testes obrigatórios

### Unidade

- fórmulas;
- progressão;
- atributos;
- Cansaço;
- sono;
- validadores;
- autorização;
- idempotência;
- classificação narrativa.

### Integração

- MCP → serviço → banco;
- bundle atômico;
- treino em lote;
- mapa e viagem;
- combate e XP;
- loot;
- descanso e sono;
- recuperação de sessão;
- widget chamando ferramenta.

### E2E

```text
login
→ carregar campanha
→ visualizar progressão
→ lançar magia em treino até acabar Mana
→ receber XP e Cansaço
→ descansar
→ abrir mapa
→ viajar
→ iniciar combate
→ mover e atacar
→ receber XP de ação e defesa aplicável
→ derrotar
→ coletar loot
→ receber narração
→ reabrir sessão
```

## 21. Critérios para não avançar

Não avançar quando:

- testes críticos falharem;
- migrations não tiverem recuperação;
- widget e backend divergirem em versão;
- dados do Mestre vazarem;
- domínio estiver duplicado no widget;
- operação não for idempotente;
- sessão não puder ser retomada;
- treino duplicar XP;
- passagem de tempo não afetar Cansaço;
- ação rejeitada conceder XP;
- dano passivo conceder Esquiva sem tentativa.

## 22. Primeiro prompt recomendado ao Codex

```text
Audite o repositório RalphCajazeira/cronicas-de-outro-mundo na branch develop contra o commit atual de RalphCajazeira/Regras-Game-GPT.

Considere como arquitetura canônica ChatGPT App/Plugin + Apps SDK + MCP Tools + Widget JS + serviços de domínio + backend/PostgreSQL.

Considere também como regras canônicas a progressão por uso, capacidade de crescimento primário, domínio individual de habilidades e magias, treino em lote, Vigor, Cansaço, passagem de tempo, pressão de sono e recuperação.

Mapeie o que já existe e pode ser reaproveitado, o que conflita, o que falta e quais migrations seriam necessárias. Proponha um roadmap expand/contract em PRs pequenos.

Não faça reescrita ampla nem migration destrutiva antes de apresentar auditoria, plano, evidências, testes e riscos.
```

## 23. Resultado esperado da auditoria

- baseline e commits;
- árvore limpa ou alterações explicadas;
- inventário técnico;
- matriz regra × implementação;
- plano de dados;
- plano MCP;
- plano de widget;
- plano de autenticação;
- plano de progressão;
- plano de Cansaço e tempo;
- ordem de PRs;
- critérios de aceite;
- riscos e reversão;
- primeira task implementável.

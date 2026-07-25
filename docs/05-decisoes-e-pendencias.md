# Decisões e Pendências

Este arquivo separa decisões canônicas de pontos ainda sujeitos a implementação e simulação.

## 1. Arquitetura canônica

```text
GPT → intenção, criatividade, diálogo e narração
Widget JS → interface, prévias e interações rápidas
MCP Server → ferramentas tipadas e recursos de UI
Backend → regras oficiais, resolução e persistência
Banco → estado, histórico, definições e instâncias
```

Decisões:

- a experiência principal será um ChatGPT App/Plugin com Apps SDK, MCP e widgets;
- Custom GPT Actions/OpenAPI passa a ser legado ou fallback;
- REST pode permanecer como transporte interno ou cliente externo;
- MCP, REST, widget e administração reutilizam os mesmos serviços;
- regras não ficam duplicadas no widget ou prompt;
- estado oficial permanece no backend;
- identidade deriva de OAuth ou token;
- dados do Mestre não podem vazar ao widget.

## 2. Fluxos

### Ação comum

```text
widget → prévia → confirmação → backend resolve → widget atualiza
```

### Ação criativa

```text
jogador escreve → GPT interpreta → backend resolve → GPT narra
```

### Evento importante

```text
backend retorna narrationDirective → widget atualiza → GPT narra ou pede decisão
```

### Treinamento

```text
jogador declara treino
→ GPT ou widget estrutura atividade
→ backend resolve tempo, custos e repetições
→ concede progressão oficial
→ widget atualiza
```

## 3. Ferramentas MCP

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

Decisões:

- manter catálogo focado;
- não criar ferramenta por microação;
- não criar ferramenta genérica arbitrária;
- separar leitura, prévia e escrita;
- widget pode chamar ferramentas autorizadas diretamente;
- o antigo limite de 30 Actions não é regra canônica da arquitetura MCP;
- treino em lote reutiliza `resolveGameCommand`, `resolveCreativeIntent` e `advanceWorldTime`.

## 4. Estado e segurança

- UUID é identidade técnica;
- definições possuem `code` e `version`;
- `clientRef` liga nós em bundles;
- toda mutação usa idempotência e `stateVersion`;
- autorização é validada por recurso;
- `playerId` enviado não comprova identidade;
- segredos não entram em dados renderizados;
- logs e tracing são obrigatórios.

## 5. Bundles

- GPT pode montar pacote completo em memória;
- backend valida todos os componentes aplicáveis;
- todos os erros detectáveis são retornados juntos;
- conteúdo existente é reutilizado;
- conteúdo faltante pode ser criado quando autorizado;
- persistência completa usa `ALL_OR_NOTHING`;
- materialização temporária pode ser promovida;
- widget pode renderizar pacote temporário sem torná-lo canônico.

## 6. Mestre e resolução

- backend é autoridade oficial para ações comuns no widget;
- backend pode resolver vários eventos até `NEXT_PLAYER_DECISION`;
- GPT controla intenção aberta, NPCs importantes e narrativa;
- GPT resolve mecânica localmente somente em fallback explicitamente autorizado;
- checkpoints ocorrem por fase ou consequência importante, não por microação;
- replay parcial preserva eventos válidos;
- resumo mecânico e narrativo suportam retomada.

## 7. Widget

Modos:

```text
CHARACTER_SHEET
EXPLORATION_MAP
TACTICAL_BATTLE_MAP
INVENTORY
LOOT
TRADE
CRAFTING
MISSION_LOG
RELATIONSHIP_VIEW
TRAINING
PROGRESSION
REST
```

Decisões:

- mapa inicial pode ser grafo de pontos e rotas;
- viagem avança `worldTime` oficialmente;
- combate usa mapa 2D simples e metros;
- interface apresenta janelas de decisão;
- motor continua baseado em ticks;
- jogador pode planejar mover, atacar e recuar;
- backend pode interromper o plano;
- loot é clicável e reflete estado real;
- GPT não narra cada ataque comum;
- widget deve ser remontável a partir do backend;
- widget mostra prévias de treino, custo, duração, XP estimada e Cansaço;
- widget nunca concede XP ou recupera recursos por conta própria.

## 8. Atributos

```text
Força
Agilidade
Destreza
Vitalidade
Inteligência
```

- Agilidade representa rapidez e mobilidade;
- Destreza representa precisão e controle;
- cada nível concede `10` pontos de capacidade de crescimento primário;
- uso e treinamento determinam quais atributos consomem essa capacidade;
- atributos acumulam XP de treinamento próprio;
- pontos não utilizados e XP pendente são persistidos;
- o custo de aumento depende do valor permanente atual;
- especializações usam perícias, proficiências, profissões e passivas.

## 9. Progressão por uso

Decisões canônicas:

- toda ação mecanicamente executada pode gerar aprendizagem;
- treino não exige combate;
- prática livre, alvo estático, boneco, sparring, estudo e meditação são válidos;
- repetição válida não recebe `0 XP` somente por ser repetição;
- contexto, dificuldade, resultado, Cansaço e qualidade alteram a quantidade;
- ação que falhou pode conceder XP de tentativa e execução;
- comando rejeitado antes da execução não concede XP;
- ações interrompidas podem conceder XP parcial;
- progressão oficial nasce de eventos mecânicos confirmados;
- backend calcula e persiste tudo de forma idempotente;
- XP é armazenada em milésimos para preservar frações.

Camadas:

```text
nível geral
atributos
perícias
proficiências
domínio de habilidades e magias
profissões
passivas e especializações
```

### Ataque e defesa

- tentativa de ataque pode treinar ação, arma, perícia e atributos;
- acerto e dano acrescentam XP de resultado;
- receber dano treina resiliência, Vitalidade ou tolerância aplicável;
- receber dano não treina Esquiva ou Bloqueio automaticamente;
- Esquiva e Bloqueio exigem tentativa real correspondente.

### Treino contra si mesmo

- pode ocorrer quando a ação for física e mecanicamente válida;
- Vida, Vigor, Cansaço, tempo, ferimentos e morte continuam reais;
- backend respeita condições de parada declaradas;
- não existe proteção narrativa automática contra a consequência escolhida.

## 10. Baseline numérico de progressão

Documento canônico:

```text
docs/21-curvas-numericas-de-xp-e-balanceamento-inicial.md
```

### Precisão

```text
1 XP = 1.000 xpMilli
```

### Fórmula por evento

```text
ExecutionLearning =
executionBaseXp
× contexto
× dificuldade
× execução
× repetição
× Cansaço
× modificadores

ResultLearning =
resultBaseXp
× contexto
× dificuldade
× qualidade do resultado
× repetição
× Cansaço
× modificadores
```

### Contextos iniciais

```text
FREE_PRACTICE       0,50
STATIC_TARGET       0,65
TRAINING_DUMMY      0,80
CONTROLLED_EXERCISE 0,75
STRUCTURED_TRAINING 1,00
SPARRING            1,10
REAL_CHALLENGE      1,25
EXTREME_CHALLENGE   1,40
```

### Repetição semanticamente idêntica

```text
1–10   → 1,00
11–25  → 0,90
26–50  → 0,80
51–100 → 0,70
101–200 → 0,60
201+   → 0,50
```

O piso de repetição não elimina XP; custos e tempo permanecem integrais.

### Curvas

```text
ActorXpToNext(L) = 100 + 50 × (L - 1) + 10 × (L - 1)²
MasteryXpToNext(L) = 20 + 4 × L + floor(L² ÷ 10)
SkillXpToNext(L) = 30 + 6 × L + floor(L² ÷ 8)
ProficiencyXpToNext(L) = 25 + 5 × L + floor(L² ÷ 9)
ProfessionXpToNext(L) = 50 + 8 × L + floor(L² ÷ 6)
AttributeTrainingXpToIncrease(A) = 50 + 5 × A + 2 × A²
```

### Exemplo canônico

```text
10 Bolas de Fogo ao alto
→ 30 XP de domínio
→ 12 XP de Piromancia
→ 7,50 XP de atributos
→ 3 XP geral por uso
```

Esses números são baseline implementável e ainda precisam de simulação, não rediscussão conceitual antes da primeira bateria de testes.

## 11. Nível geral

- nível não depende somente de matar inimigos;
- uso, objetivos, treino, missões, descobertas, produção e marcos geram XP geral;
- nível libera capacidade de crescimento primário;
- domínio de técnicas e perícias possui progressão própria;
- NPC materializado deve respeitar orçamento e capacidade compatíveis com seu nível;
- nível geral não possui limite rígido nesta versão;
- domínio, perícia, proficiência e profissão usam níveis de `1` a `100`.

## 12. Habilidades e magias

Cada ator possui domínio individual da definição conhecida.

Uma definição pode declarar escalonamento por domínio em:

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
chance de condição
número de alvos
estabilidade
```

Nem toda habilidade melhora todos os campos.

Escalonamento gradual usa:

```text
normalizedMastery = (masteryLevel - 1) ÷ 99
bonus = maxBonusAt100 × normalizedMastery ^ curveExponent
```

Crescimento também pode combinar marcos, especializações e novas opções.

## 13. Vigor, Cansaço e sono

```text
Vigor   → esforço imediato
Cansaço → desgaste persistente de 0 a 100
Sono    → pressão de vigília e recuperação prolongada
```

- Mana não elimina Cansaço de conjuração repetida;
- ataques, corrida, natação, esquiva, bloqueio e magia podem gerar Cansaço;
- Vigor recupera mais rápido que Cansaço;
- Cansaço reduz máximo efetivo, regeneração e desempenho;
- uma ação executada pode gerar XP mesmo cansada;
- Cansaço crítico pode impedir a próxima execução por falha ou colapso;
- descanso recupera Vigor, mas não substitui sono indefinidamente;
- sono avança o tempo e recupera conforme duração e qualidade;
- comida e água serão integradas depois.

Multiplicadores iniciais de aprendizagem:

```text
RESTED        1,00
TIRED         0,95
FATIGUED      0,85
EXHAUSTED     0,70
CRITICAL      0,50
COLLAPSE_RISK 0,25
```

Faixas provisórias de vigília:

```text
0–16h  normal
16–20h pressão leve
20–24h pressão relevante
24–36h severa
36–48h crítica
48h+   colapso altamente provável
```

Essas faixas são regra de jogo, não orientação médica.

## 14. Atores

- modelo universal para pessoas, animais, monstros, criaturas, espíritos, construtos, invocações e jogadores;
- natureza, espécie, arquétipo, papel, facção e controle são separados;
- materialização ocorre quando a entidade se torna relevante;
- ficha materializada define recursos, ações, inventário e loot;
- companheiro é vínculo, não espécie;
- atores anônimos podem ser temporários e promovidos;
- atores persistentes podem acumular progressão;
- inimigos descartáveis não precisam persistir ganhos após a sessão.

## 15. Itens, qualidade e inventário

```text
Definição Comum de referência
→ variante apenas para diferença real
→ qualidade na instância
→ valores resolvidos congelados
```

- qualidade não cria cadastro duplicado;
- toda instância possui localização e propriedade;
- item não pode estar em dois lugares ou operações;
- consumo afeta loot;
- drops naturais e itens carregados são separados;
- reservas impedem uso concorrente.

## 16. Combate

- posições e alcance em metros;
- linha do tempo contínua em ticks;
- preparação e recuperação;
- uma rolagem de acerto;
- crítico, Defesa Crítica e mitigação;
- ações podem ser interrompidas;
- backend resolve oficial e retorna eventos;
- widget anima até a próxima decisão;
- ações criativas continuam pelo GPT;
- cada evento pode produzir progressão;
- inimigos fortes aumentam XP por dificuldade sem remover o risco normal.

## 17. Economia e fabricação

- moeda `CROWN`;
- definição guarda preço Comum;
- instância guarda preço resolvido;
- backend controla saldo, estoque e transação;
- fabricação referencia definição existente;
- sucesso cria instância, não definição por qualidade;
- qualidade, consumo e XP são autoritativos;
- prática profissional pode evoluir perícia, profissão, receita e atributos aplicáveis.

## 18. Pendências bloqueadoras próximas

1. Sistema de Condições e Efeitos.
2. Encontros, objetivos e transições completas.
3. Relações, companheiros, romance e domesticação.
4. Missões e recompensas.
5. Mundo, locais, rotas e exploração.
6. Morte, incapacidade e consequências.
7. Facções, reputação e crimes.
8. Espécies, origens e arquétipos.
9. Comportamentos de NPCs.
10. Comida, água e sobrevivência detalhada.

## 19. Pendências de calibração

As curvas iniciais estão definidas. Ainda faltam simulações para:

- velocidade real de progressão `1 → 20`, `1 → 50` e `1 → 100`;
- XP de cura, suporte, controle e atividades sociais;
- custo de Vigor por categoria de ação;
- regeneração de Vigor;
- geração de Cansaço por ação;
- recuperação por descanso e sono;
- treino com instrutor;
- progressão de companheiros;
- fabricação por etapa e qualidade;
- migração quando uma curva for revisada;
- comportamento após morte, rollback ou recarga.

## 20. Pendências técnicas

- auditar backend atual;
- definir migrations expand/contract;
- implementar MCP Server e recursos de UI;
- implementar OAuth e autorização;
- definir política final de permissões do App;
- implementar `structuredContent`, `content` e `_meta`;
- implementar diretiva narrativa;
- definir limites de payload e paginação;
- implementar tracing e métricas;
- testar remontagem de widget;
- testar seleção de ferramenta pelo GPT;
- testar chamadas diretas do widget;
- implementar resolução de atividades longas em lote;
- validar disponibilidade da plataforma por plano ou workspace.

## 21. Primeiro recorte vertical

```text
login
→ carregar campanha
→ visualizar progressão
→ executar treino de Bola de Fogo
→ aplicar curvas e Cansaço
→ abrir mapa
→ viajar
→ iniciar combate
→ mover
→ atacar
→ gerar progressão por ação
→ resolver até próxima decisão
→ derrotar inimigo
→ abrir loot
→ transferir itens
→ narrar desfecho
→ descansar
→ retomar após recarregar
```

## 22. Roadmap do Codex

A implementação deve seguir:

`docs/18-roadmap-de-implementacao-para-o-codex.md`

O primeiro trabalho é auditoria e plano, não reescrita ampla.

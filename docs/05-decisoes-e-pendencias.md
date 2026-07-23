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
- Custom GPT Actions/OpenAPI passa a ser legado/fallback;
- REST pode permanecer como transporte interno ou cliente externo;
- MCP, REST, widget e administração reutilizam os mesmos serviços;
- regras não ficam duplicadas no widget ou prompt;
- estado oficial permanece no backend;
- identidade deriva de OAuth/token;
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

- manter catálogo focado;
- não criar ferramenta por microação;
- não criar ferramenta genérica arbitrária;
- ferramentas de leitura, prévia e escrita são separadas;
- widget pode chamar ferramentas autorizadas diretamente;
- o antigo limite de 30 Actions não é regra canônica da arquitetura MCP.

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
- GPT localmente resolve mecânica somente em fallback explicitamente autorizado;
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
- widget deve ser remontável a partir do backend.

## 8. Atributos

```text
Força
Agilidade
Destreza
Vitalidade
Inteligência
```

- Agilidade = rapidez e mobilidade;
- Destreza = precisão e controle;
- cada nível concede 10 pontos primários, ainda sujeito a calibração;
- especializações usam perícias, proficiências, profissões e passivas.

## 9. Atores

- modelo universal para pessoas, animais, monstros, criaturas, espíritos, construtos, invocações e jogadores;
- natureza, espécie, arquétipo, papel, facção e controle são separados;
- materialização ocorre quando a entidade se torna relevante;
- ficha materializada define recursos, ações, inventário e loot;
- companheiro é vínculo, não espécie;
- atores anônimos podem ser temporários e promovidos.

## 10. Itens, qualidade e inventário

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

## 11. Combate

- posições e alcance em metros;
- linha do tempo contínua em ticks;
- preparação e recuperação;
- uma rolagem de acerto;
- crítico, Defesa Crítica e mitigação;
- ações podem ser interrompidas;
- backend resolve oficial e retorna eventos;
- widget anima até a próxima decisão;
- ações criativas continuam pelo GPT.

## 12. Economia e fabricação

- moeda `CROWN`;
- definição guarda preço Comum;
- instância guarda preço resolvido;
- backend controla saldo, estoque e transação;
- fabricação referencia definição existente;
- sucesso cria instância, não definição por qualidade;
- qualidade, consumo e XP são autoritativos.

## 13. Pendências bloqueadoras próximas

1. Sistema de Condições e Efeitos.
2. Progressão, XP e níveis.
3. Encontros, objetivos e transições completas.
4. Relações, companheiros, romance e domesticação.
5. Missões e recompensas.
6. Tempo, descanso e recuperação detalhados.
7. Mundo, locais e exploração.
8. Morte, incapacidade e consequências.
9. Facções, reputação e crimes.

## 14. Pendências técnicas

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
- validar disponibilidade da plataforma por plano/workspace.

## 15. Primeiro recorte vertical

```text
login
→ carregar campanha
→ abrir mapa
→ viajar
→ iniciar combate
→ mover
→ atacar
→ resolver até próxima decisão
→ derrotar inimigo
→ abrir loot
→ transferir itens
→ narrar desfecho
→ retomar após recarregar
```

## 16. Roadmap do Codex

A implementação deve seguir:

`docs/18-roadmap-de-implementacao-para-o-codex.md`

O primeiro trabalho é auditoria e plano, não reescrita ampla.

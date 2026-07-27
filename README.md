# Regras Game GPT

Base canônica de produto, domínio e arquitetura para o RPG narrativo **Crônicas de Outro Mundo**.

Este repositório define **o que o jogo deve ser**, quais regras e decisões são oficiais e qual arquitetura-alvo deve orientar a evolução incremental do repositório executável `RalphCajazeira/cronicas-de-outro-mundo`.

Ele não é a fonte isolada para afirmar o que já está implantado. O estado real de implementação deve ser confirmado no código, banco, ambientes, testes e documentos vivos do repositório do jogo.

## Fontes de verdade

Quando houver conflito, priorizar:

1. ambiente, banco ou serviço atual;
2. repositório, branch, arquivos e Git atuais;
3. lint, typecheck, testes, build e CI atuais;
4. contratos e documentação versionada no repositório do jogo;
5. este repositório de regras canônicas;
6. relatórios anteriores;
7. memória e hipóteses.

Nunca registrar hipótese como funcionalidade implantada.

## Arquitetura canônica atual

```text
GPT personalizado
→ Instructions + Knowledge + App MCP
→ intenção, criatividade, diálogo, interpretação e narração

Extensão Chromium Manifest V3
→ frontend principal do jogador
→ botão flutuante, overlay/full-page e página própria
→ ficha, inventário, equipamentos, habilidades, mapa, combate e demais controles

App MCP
→ ponte tipada entre o GPT e os serviços do jogo
→ tools oficiais, autenticação e contratos públicos

Backend
→ autorização, validação, rolagens, cálculos, resolução, progressão e persistência

PostgreSQL
→ estado durável, histórico, definições, instâncias, eventos e sessões

Widget do ChatGPT
→ fallback leve, bootstrap, diagnóstico e compatibilidade
→ não é mais o frontend principal

Actions/OpenAPI no GPT
→ legado temporário durante a migração
→ não farão parte do GPT personalizado definitivo
```

A decisão detalhada está em [`docs/22-arquitetura-gpt-extensao-backend.md`](docs/22-arquitetura-gpt-extensao-backend.md).

## Responsabilidades

### GPT personalizado

- manter identidade, tom e comportamento do Mestre;
- interpretar ações livres e intenções criativas;
- conduzir diálogos e narrativa;
- usar as tools MCP corretas;
- narrar somente resultados oficiais quando houver mecânica;
- nunca inventar recursos, custos, rolagens ou efeitos não confirmados pelo backend.

### Extensão

- ser o frontend principal e persistente;
- apresentar estado oficial por projeções autorizadas;
- oferecer navegação, prévias, confirmações e ações mecânicas;
- atualizar automaticamente quando o backend mudar;
- manter apenas estado visual efêmero;
- não ler ou copiar a conversa do ChatGPT;
- não capturar cookies, tokens ou APIs internas não documentadas do ChatGPT.

### Backend

- ser a autoridade mecânica e de autorização;
- derivar identidade do token;
- aplicar `stateVersion`, idempotência e transações;
- proteger dados `MASTER_ONLY`;
- publicar projeções públicas allowlisted;
- persistir eventos e consequências oficiais;
- notificar a extensão sobre invalidações, sem transformar o canal realtime em fonte de verdade.

## Fluxos

### Ação mecânica comum

```text
jogador usa a extensão
→ extensão solicita prévia quando aplicável
→ jogador confirma
→ backend autoriza, resolve e persiste
→ backend publica evento de invalidação
→ extensão recarrega as projeções afetadas
→ GPT narra somente quando necessário
```

### Ação criativa

```text
jogador escreve ao GPT
→ GPT interpreta a intenção
→ App MCP envia intenção estruturada
→ backend valida e resolve
→ extensão recebe a atualização oficial
→ GPT narra o resultado confirmado
```

### Criação em lote

```text
GPT monta pacote completo
→ backend valida todos os nós e vínculos
→ reutiliza conteúdo existente
→ cria somente o necessário
→ materializa temporariamente ou persiste atomicamente
```

### Treinamento e progressão

```text
jogador declara treino ou executa ação
→ backend aplica tempo, custos e consequências
→ gera eventos de aprendizagem
→ evolui domínio, perícias, atributos e nível
→ extensão atualiza barras e estado
→ GPT narra somente marcos importantes
```

Toda ação mecanicamente executada pode ensinar, inclusive prática livre, alvo estático, boneco, sparring ou combate real. O contexto altera a XP, mas repetição válida não recebe zero apenas por ser repetição.

## Atualização automática da extensão

O canal em tempo real será usado como notificação, não como estado oficial.

```text
transação confirmada no PostgreSQL
→ backend publica evento pequeno
→ extensão invalida as consultas afetadas
→ extensão busca novamente o estado oficial
```

A implementação pode começar com recarga após mutações e polling controlado, evoluindo para SSE ou WebSocket autenticado conforme a frequência e a necessidade de combate/mapa.

## Mapa e combate

```text
mapa de exploração
→ viagem e passagem do tempo
→ encontro
→ mapa tático
→ janela de decisão
→ movimento e ações por ticks
→ resolução até próxima decisão
→ fim do combate
→ loot clicável
→ retorno à exploração
```

O motor utiliza tempo contínuo em ticks; a extensão apresenta decisões semelhantes a turnos.

## Progressão por uso

```text
uso válido
→ custos reais
→ XP de domínio
→ XP de perícia ou proficiência
→ treinamento de atributos
→ experiência geral de desenvolvimento
```

O nível geral libera capacidade de crescimento. O uso decide quais atributos, habilidades, magias, perícias e profissões realmente evoluem.

Treinamento não exige combate. Exemplos válidos:

- lançar magia até consumir a Mana;
- praticar contra árvore, alvo ou boneco;
- sparring;
- corrida, natação ou condicionamento;
- estudo, meditação ou fabricação;
- treino criativo interpretado pelo GPT.

### Baseline numérico

```text
XP armazenada em milésimos
→ contexto, dificuldade, execução, resultado, repetição e Cansaço
→ distribuição entre trilhas
→ curvas próprias para ator, domínio, perícia, proficiência, profissão e atributo
```

As fórmulas iniciais estão em [`docs/21-curvas-numericas-de-xp-e-balanceamento-inicial.md`](docs/21-curvas-numericas-de-xp-e-balanceamento-inicial.md).

Exemplo canônico:

```text
40 Mana
Bola de Fogo custa 4
→ 10 conjurações em prática livre
→ 30 XP de domínio
→ 12 XP de Piromancia
→ 7,50 XP distribuída entre atributos
→ 3 XP geral no total
```

Custos, tempo, Cansaço e eventos ambientais continuam integrais.

## Vigor, Cansaço e sono

```text
Vigor   → esforço curto
Cansaço → desgaste acumulado
Sono    → recuperação prolongada e pressão de vigília
```

Tempo, Mana, Vigor, Vida, Cansaço, ferimentos e risco são limites naturais do treinamento. O backend não impede uma escolha perigosa; aplica suas consequências.

## Regras canônicas importantes

### Atributos

```text
Força
Agilidade
Destreza
Vitalidade
Inteligência
```

### Itens

```text
Definição única com perfil Comum
→ variante apenas para diferença real
→ qualidade na instância
→ valores resolvidos e congelados
```

### Identidade

```text
UUID      → identidade técnica
code      → definição legível
version   → versão da definição
clientRef → referência local em bundle
```

### Estado

```text
backend  = autoridade oficial
extensão = estado visual efêmero e frontend principal
widget   = fallback
GPT      = interpretação e narrativa
```

Toda mutação crítica usa idempotência, `stateVersion`, versões de regras e autorização derivada do token.

## Governança de estado e roadmap

A separação obrigatória é:

```text
Regras-Game-GPT
→ produto, domínio, arquitetura-alvo e decisões canônicas

cronicas-de-outro-mundo
→ estado implementado, evidências, rollout, roadmap executável e backlog
```

O processo está definido em [`docs/23-governanca-estado-roadmap-e-mudancas.md`](docs/23-governanca-estado-roadmap-e-mudancas.md).

No repositório executável, os documentos vivos são:

- `docs/ai/project/PROJECT_STATE.md`;
- `docs/ai/project/ROADMAP.md`;
- `docs/ai/project/DECISIONS.md`.

## Documentos

### Sistemas de domínio

- [`docs/01-sistema-de-atributos.md`](docs/01-sistema-de-atributos.md)
- [`docs/02-sistema-de-combate.md`](docs/02-sistema-de-combate.md)
- [`docs/03-sistema-de-equipamentos.md`](docs/03-sistema-de-equipamentos.md)
- [`docs/06-sistema-de-economia-e-comercio.md`](docs/06-sistema-de-economia-e-comercio.md)
- [`docs/07-sistema-de-acoes-habilidades-e-magias.md`](docs/07-sistema-de-acoes-habilidades-e-magias.md)
- [`docs/08-sistema-de-pericias-profissoes-e-passivas.md`](docs/08-sistema-de-pericias-profissoes-e-passivas.md)
- [`docs/09-sistema-de-fabricacao-e-qualidade.md`](docs/09-sistema-de-fabricacao-e-qualidade.md)
- [`docs/10-sistema-de-atores-definicoes-e-instancias.md`](docs/10-sistema-de-atores-definicoes-e-instancias.md)
- [`docs/11-sistema-de-inventario-itens-drops-e-saque.md`](docs/11-sistema-de-inventario-itens-drops-e-saque.md)
- [`docs/19-progressao-niveis-experiencia-treinamento-e-dominio.md`](docs/19-progressao-niveis-experiencia-treinamento-e-dominio.md)
- [`docs/20-vigor-cansaco-sono-e-recuperacao.md`](docs/20-vigor-cansaco-sono-e-recuperacao.md)
- [`docs/21-curvas-numericas-de-xp-e-balanceamento-inicial.md`](docs/21-curvas-numericas-de-xp-e-balanceamento-inicial.md)

### Arquitetura, integração e governança

- [`docs/04-integracao-gpt-backend.md`](docs/04-integracao-gpt-backend.md)
- [`docs/05-decisoes-e-pendencias.md`](docs/05-decisoes-e-pendencias.md): decisões anteriores; prevalece o documento 22 quando houver conflito de superfície visual.
- [`docs/12-contratos-operacionais-mcp-erros-e-recuperacao.md`](docs/12-contratos-operacionais-mcp-erros-e-recuperacao.md)
- [`docs/13-criacao-validacao-e-materializacao-em-lote.md`](docs/13-criacao-validacao-e-materializacao-em-lote.md)
- [`docs/14-sessoes-do-mestre-resolucao-local-checkpoints-e-replay.md`](docs/14-sessoes-do-mestre-resolucao-local-checkpoints-e-replay.md)
- [`docs/15-arquitetura-chatgpt-app-widget-mcp-e-backend.md`](docs/15-arquitetura-chatgpt-app-widget-mcp-e-backend.md): referência histórica da fase widget-first.
- [`docs/16-widget-mapas-combate-tempo-e-loot.md`](docs/16-widget-mapas-combate-tempo-e-loot.md): requisitos visuais reaproveitáveis pela extensão.
- [`docs/17-autenticacao-identidade-seguranca-e-observabilidade.md`](docs/17-autenticacao-identidade-seguranca-e-observabilidade.md)
- [`docs/18-roadmap-de-implementacao-para-o-codex.md`](docs/18-roadmap-de-implementacao-para-o-codex.md): baseline histórico de migração.
- [`docs/22-arquitetura-gpt-extensao-backend.md`](docs/22-arquitetura-gpt-extensao-backend.md): arquitetura canônica atual.
- [`docs/23-governanca-estado-roadmap-e-mudancas.md`](docs/23-governanca-estado-roadmap-e-mudancas.md): manutenção de estado, decisões e backlog.
- [`docs/24-roadmap-extensao-e-jogabilidade.md`](docs/24-roadmap-extensao-e-jogabilidade.md): ordem atual de entrega da experiência jogável.

### Legado

- [`docs/legacy/01-arquitetura-custom-gpt-actions.md`](docs/legacy/01-arquitetura-custom-gpt-actions.md)

## Versões atuais

```text
attributes-v0.5
combat-v0.2
equipment-v0.5
actions-v0.1
economy-v0.3
skills-v0.2
crafting-v0.2
actors-v0.1
inventory-v0.2
mcp-contracts-v0.1
bundles-v0.2
master-runtime-v0.2
extension-architecture-v0.1
state-governance-v0.1
implementation-roadmap-v0.4
progression-v0.1
progression-curves-v0.1
fatigue-v0.1
integration-v1.1
```

## Próximos sistemas de regra

1. Condições e Efeitos.
2. Encontros, Objetivos e Transições.
3. Relações, Vínculos e Companheiros.
4. Missões e Recompensas.
5. Mundo, Locais, Rotas e Exploração.
6. Morte, Incapacidade e Consequências.
7. Facções, Reputação, Crimes e Testemunhas.
8. Espécies, Origens e Arquétipos.
9. Comportamentos de NPCs.
10. Criação Inicial e Início de Campanha.
11. Comida, Água e Sobrevivência detalhada.

## Diretriz para o Codex

O Codex deve começar pelo estado real do repositório `RalphCajazeira/cronicas-de-outro-mundo` na branch `develop`, consultar os documentos vivos do projeto e comparar com estas regras.

Não fazer reescrita ampla nem migration destrutiva antes de auditoria, matriz de reaproveitamento, plano expand/contract e critérios de aceite.

Aplicar:

```text
Pesquisar → Reutilizar → Adaptar → Extrair pequeno → Criar novo
```

## Convenções

- nomes exibidos em português;
- códigos e campos em inglês;
- UUID como identidade principal;
- distâncias em metros;
- combate em ticks;
- moeda `CROWN`;
- XP interna em `xpMilli`, com `1 XP = 1.000 xpMilli`;
- backend como autoridade oficial;
- extensão sem regra autoritativa duplicada;
- widget mantido apenas como fallback;
- GPT não inventa recursos mecânicos ausentes;
- informações do Mestre não aparecem em superfícies do jogador;
- ações executadas podem gerar aprendizagem mesmo fora de combate;
- custos, tempo, Cansaço e consequências nunca são ignorados em treino;
- toda mudança de regra atualiza a versão correspondente;
- toda mudança de capacidade ou rollout atualiza os documentos vivos do repositório executável.
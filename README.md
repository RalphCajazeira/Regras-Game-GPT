# Regras Game GPT

Base de conhecimento para um RPG conduzido pelo GPT como Mestre narrativo, com widget interativo dentro do ChatGPT, ferramentas MCP, backend autoritativo e banco persistente.

## Arquitetura canônica

```text
GPT
→ compreensão da intenção, criatividade, diálogo e narração

Widget JS
→ ficha, mapas, combate, inventário, loot, comércio e prévias

MCP Server / Apps SDK
→ ponte tipada entre ChatGPT e serviços do jogo

Backend
→ validação, rolagens e cálculos oficiais, resolução e persistência

PostgreSQL
→ estado durável, histórico, definições, instâncias e sessões
```

A experiência principal será um ChatGPT App/Plugin com Apps SDK, MCP e widgets. A arquitetura anterior de Custom GPT com Actions/OpenAPI permanece apenas como legado, fallback textual ou compatibilidade de migração.

## Fluxos

### Ação comum

```text
jogador clica
→ widget mostra prévia
→ backend resolve oficialmente
→ widget atualiza
→ GPT narra apenas quando necessário
```

### Ação criativa

```text
jogador escreve
→ GPT interpreta
→ backend valida e resolve
→ GPT narra
```

### Criação em lote

```text
GPT monta pacote completo
→ backend valida todos os nós e vínculos
→ reutiliza conteúdo existente
→ cria somente o necessário
→ materializa temporariamente ou persiste atomicamente
```

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

O motor utiliza tempo contínuo em ticks; o widget apresenta decisões semelhantes a turnos.

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
backend = autoridade
widget = estado visual efêmero
GPT = interpretação e narrativa
```

Toda mutação crítica usa idempotência, `stateVersion`, versões de regras e autorização derivada do token.

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

### Arquitetura e integração

- [`docs/04-integracao-gpt-backend.md`](docs/04-integracao-gpt-backend.md): integração canônica App/Widget/MCP/Backend.
- [`docs/05-decisoes-e-pendencias.md`](docs/05-decisoes-e-pendencias.md): decisões consolidadas e lacunas.
- [`docs/12-contratos-operacionais-mcp-erros-e-recuperacao.md`](docs/12-contratos-operacionais-mcp-erros-e-recuperacao.md): ferramentas MCP, contratos, erros e recuperação.
- [`docs/13-criacao-validacao-e-materializacao-em-lote.md`](docs/13-criacao-validacao-e-materializacao-em-lote.md): bundles, UUID, `clientRef`, validação e materialização.
- [`docs/14-sessoes-do-mestre-resolucao-local-checkpoints-e-replay.md`](docs/14-sessoes-do-mestre-resolucao-local-checkpoints-e-replay.md): orquestração do Mestre, sessões, checkpoints e replay.
- [`docs/15-arquitetura-chatgpt-app-widget-mcp-e-backend.md`](docs/15-arquitetura-chatgpt-app-widget-mcp-e-backend.md): responsabilidades de cada componente e fluxos principais.
- [`docs/16-widget-mapas-combate-tempo-e-loot.md`](docs/16-widget-mapas-combate-tempo-e-loot.md): experiência visual jogável.
- [`docs/17-autenticacao-identidade-seguranca-e-observabilidade.md`](docs/17-autenticacao-identidade-seguranca-e-observabilidade.md): OAuth, autorização, privacidade e tracing.
- [`docs/18-roadmap-de-implementacao-para-o-codex.md`](docs/18-roadmap-de-implementacao-para-o-codex.md): fases e primeiro prompt de auditoria.

### Legado

- [`docs/legacy/01-arquitetura-custom-gpt-actions.md`](docs/legacy/01-arquitetura-custom-gpt-actions.md)

## Versões atuais

```text
attributes-v0.4
combat-v0.2
equipment-v0.5
actions-v0.1
economy-v0.3
skills-v0.1
crafting-v0.2
actors-v0.1
inventory-v0.2
mcp-contracts-v0.1
bundles-v0.2
master-runtime-v0.2
app-architecture-v0.1
widget-gameplay-v0.1
security-v0.1
implementation-roadmap-v0.1
integration-v1.0
```

## Próximos sistemas de regra

1. Condições e Efeitos.
2. Progressão, Níveis e Experiência.
3. Encontros, Objetivos e Transições.
4. Relações, Vínculos e Companheiros.
5. Missões e Recompensas.
6. Tempo, Descanso e Recuperação.
7. Mundo, Locais e Exploração.
8. Morte, Incapacidade e Consequências.
9. Facções, Reputação e Crimes.

## Diretriz para o Codex

O Codex deve começar por auditoria do repositório `RalphCajazeira/cronicas-de-outro-mundo` na branch `develop`, comparar o estado atual com estas regras e propor migração incremental.

Não fazer reescrita ampla nem migration destrutiva antes da auditoria, matriz de reaproveitamento, plano expand/contract e critérios de aceite.

Seguir:

`docs/18-roadmap-de-implementacao-para-o-codex.md`

## Convenções

- nomes exibidos em português;
- códigos e campos em inglês;
- UUID como identidade principal;
- distâncias em metros;
- combate em ticks;
- moeda `CROWN`;
- backend como autoridade oficial;
- widget sem regra autoritativa duplicada;
- GPT não inventa recursos mecânicos ausentes;
- informações do Mestre não aparecem no widget;
- toda mudança de regra atualiza a versão correspondente.

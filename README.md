# Regras Game GPT

Base de conhecimento para um RPG conduzido por um GPT com Actions, backend de validação/persistência e futuro frontend de acompanhamento.

## Objetivo

O GPT interpreta as decisões do jogador, conduz a narrativa e resolve interações usando regras e estados carregados. O backend evita perda de estado, valida resultados, persiste alterações e executa consequências autoritativas, sem precisar ser chamado para cada microetapa de combate, comércio ou exploração.

O combate utiliza posições em metros, ações temporizadas e uma linha do tempo contínua. O GPT recebe tudo que precisa no snapshot, mantém a fila de eventos localmente e envia o resultado consolidado ao backend.

## Documentos

- [`docs/01-sistema-de-atributos.md`](docs/01-sistema-de-atributos.md): atributos primários, secundários, progressão, movimento e velocidades de ação.
- [`docs/02-sistema-de-combate.md`](docs/02-sistema-de-combate.md): linha do tempo contínua, posições em metros, chance única de acerto, crítico, defesa e fluxo do encontro.
- [`docs/03-sistema-de-equipamentos.md`](docs/03-sistema-de-equipamentos.md): estrutura universal, slots, qualidade, orçamento de pontos, penalidades, ações concedidas, peso e preços-base.
- [`docs/04-integracao-gpt-backend.md`](docs/04-integracao-gpt-backend.md): divisão de responsabilidades, snapshots, fila de eventos, resolução local e persistência final.
- [`docs/05-decisoes-e-pendencias.md`](docs/05-decisoes-e-pendencias.md): decisões consolidadas e pontos ainda sujeitos a simulações.
- [`docs/06-sistema-de-economia-e-comercio.md`](docs/06-sistema-de-economia-e-comercio.md): moeda única, escala econômica, comerciantes, negociação, estoque, recompensas e transações.
- [`docs/07-sistema-de-acoes-habilidades-e-magias.md`](docs/07-sistema-de-acoes-habilidades-e-magias.md): estrutura universal de ações, alcances em metros, áreas, preparação, recuperação, custos, interrupção e reações.

## Estado atual

Os documentos representam uma especificação inicial em evolução. Fórmulas, tempos, multiplicadores, preços e faixas devem ser versionados e validados por simulações antes de serem tratados como definitivos.

## Convenções

- Nomes exibidos ao jogador ficam em português.
- Códigos persistidos e campos de API usam inglês em `camelCase` ou enums em `SCREAMING_SNAKE_CASE`.
- Distâncias e áreas usam metros.
- O tempo de combate usa `ticks`, com proposta inicial de 10 ticks por segundo.
- A moeda canônica é a Coroa, persistida como `CROWN`.
- O backend é a autoridade de persistência e validação.
- O GPT não inventa campos mecânicos, ações, saldos, estoques ou preços ausentes.
- Toda mudança de regra deve atualizar a versão correspondente.
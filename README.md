# Regras Game GPT

Base de conhecimento para um RPG conduzido por um GPT com Actions, backend de validação/persistência e futuro frontend de acompanhamento.

## Objetivo

O GPT interpreta as decisões do jogador, conduz a narrativa e resolve interações usando regras e estados carregados. O backend evita perda de estado, valida resultados, persiste alterações e executa consequências autoritativas, sem precisar ser chamado para cada microetapa de combate, comércio ou exploração.

## Documentos

- [`docs/01-sistema-de-atributos.md`](docs/01-sistema-de-atributos.md): atributos primários, secundários, progressão e composição.
- [`docs/02-sistema-de-combate.md`](docs/02-sistema-de-combate.md): chance única de acerto, crítico, defesa crítica, mitigação e fluxo de combate.
- [`docs/03-sistema-de-equipamentos.md`](docs/03-sistema-de-equipamentos.md): estrutura universal, slots, qualidade, orçamento de pontos, penalidades, peso e preços-base.
- [`docs/04-integracao-gpt-backend.md`](docs/04-integracao-gpt-backend.md): divisão de responsabilidades, snapshots, resolução local e persistência final.
- [`docs/05-decisoes-e-pendencias.md`](docs/05-decisoes-e-pendencias.md): decisões consolidadas e pontos ainda sujeitos a simulações.
- [`docs/06-sistema-de-economia-e-comercio.md`](docs/06-sistema-de-economia-e-comercio.md): moeda única, escala econômica, comerciantes, negociação, estoque, recompensas e transações.

## Estado atual

Os documentos representam uma especificação inicial em evolução. Fórmulas, multiplicadores, preços e faixas devem ser versionados e validados por simulações antes de serem tratados como definitivos.

## Convenções

- Nomes exibidos ao jogador ficam em português.
- Códigos persistidos e campos de API usam inglês em `camelCase` ou enums em `SCREAMING_SNAKE_CASE`.
- A moeda canônica é a Coroa, persistida como `CROWN`.
- O backend é a autoridade de persistência e validação.
- O GPT não inventa campos mecânicos, saldos, estoques ou preços ausentes.
- Toda mudança de regra deve atualizar a versão correspondente.

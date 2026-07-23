# Regras Game GPT

Base de conhecimento para um RPG conduzido por GPT com Actions, backend de validação/persistência e futuro frontend de acompanhamento.

## Objetivo

O GPT interpreta decisões, conduz a narrativa e resolve interações usando regras e snapshots carregados. O backend evita perda de estado, valida resultados, persiste alterações e aplica consequências autoritativas sem precisar ser chamado para cada microetapa.

A base utiliza cinco atributos primários universais. Especializações são representadas por atributos secundários, perícias, proficiências, profissões, habilidades, passivas, equipamentos e condições.

## Documentos

- [`docs/01-sistema-de-atributos.md`](docs/01-sistema-de-atributos.md): cinco atributos primários, secundários, progressão, movimento e velocidades.
- [`docs/02-sistema-de-combate.md`](docs/02-sistema-de-combate.md): linha do tempo contínua, posições em metros, chance única de acerto, crítico e defesa.
- [`docs/03-sistema-de-equipamentos.md`](docs/03-sistema-de-equipamentos.md): slots, qualidade, orçamento, penalidades, ações/passivas concedidas, peso e preços-base.
- [`docs/04-integracao-gpt-backend.md`](docs/04-integracao-gpt-backend.md): snapshots, sessões versionadas, resolução local e persistência final.
- [`docs/05-decisoes-e-pendencias.md`](docs/05-decisoes-e-pendencias.md): decisões consolidadas e validações pendentes.
- [`docs/06-sistema-de-economia-e-comercio.md`](docs/06-sistema-de-economia-e-comercio.md): moeda, preços, comerciantes, perícia de Comércio, passivas e transações.
- [`docs/07-sistema-de-acoes-habilidades-e-magias.md`](docs/07-sistema-de-acoes-habilidades-e-magias.md): ações universais, alcance em metros, áreas, preparação, recuperação, custos e interrupção.
- [`docs/08-sistema-de-pericias-profissoes-e-passivas.md`](docs/08-sistema-de-pericias-profissoes-e-passivas.md): perícias, proficiências, profissões, passivas, progressão pelo uso e sessões.
- [`docs/09-sistema-de-fabricacao-e-qualidade.md`](docs/09-sistema-de-fabricacao-e-qualidade.md): receitas, materiais, ferramentas, oficinas, sucesso, qualidade e XP de produção.

## Estado atual

Os documentos representam uma especificação em evolução. Fórmulas, tempos, multiplicadores, preços, pesos de perícias, limiares de qualidade e faixas devem ser versionados e simulados antes de versões `v1.0`.

## Convenções

- Atributos primários: Força, Agilidade, Destreza, Vitalidade e Inteligência.
- Nomes exibidos ficam em português.
- Códigos e campos de API usam inglês em `camelCase` ou enums em `SCREAMING_SNAKE_CASE`.
- Distâncias e áreas usam metros.
- O tempo de combate usa ticks, com proposta inicial de 10 ticks por segundo.
- A moeda canônica é a Coroa, persistida como `CROWN`.
- O backend é autoridade de cálculo, validação e persistência.
- O GPT não inventa atributos, perícias, passivas, receitas, ações, saldos, estoques ou preços ausentes.
- Toda mudança de regra atualiza a versão correspondente.

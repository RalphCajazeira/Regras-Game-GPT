# Regras Game GPT

Base de conhecimento para um RPG conduzido por GPT com Actions, backend de validação/persistência e futuro frontend de acompanhamento.

## Objetivo

O GPT interpreta decisões, conduz a narrativa, cria e revisa conteúdos e resolve interações usando regras e snapshots carregados. O backend valida, calcula e persiste consequências sem precisar ser chamado para cada microetapa.

A base utiliza cinco atributos primários universais. Especializações usam atributos secundários, perícias, proficiências, profissões, habilidades, passivas, equipamentos e condições.

Entidades e itens são materializados quando se tornam relevantes. Depois disso, o GPT usa somente os recursos existentes no snapshot.

## Arquitetura das Actions

O GPT possui limite de 30 Actions expostas.

```text
REST + OpenAPI
22 Actions planejadas
8 vagas reservadas
```

As Actions representam domínios e fluxos completos. Microoperações usam enums internos tipados. Serviços e endpoints internos do backend não precisam ser expostos ao GPT.

Novos sistemas devem consultar o orçamento de Actions antes de criar um novo `operationId`.

## Regra canônica de itens

```text
Definição única com perfil Comum
→ variante apenas para diferença real
→ qualidade na instância
→ valores resolvidos e congelados
```

Uma Adaga Élfica possui um único cadastro reutilizável. Instâncias Comuns, Raras, Épicas ou Lendárias apontam para a mesma definição, salvo quando houver uma variante mecanicamente diferente.

## Documentos

- [`docs/01-sistema-de-atributos.md`](docs/01-sistema-de-atributos.md): cinco atributos primários, secundários, progressão, movimento e velocidades.
- [`docs/02-sistema-de-combate.md`](docs/02-sistema-de-combate.md): linha do tempo contínua, posições em metros, chance única de acerto, crítico e defesa.
- [`docs/03-sistema-de-equipamentos.md`](docs/03-sistema-de-equipamentos.md): definição única, variantes, qualidade por instância, slots, orçamento, modificadores e preços-base.
- [`docs/04-integracao-gpt-backend.md`](docs/04-integracao-gpt-backend.md): Action Gateway, snapshots, sessões versionadas, criação de instâncias, revisão e persistência.
- [`docs/05-decisoes-e-pendencias.md`](docs/05-decisoes-e-pendencias.md): decisões consolidadas, orçamento de Actions e validações pendentes.
- [`docs/06-sistema-de-economia-e-comercio.md`](docs/06-sistema-de-economia-e-comercio.md): moeda, preço Comum da definição, preço resolvido da instância, comerciantes e transações.
- [`docs/07-sistema-de-acoes-habilidades-e-magias.md`](docs/07-sistema-de-acoes-habilidades-e-magias.md): ações universais, alcance, áreas, preparação, recuperação, custos e interrupção.
- [`docs/08-sistema-de-pericias-profissoes-e-passivas.md`](docs/08-sistema-de-pericias-profissoes-e-passivas.md): perícias, proficiências, profissões, passivas, progressão pelo uso e sessões.
- [`docs/09-sistema-de-fabricacao-e-qualidade.md`](docs/09-sistema-de-fabricacao-e-qualidade.md): receitas que produzem instâncias, sucesso, qualidade, valores resolvidos e XP.
- [`docs/10-sistema-de-atores-definicoes-e-instancias.md`](docs/10-sistema-de-atores-definicoes-e-instancias.md): modelo universal de atores, materialização sob demanda, persistência, relações e revisão.
- [`docs/11-sistema-de-inventario-itens-drops-e-saque.md`](docs/11-sistema-de-inventario-itens-drops-e-saque.md): definições, variantes, instâncias, pilhas, peso, propriedade, drops e saque.
- [`docs/12-contratos-operacionais-actions-erros-e-recuperacao.md`](docs/12-contratos-operacionais-actions-erros-e-recuperacao.md): limite de Actions, catálogo canônico, envelopes, idempotência, erros acionáveis e recuperação do GPT.

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
operations-v0.1
integration-v0.8
```

## Estado atual

Os documentos representam uma especificação em evolução. Fórmulas, tempos, multiplicadores, preços, custos, limiares, pilhas, carga, durabilidade e contratos operacionais devem ser simulados antes de versões `v1.0`.

## Convenções

- Atributos primários: Força, Agilidade, Destreza, Vitalidade e Inteligência.
- Nomes exibidos ficam em português.
- Códigos e campos de API usam inglês em `camelCase` ou enums em `SCREAMING_SNAKE_CASE`.
- Distâncias e áreas usam metros.
- O tempo de combate usa ticks, com proposta inicial de 10 ticks por segundo.
- A moeda canônica é a Coroa, persistida como `CROWN`.
- A qualidade Comum é a referência de definições escaláveis.
- Qualidade não gera cadastro duplicado.
- O backend é autoridade de cálculo, validação e persistência.
- O GPT pode criar e revisar conteúdos por operações válidas, mas não altera retroativamente snapshots em andamento.
- O GPT não inventa atributos, perícias, passivas, receitas, ações, itens, quantidades, saldos, estoques, drops ou preços ausentes.
- Toda mutação crítica usa idempotência e controle por `stateVersion`.
- Erros devem indicar recuperação e próxima operação possível.
- O OpenAPI exposto ao GPT não pode ultrapassar 30 `operationId`s.
- Toda mudança de regra atualiza a versão correspondente.
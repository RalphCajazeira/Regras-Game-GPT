# Governança de Estado, Roadmap e Mudanças

**Versão:** `state-governance-v0.1`  
**Status:** canônica

## 1. Objetivo

Separar claramente:

- regra desejada;
- decisão aprovada;
- funcionalidade planejada;
- funcionalidade implementada;
- funcionalidade implantada;
- funcionalidade validada manualmente;
- funcionalidade abandonada ou substituída.

Sem essa separação, documentos de produto podem parecer evidência de código existente e relatórios antigos podem parecer mais confiáveis que o ambiente atual.

## 2. Responsabilidade dos repositórios

### `RalphCajazeira/Regras-Game-GPT`

Fonte canônica para:

- regras de domínio;
- arquitetura-alvo;
- decisões de produto;
- princípios de segurança;
- contratos conceituais;
- funcionalidades desejadas;
- sistemas futuros;
- convenções e versões de regra.

Responde:

```text
O que o jogo deve fazer?
Por que essa decisão existe?
Qual comportamento é canônico?
```

### `RalphCajazeira/cronicas-de-outro-mundo`

Fonte operacional para:

- código atual;
- schema e migrations;
- serviços;
- REST, MCP e extensão;
- testes;
- CI;
- rollout;
- staging;
- limitações reais;
- roadmap executável;
- backlog priorizado.

Responde:

```text
O que existe hoje?
Onde está implementado?
Em qual ambiente foi validado?
O que falta fazer em seguida?
```

## 3. Prioridade de evidências

Quando houver divergência:

1. ambiente, banco ou serviço atual;
2. Git e arquivos atuais;
3. testes, build e CI atuais;
4. contratos e documentos vivos do repositório executável;
5. regras canônicas;
6. relatórios anteriores;
7. memória e hipóteses.

Uma regra canônica não prova implementação. Um código implantado não altera automaticamente uma regra canônica sem decisão registrada.

## 4. Documentos vivos obrigatórios

No repositório executável:

```text
docs/ai/project/PROJECT_STATE.md
docs/ai/project/ROADMAP.md
docs/ai/project/DECISIONS.md
```

### `PROJECT_STATE.md`

Registra o estado real por capacidade.

Campos mínimos:

- capacidade;
- status;
- localização no código;
- ambiente validado;
- evidências;
- limitações;
- próxima ação.

### `ROADMAP.md`

Registra a ordem atual de execução.

Cada item deve conter:

- identificador;
- objetivo;
- dependências;
- critérios de aceite;
- fora de escopo;
- estado;
- PRs/Issues relacionados.

### `DECISIONS.md`

Registra mudanças de direção e decisões que afetam mais de uma task.

Exemplos:

- extensão substituiu widget como frontend principal;
- GPT final não terá Actions;
- realtime será invalidação, não estado;
- widget permanece como fallback;
- staging não exige aprovação manual depois de uma task autorizada.

## 5. Vocabulário de status

Usar apenas:

```text
NOT_STARTED
IN_PROGRESS
IMPLEMENTED_LOCAL
INTEGRATED
DEPLOYED_STAGING
MANUALLY_VALIDATED
BLOCKED
DEPRECATED
REMOVED
```

### Significados

- `NOT_STARTED`: sem implementação iniciada;
- `IN_PROGRESS`: trabalho ativo, não integrado;
- `IMPLEMENTED_LOCAL`: código e testes locais prontos, sem merge;
- `INTEGRATED`: mesclado na branch de integração;
- `DEPLOYED_STAGING`: implantado e com checks automáticos aprovados;
- `MANUALLY_VALIDATED`: fluxo real comprovado manualmente no ambiente-alvo;
- `BLOCKED`: impedimento técnico ou de produto explícito;
- `DEPRECATED`: ainda existe, mas não recebe evolução principal;
- `REMOVED`: retirado do produto ou código aplicável.

Não usar “concluído” sem indicar o nível de evidência.

## 6. Capacidade versus task

Uma task é um change set. Uma capacidade é uma funcionalidade do produto.

Exemplo:

```text
Capacidade: autenticação da extensão

Tasks:
- endpoint OAuth
- armazenamento seguro
- refresh
- logout
- testes de revogação
```

O roadmap organiza tasks. O estado organiza capacidades.

## 7. Backlog dinâmico

Ideias e problemas descobertos não entram automaticamente como decisão canônica.

Devem virar GitHub Issues com classificação:

```text
type: bug
type: feature
type: change
type: removal
type: technical-debt

area: extension
area: backend
area: oauth
area: mcp
area: gameplay
area: combat
area: inventory
area: progression
area: security
```

A Issue deve informar:

- problema ou oportunidade;
- evidência;
- impacto;
- proposta inicial;
- riscos;
- decisão pendente;
- critérios de aceite quando priorizada.

## 8. Atualização por PR

Todo PR que altera uma capacidade, arquitetura ou rollout deve:

1. atualizar `PROJECT_STATE.md`; ou
2. declarar no corpo do PR que não altera estado de produto.

Todo PR que muda a ordem das próximas entregas deve atualizar `ROADMAP.md`.

Todo PR que muda uma decisão canônica deve atualizar:

- `DECISIONS.md` no repositório executável;
- o documento correspondente em `Regras-Game-GPT`.

## 9. Regra para funcionalidades novas, alteradas ou removidas

### Nova funcionalidade

```text
ideia
→ Issue
→ decisão/priorização
→ ROADMAP
→ task
→ implementação
→ PROJECT_STATE
→ validação
```

### Mudança de funcionalidade

Registrar:

- comportamento anterior;
- comportamento novo;
- motivo;
- compatibilidade;
- migração;
- riscos;
- data/PR da decisão.

### Remoção

Registrar:

- capacidade removida;
- motivo;
- substituto;
- impacto em dados/contratos;
- rollback, quando aplicável;
- status `DEPRECATED` antes de `REMOVED`, salvo risco crítico.

## 10. Regras versus implementação parcial

Quando uma regra ainda não estiver implementada:

- manter a regra como estado-alvo;
- marcar a capacidade como `NOT_STARTED` ou parcial no `PROJECT_STATE.md`;
- não criar defaults silenciosos;
- não fingir equivalência;
- não bloquear testes de recortes independentes sem necessidade.

Quando o código divergir da regra:

- registrar a divergência;
- classificar risco;
- decidir entre adaptar código, atualizar regra ou manter compatibilidade temporária.

## 11. Evidências mínimas

### `INTEGRATED`

- PR mesclado;
- branch de integração sincronizada;
- CI relevante aprovado.

### `DEPLOYED_STAGING`

- SHA exato implantado;
- migration state confirmado;
- health/readiness aprovados;
- smoke aplicável aprovado.

### `MANUALLY_VALIDATED`

- fluxo real executado;
- identidade/ambiente confirmados;
- resultado observado;
- limitações registradas;
- nenhuma substituição por teste automatizado quando o critério era visual ou operacional.

## 12. Manutenção periódica

Revisar os documentos vivos:

- após cada fase importante;
- após mudança arquitetural;
- quando surgir divergência relevante;
- antes de iniciar uma nova frente grande;
- antes de produção.

Não atualizar datas ou status apenas para aparentar atividade.

## 13. Uso pelo ChatGPT e Codex

Antes de criar prompt de implementação:

1. ler `PROJECT_STATE.md`;
2. ler `ROADMAP.md`;
3. consultar `DECISIONS.md` somente para decisões relacionadas;
4. consultar estas regras canônicas;
5. confirmar o estado real no repositório e ambiente.

No mesmo chat, não reler tudo por rotina. Reabrir documentos somente quando houver mudança, conflito, perda de contexto ou detalhe necessário ausente.

## 14. Regra final

```text
regra desejada ≠ implementação comprovada
PR mesclado ≠ staging validado
staging validado ≠ produção
feito em teste automatizado ≠ validado visualmente
ideia registrada ≠ decisão aprovada
```

A documentação deve tornar essas diferenças explícitas.
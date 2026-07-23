# Criação, Validação e Materialização em Lote

**Versão da proposta:** `bundles-v0.2`  
**Status:** em validação

## 1. Objetivo

Permitir que GPT, widget, administração ou frontend externo enviem um pacote completo de conteúdos relacionados, recebam validação integral e, quando autorizado, tenham definições, instâncias e vínculos persistidos em uma única transação.

## 2. Entradas permitidas

```text
GPT → cria ou revisa conteúdo narrativo e mecânico
Widget → envia comandos estruturados e seleções
Administração → revisa e migra
Frontend externo → usa os mesmos serviços
```

Todos chegam ao mesmo serviço de bundle por MCP ou REST interno.

## 3. Princípios

1. O pacote pode ser montado em memória antes da chamada.
2. Componentes não aplicáveis são opcionais.
3. Componentes enviados são validados pelo próprio domínio.
4. Todos os problemas independentes detectáveis são retornados juntos.
5. Conteúdo existente é reutilizado quando compatível.
6. Conteúdo inexistente pode ser criado no mesmo pacote.
7. Persistência completa usa `ALL_OR_NOTHING` por padrão.
8. Materialização temporária não polui o catálogo canônico.
9. Conteúdo temporário pode ser promovido depois.
10. Cada erro informa localização, regra, esperado, recebido e recuperação.
11. Operações são idempotentes.
12. Relações usam UUIDs persistidos e `clientRef` durante a montagem.

## 4. Componentes de ator

### Núcleo mínimo

- identidade técnica;
- natureza e espécie;
- nível ou escala equivalente;
- atributos primários;
- secundários calculados;
- recursos aplicáveis;
- estado;
- comportamento mínimo;
- versões.

### Componentes opcionais

```text
combat
inventory
equipment
skills
proficiencies
abilities
spells
passives
loot
merchant
relationships
companion
missionLinks
crafting
```

O backend distingue:

```text
OPTIONAL_NOT_PROVIDED
NOT_APPLICABLE
REQUIRED_MISSING
PROVIDED_INVALID
```

## 5. Modos

```text
VALIDATE_ONLY
MATERIALIZE_TEMPORARY
PERSIST_IF_VALID
PERSIST_REQUIRED
PROMOTE_TEMPORARY
```

### `VALIDATE_ONLY`

Valida, normaliza, resolve referências e calcula prontidão sem criar estado utilizável.

### `MATERIALIZE_TEMPORARY`

Gera snapshot temporário e versionado para combate, avaliação, simulação ou cena provisória.

### `PERSIST_IF_VALID`

Persiste somente quando o pacote inteiro estiver válido.

### `PERSIST_REQUIRED`

Exige criação persistente completa.

### `PROMOTE_TEMPORARY`

Promove materialização preservando identidade, eventos e vínculos quando possível.

## 6. Identificadores

```text
UUID      → identidade técnica
code      → definição legível
version   → versão da definição
clientRef → referência local dentro do bundle
sequence  → ordenação e número amigável
```

## 7. Estratégias por nó

```text
REUSE_REQUIRED
REUSE_OR_CREATE
CREATE_NEW
CREATE_VARIANT
INLINE_TEMPORARY
UPDATE_EXISTING
REVIEW_EXISTING
```

Qualidade diferente nunca cria automaticamente nova definição de item.

## 8. Ferramenta MCP

Ferramenta canônica:

```text
manageContentBundle
```

Comandos:

```text
VALIDATE_BUNDLE
MATERIALIZE_BUNDLE
PERSIST_BUNDLE
PROMOTE_BUNDLE
REVIEW_BUNDLE
MIGRATE_BUNDLE
```

`materializeActors` pode chamar internamente o mesmo serviço para criação de atores em lote.

## 9. Estrutura resumida

```json
{
  "command": "MATERIALIZE_BUNDLE",
  "requestId": "uuid",
  "idempotencyKey": "uuid",
  "baseStateVersion": 15,
  "mode": "MATERIALIZE_TEMPORARY",
  "atomicity": "ALL_OR_NOTHING",
  "context": {
    "worldId": "uuid",
    "campaignId": "uuid",
    "requestedReadinessProfiles": ["COMBAT_READY"]
  },
  "nodes": [
    {
      "clientRef": "actor:bandit-leader",
      "nodeType": "ACTOR_INSTANCE",
      "resolutionStrategy": "INLINE_TEMPORARY",
      "data": {}
    },
    {
      "clientRef": "definition:iron-dagger",
      "nodeType": "ITEM_DEFINITION",
      "resolutionStrategy": "REUSE_REQUIRED",
      "selector": { "code": "iron-dagger" }
    }
  ],
  "relations": []
}
```

## 10. Validação

Ordem mínima:

```text
schema
autorização
referências e clientRefs
conteúdo equivalente
aplicabilidade
atributos e orçamento
secundários
habilidades, magias e passivas
equipamentos e poder
inventário, peso e localização
perícias e progressão
drops e economia
relações e vínculos
validação cruzada
dependency closure
readiness
plano de persistência
```

## 11. Erro por orçamento

```json
{
  "code": "PRIMARY_ATTRIBUTE_BUDGET_MISMATCH",
  "blocking": true,
  "clientRef": "actor:bandit-leader",
  "domain": "ATTRIBUTES",
  "path": "/nodes/0/data/primaryAttributes",
  "ruleCode": "PRIMARY_POINTS_BY_LEVEL",
  "expected": { "totalPrimaryPoints": 81 },
  "received": { "totalPrimaryPoints": 79 },
  "difference": 2,
  "recovery": {
    "tool": "manageContentBundle",
    "command": "VALIDATE_BUNDLE"
  }
}
```

## 12. Resposta inválida acumulativa

```json
{
  "status": "REQUIRES_REVIEW",
  "validationStatus": "INVALID",
  "persisted": false,
  "issues": [
    { "clientRef": "actor:bandit-leader", "code": "PRIMARY_ATTRIBUTE_BUDGET_MISMATCH" },
    { "clientRef": "action:flame-slash", "code": "ACTION_COST_MISSING" },
    { "clientRef": "item:leader-dagger", "code": "POWER_BUDGET_EXCEEDED" }
  ],
  "recoveryTools": [
    { "tool": "manageContentBundle", "command": "VALIDATE_BUNDLE" }
  ]
}
```

Com `ALL_OR_NOTHING`, nenhum nó é persistido quando houver erro bloqueante.

## 13. Resposta válida

```json
{
  "status": "SUCCESS",
  "validationStatus": "VALID",
  "persisted": true,
  "bundleId": "uuid",
  "referenceMap": {
    "actor:bandit-leader": { "id": "uuid", "result": "CREATED" },
    "definition:iron-dagger": { "id": "uuid", "code": "iron-dagger", "result": "REUSED" }
  },
  "readiness": {
    "COMBAT_READY": { "status": "READY" }
  },
  "snapshot": {},
  "newStateVersion": 16
}
```

## 14. Materialização temporária e widget

A materialização temporária pode alimentar imediatamente:

- mapa tático;
- ficha de avaliação;
- painel de diálogo;
- inventário do encontro;
- loot futuro;
- sessão criativa do GPT.

Conteúdo secreto deve ser separado dos dados enviados ao widget.

## 15. Perfis de prontidão

```text
ACTOR_COMPLETE
COMBAT_READY
TRADE_READY
CRAFT_READY
EVALUATION_READY
COMPANION_READY
PERSISTENCE_READY
WIDGET_RENDER_READY
```

`WIDGET_RENDER_READY` verifica que os dados públicos necessários à interface estão disponíveis sem vazar informações do Mestre.

## 16. Plano de persistência

Antes do commit:

```text
REUSE definições
CREATE definições faltantes autorizadas
CREATE instâncias
LINK relações
CALCULATE valores derivados
VALIDATE dependency closure
VALIDATE readiness
COMMIT
```

No modo de validação, o plano pode ser retornado sem execução.

## 17. Segurança

- identidade deriva do token;
- cada nó e relação valida autorização;
- bundle não permite acesso cruzado;
- segredos não entram no widget;
- `clientRef` não é identificador persistente;
- payload diferente com mesma idempotencyKey é rejeitado.

## 18. Critérios para o Codex

1. Implementar schemas discriminados por `nodeType`.
2. Implementar validadores acumulativos.
3. Implementar resolver de referências.
4. Implementar pesquisa de equivalência.
5. Implementar plano transacional.
6. Implementar materialização temporária recuperável.
7. Implementar promoção sem duplicação.
8. Integrar MCP, REST e administração ao mesmo serviço.
9. Testar rollback e idempotência.
10. Testar conteúdo público versus secreto.

# Criação, Validação e Materialização em Lote

**Versão da proposta:** `bundles-v0.1`  
**Status:** em validação

## 1. Objetivo

Permitir que o GPT envie ao backend um pacote completo de conteúdos relacionados, receba a validação integral de todos os componentes e, quando autorizado, tenha definições, instâncias e vínculos persistidos em uma única transação.

O mesmo contrato deve atender:

- criação de personagem jogável;
- materialização de NPC, animal, monstro, criatura ou espírito;
- criação de grupos;
- preparação temporária para combate, Avaliação ou outra interação;
- criação de equipamentos, itens, ações, habilidades, magias, perícias e passivas vinculadas;
- reutilização de conteúdos existentes;
- promoção posterior de conteúdo temporário para persistente.

## 2. Princípios

1. O GPT pode montar o pacote completo em memória antes de chamar o backend.
2. O backend valida todos os componentes aplicáveis e todos os vínculos antes de persistir.
3. Componentes não aplicáveis são opcionais; sua ausência não é erro.
4. Componentes enviados são obrigatoriamente validados pelas regras do próprio domínio.
5. A validação deve acumular todos os problemas detectáveis na mesma resposta.
6. O backend não deve falhar no primeiro erro e ocultar os demais problemas já identificáveis.
7. Conteúdo existente deve ser reutilizado sempre que compatível.
8. Conteúdo inexistente pode ser criado dentro do mesmo pacote quando a política permitir.
9. A persistência padrão de um pacote completo é atômica: tudo ou nada.
10. O backend pode validar ou materializar temporariamente sem persistir definições canônicas.
11. Conteúdo temporário pode ser promovido depois sem reconstruir toda a interação.
12. O GPT recebe erros localizados, valores esperados, valores recebidos e sugestões de correção.
13. O pacote e seus resultados são idempotentes.
14. Todas as entidades materializadas ou persistidas usam identidade técnica estável.

## 3. Estrutura componente de ator

Nem todo ator possui todas as capacidades.

### 3.1 Núcleo mínimo

Todo ator materializado possui, conforme a natureza do mundo:

- identidade técnica;
- natureza e espécie;
- nível ou escala equivalente;
- atributos primários;
- atributos secundários calculados;
- recursos aplicáveis;
- estado atual;
- comportamento mínimo;
- versão das regras utilizadas.

### 3.2 Componentes opcionais

Podem existir apenas quando aplicáveis:

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

Exemplos:

- um animal comum pode não possuir equipamento, dinheiro ou magia;
- um espírito pode possuir magia e passivas, mas nenhum inventário físico;
- um comerciante pode possuir estoque e Comércio, mas não ser preparado para combate;
- um bandido pode possuir arma, dinheiro, consumível e ações de combate;
- um objeto animado pode possuir ações e recursos, mas não relações sociais.

A ausência de um componente opcional não deve gerar campos mecânicos inventados pelo backend ou pelo GPT.

## 4. Modos do pacote

```text
VALIDATE_ONLY
MATERIALIZE_TEMPORARY
PERSIST_IF_VALID
PERSIST_REQUIRED
PROMOTE_TEMPORARY
```

### 4.1 `VALIDATE_ONLY`

Valida, normaliza, resolve referências e calcula prontidão sem criar estado persistente ou temporário utilizável.

### 4.2 `MATERIALIZE_TEMPORARY`

Cria um snapshot temporário, versionado e utilizável por uma sessão, sem transformar todos os nós em cadastros canônicos.

Pode ser usado para:

- combate com inimigos anônimos;
- grupo de bandidos que pode nunca reaparecer;
- criatura avaliada e depois ignorada;
- simulação ou teste;
- conteúdo narrativamente provisório.

### 4.3 `PERSIST_IF_VALID`

Se o pacote inteiro estiver válido, persiste definições, instâncias e vínculos necessários. Se houver qualquer erro bloqueante, nada é persistido.

### 4.4 `PERSIST_REQUIRED`

Exige persistência completa e rejeita estratégias temporárias incompatíveis.

### 4.5 `PROMOTE_TEMPORARY`

Promove um pacote ou entidade temporária para escopo persistente, preservando identidade de materialização, histórico e vínculos sempre que possível.

## 5. Identificadores

### 5.1 UUID técnico

Usar UUID como chave primária para:

- definições;
- variantes;
- atores materializados;
- instâncias de itens;
- sessões;
- encontros;
- relações;
- missões;
- contêineres;
- transações;
- pacotes de criação.

### 5.2 Código estável

Definições reutilizáveis também possuem:

```text
code
version
```

Exemplo:

```json
{
  "id": "uuid-da-definicao",
  "code": "elven-dagger",
  "version": 3
}
```

O `code` facilita busca e reutilização. O UUID identifica tecnicamente o registro. A versão identifica a definição utilizada.

### 5.3 `clientRef`

Durante a montagem do pacote, o GPT usa referências locais legíveis:

```text
actor:bandit-leader
item:leader-dagger
action:dagger-attack
skill:intimidation
```

O `clientRef` existe somente dentro daquele pacote e permite criar relações antes de o banco gerar os IDs definitivos.

### 5.4 Sequências numéricas

Usar números sequenciais para:

- ordem de eventos;
- número de revisão;
- versão;
- número amigável de missão ou encontro;
- ordenação de logs.

Números sequenciais não substituem a identidade técnica principal.

## 6. Estratégia por nó

Cada nó declara como deve ser resolvido:

```text
REUSE_REQUIRED
REUSE_OR_CREATE
CREATE_NEW
CREATE_VARIANT
INLINE_TEMPORARY
UPDATE_EXISTING
REVIEW_EXISTING
```

### `REUSE_REQUIRED`

O conteúdo precisa existir e ser compatível. O pacote falha se não houver correspondência única.

### `REUSE_OR_CREATE`

O backend procura conteúdo equivalente. Se encontrar, reutiliza. Se não encontrar e a proposta for válida, cria.

### `CREATE_NEW`

Cria uma nova definição ou instância, mas ainda executa verificação de duplicação semântica.

### `CREATE_VARIANT`

Cria variante apenas quando existir diferença real de identidade ou mecânica.

### `INLINE_TEMPORARY`

Usa o conteúdo somente no snapshot temporário. Não cria definição canônica naquele momento.

### `UPDATE_EXISTING` e `REVIEW_EXISTING`

Aplicam fluxo versionado de alteração, com escopo e impacto explícitos.

## 7. Estrutura do pacote

```json
{
  "operation": "VALIDATE_OR_MATERIALIZE_BUNDLE",
  "requestId": "request-id",
  "idempotencyKey": "bundle-idempotency-key",
  "mode": "MATERIALIZE_TEMPORARY",
  "atomicity": "ALL_OR_NOTHING",
  "context": {
    "worldId": "world-id",
    "campaignId": "campaign-id",
    "encounterId": "encounter-id",
    "requestedReadinessProfiles": ["COMBAT_READY"]
  },
  "nodes": [
    {
      "clientRef": "actor:bandit-leader",
      "nodeType": "ACTOR_INSTANCE",
      "resolutionStrategy": "INLINE_TEMPORARY",
      "data": {
        "name": "Líder Bandido",
        "nature": "PERSON",
        "speciesCode": "human",
        "archetypeCode": "bandit-leader",
        "level": 5,
        "primaryAttributes": {
          "strength": 18,
          "agility": 15,
          "dexterity": 17,
          "vitality": 16,
          "intelligence": 15
        }
      }
    },
    {
      "clientRef": "definition:iron-dagger",
      "nodeType": "ITEM_DEFINITION",
      "resolutionStrategy": "REUSE_REQUIRED",
      "selector": {
        "code": "iron-dagger"
      }
    },
    {
      "clientRef": "item:leader-dagger",
      "nodeType": "ITEM_INSTANCE",
      "resolutionStrategy": "INLINE_TEMPORARY",
      "data": {
        "definitionRef": "definition:iron-dagger",
        "quality": "COMMON"
      }
    }
  ],
  "relations": [
    {
      "relationType": "OWNS_ITEM",
      "fromRef": "actor:bandit-leader",
      "toRef": "item:leader-dagger"
    },
    {
      "relationType": "EQUIPPED_IN",
      "fromRef": "item:leader-dagger",
      "toRef": "actor:bandit-leader",
      "metadata": {
        "slot": "MAIN_HAND"
      }
    }
  ]
}
```

## 8. Etapas da validação

O backend executa, no mínimo:

```text
1. validação estrutural do envelope;
2. validação de schema por tipo de nó;
3. resolução de seletores, códigos, versões e clientRefs;
4. pesquisa de conteúdo equivalente;
5. validação de aplicabilidade dos componentes;
6. validação de atributos e orçamento por nível;
7. cálculo e validação de atributos secundários;
8. validação de ações, habilidades, magias e passivas;
9. validação de equipamentos, qualidade e orçamento de poder;
10. validação de inventário, quantidades, peso e localização;
11. validação de perícias, proficiências e progressão;
12. validação de drops, recompensas e economia;
13. validação de relações e vínculos;
14. validação cruzada entre todos os nós;
15. avaliação dos perfis de prontidão solicitados;
16. elaboração do plano de persistência;
17. persistência ou materialização temporária conforme o modo.
```

## 9. Validação por nível e orçamento

O backend é autoridade sobre os valores esperados.

Exemplo de erro em ator nível 5:

```json
{
  "code": "PRIMARY_ATTRIBUTE_BUDGET_MISMATCH",
  "severity": "ERROR",
  "blocking": true,
  "clientRef": "actor:bandit-leader",
  "domain": "ATTRIBUTES",
  "path": "/nodes/0/data/primaryAttributes",
  "ruleCode": "PRIMARY_POINTS_BY_LEVEL",
  "message": "A distribuição de atributos primários não utiliza o orçamento exigido para o nível 5.",
  "expected": {
    "level": 5,
    "basePoints": 25,
    "creationFreePoints": 16,
    "levelProgressionPoints": 40,
    "totalPrimaryPoints": 81
  },
  "received": {
    "totalPrimaryPoints": 79
  },
  "difference": 2,
  "recovery": {
    "canRetry": true,
    "suggestedAction": "REDISTRIBUTE_PRIMARY_ATTRIBUTES"
  }
}
```

O backend deve informar a regra vigente e não obrigar o GPT a deduzir o orçamento.

## 10. Resposta inválida acumulativa

Se o pacote tiver vários erros, a resposta deve reunir todos os problemas independentes detectáveis:

```json
{
  "status": "REQUIRES_REVIEW",
  "validationStatus": "INVALID",
  "persisted": false,
  "issues": [
    {
      "clientRef": "actor:bandit-leader",
      "code": "PRIMARY_ATTRIBUTE_BUDGET_MISMATCH",
      "path": "/nodes/0/data/primaryAttributes"
    },
    {
      "clientRef": "action:flame-slash",
      "code": "ACTION_COST_MISSING",
      "path": "/nodes/4/data/resourceCosts"
    },
    {
      "clientRef": "item:leader-dagger",
      "code": "POWER_BUDGET_EXCEEDED",
      "path": "/nodes/2/data/resolvedModifiers"
    }
  ],
  "warnings": [],
  "recoveryActions": [
    {
      "operationId": "manageContent",
      "operation": "VALIDATE_CONTENT_BUNDLE",
      "reuseRequestId": true
    }
  ]
}
```

Nenhum nó é persistido quando `atomicity` for `ALL_OR_NOTHING` e existir erro bloqueante.

## 11. Resposta válida persistida

```json
{
  "status": "SUCCESS",
  "validationStatus": "VALID",
  "persisted": true,
  "bundleId": "bundle-uuid",
  "bundleVersion": 1,
  "referenceMap": {
    "actor:bandit-leader": {
      "id": "actor-uuid",
      "result": "CREATED"
    },
    "definition:iron-dagger": {
      "id": "definition-uuid",
      "code": "iron-dagger",
      "version": 2,
      "result": "REUSED"
    },
    "item:leader-dagger": {
      "id": "item-instance-uuid",
      "result": "CREATED"
    }
  },
  "created": [],
  "reused": [],
  "updated": [],
  "relationsCreated": [],
  "readiness": {
    "COMBAT_READY": {
      "status": "READY",
      "issues": []
    }
  },
  "snapshot": {},
  "newStateVersion": 16,
  "warnings": [],
  "issues": []
}
```

## 12. Materialização temporária

Quando o pacote for válido em `MATERIALIZE_TEMPORARY`, o backend retorna:

```json
{
  "status": "SUCCESS",
  "validationStatus": "VALID",
  "persisted": false,
  "materializationId": "materialization-uuid",
  "materializationVersion": 1,
  "expiresAt": null,
  "referenceMap": {},
  "readiness": {
    "COMBAT_READY": {
      "status": "READY"
    }
  },
  "snapshot": {},
  "promotionPolicy": {
    "canPromote": true,
    "operationId": "manageContent",
    "operation": "PROMOTE_CONTENT_BUNDLE"
  }
}
```

A materialização temporária deve ser suficientemente estável para:

- participar de encontro;
- consumir recursos;
- sofrer dano e condições;
- gerar eventos;
- ser avaliada;
- ser promovida posteriormente.

Se for descartada, seus conteúdos não canônicos não poluem o catálogo permanente.

## 13. Perfis de prontidão

```text
ACTOR_COMPLETE
COMBAT_READY
TRADE_READY
CRAFT_READY
EVALUATION_READY
COMPANION_READY
PERSISTENCE_READY
```

### `COMBAT_READY`

Exige, quando aplicável:

- atributos e recursos válidos;
- ações completas;
- custos declarados;
- perfis de ataque ou magia completos;
- equipamentos e slots válidos;
- munições, consumíveis e cargas existentes;
- condições iniciais válidas;
- inventário materializado;
- linha de controle e facção definidas.

Um ator pode ser `ACTOR_COMPLETE` para narrativa e ainda não ser `COMBAT_READY`.

## 14. Plano de persistência

Antes do commit, o backend deve produzir internamente um plano:

```text
REUSE definition iron-dagger@2
CREATE temporary actor bandit-leader
CREATE temporary item instance leader-dagger
LINK actor owns item
LINK item equipped in MAIN_HAND
CALCULATE actor secondary attributes
CALCULATE item resolved modifiers
VALIDATE COMBAT_READY
```

No modo `VALIDATE_ONLY`, esse plano pode ser devolvido ao GPT sem execução.

## 15. Atomicidade

Políticas:

```text
ALL_OR_NOTHING
PARTIAL_BY_INDEPENDENT_GROUP
```

O padrão para criação de ator completo, materialização de combate e promoção é:

```text
ALL_OR_NOTHING
```

`PARTIAL_BY_INDEPENDENT_GROUP` só pode ser usado quando os grupos forem realmente independentes e a resposta identificar exatamente o que foi aceito ou rejeitado.

## 16. Idempotência

Repetir a mesma `idempotencyKey` com o mesmo conteúdo deve retornar o mesmo resultado autoritativo.

Repetir a chave com conteúdo diferente deve retornar conflito:

```text
IDEMPOTENCY_PAYLOAD_MISMATCH
```

O backend deve usar um hash normalizado do pacote para auditoria e promoção segura.

## 17. Integração com Actions

Não criar nova Action.

### `manageContent`

Operações:

```text
VALIDATE_CONTENT_BUNDLE
PERSIST_CONTENT_BUNDLE
PROMOTE_CONTENT_BUNDLE
REVIEW_CONTENT_BUNDLE
```

### `materializeActors`

Pode receber um pacote de ator completo ou uma referência a pacote validado:

```text
VALIDATE_AND_MATERIALIZE
MATERIALIZE_FROM_VALIDATED_BUNDLE
MATERIALIZE_GROUP
PROMOTE_MATERIALIZED_ACTOR
```

Os dois caminhos reutilizam os mesmos serviços internos de resolução, validação e persistência.

## 18. Responsabilidades do GPT

- pesquisar conteúdo semelhante antes da criação;
- reutilizar códigos e versões retornados;
- montar nós e relações com `clientRef`;
- indicar o modo desejado;
- enviar somente componentes aplicáveis;
- fornecer definições completas quando criar conteúdo novo;
- ler todos os erros e warnings;
- corrigir o pacote sem perder referências válidas;
- reenviar com a mesma intenção e nova tentativa idempotente apropriada;
- solicitar promoção quando conteúdo temporário se tornar relevante.

## 19. Responsabilidades do backend

- validar todos os nós e relações;
- calcular orçamentos e valores derivados;
- detectar duplicação e reutilizar conteúdo;
- distinguir ausência permitida de campo obrigatório ausente;
- retornar todos os erros detectáveis;
- localizar cada erro por `clientRef` e caminho;
- informar esperado, recebido e regra violada;
- produzir plano de persistência;
- aplicar transação atômica;
- gerar mapa de referências;
- criar snapshot temporário quando solicitado;
- promover sem duplicação;
- preservar identidade, versões e histórico;
- impedir persistência parcial acidental.

## 20. Requisitos para o Codex

A implementação deve possuir:

1. schemas discriminados por `nodeType`;
2. schemas discriminados por componente opcional;
3. validador agregado que não pare no primeiro erro;
4. resolução de `clientRef`;
5. pesquisa de reutilização e duplicação semântica;
6. grafo de dependências entre nós;
7. ordenação topológica da persistência;
8. transação única para `ALL_OR_NOTHING`;
9. UUIDs técnicos e códigos versionados;
10. mapa de referências no retorno;
11. modo `dryRun` e `VALIDATE_ONLY`;
12. armazenamento temporário versionado;
13. promoção temporária para persistente;
14. perfis de prontidão;
15. erros por domínio, nó e caminho;
16. idempotência baseada em chave e hash normalizado;
17. testes de rollback;
18. testes de reutilização de conteúdo existente;
19. testes com componentes opcionais ausentes;
20. exemplos completos de sucesso e rejeição.

## 21. Pendências

- definir duração e limpeza de materializações temporárias;
- definir quando UUID temporário é preservado na promoção;
- definir limite de nós, relações e tamanho do payload;
- definir paginação ou upload para pacotes muito grandes;
- definir política de validação parcial por grupos independentes;
- definir catálogo inicial de `nodeType` e `relationType`;
- definir códigos de erro detalhados por domínio;
- definir assinatura ou hash do pacote validado;
- testar pacotes de ator, grupo, missão, local e facção;
- testar correção automática do GPT após múltiplos erros.
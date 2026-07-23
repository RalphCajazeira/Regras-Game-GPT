# Integração entre GPT, Backend e Frontend

**Versão da proposta:** `integration-v0.9`  
**Status:** em validação

## 1. Objetivo

Permitir que o GPT conduza interações, crie e revise conteúdos e resolva sessões localmente, enquanto o backend mantém autoridade sobre dados persistidos, regras versionadas, validações e consequências permanentes.

A integração deve:

- funcionar dentro do limite de 30 Actions;
- aceitar criação e validação em lote;
- reutilizar conteúdo existente;
- criar conteúdo faltante quando autorizado;
- materializar conteúdo temporário;
- persistir pacotes completos atomicamente;
- devolver erros completos e corrigíveis;
- impedir estados parciais ou incoerentes.

## 2. Regra central

```text
Backend = autoridade de dados, cálculo, validação e persistência.
GPT = condutor, criador, revisor e resolvedor dentro das regras carregadas.
Frontend = visualização e administração usando o mesmo domínio.
```

Conteúdos não são imutáveis. O GPT pode propor correções, balanceamentos, evoluções e migrações. O backend valida antes de persistir.

Depois que uma sessão começa, o GPT não acrescenta retroativamente recursos ausentes do snapshot.

## 3. Action Gateway

```text
GPT
↓ REST + OpenAPI
Action Gateway
↓
Serviços de aplicação e domínio
↓
Persistência, sessões temporárias e integrações
```

O gateway expõe no máximo 30 `operationId`s. Microoperações usam enums tipados dentro de Actions de domínio.

## 4. Responsabilidades do GPT

- interpretar a intenção do jogador;
- pesquisar definições, variantes e conteúdos semelhantes;
- reutilizar códigos, IDs e versões existentes;
- montar pacotes com nós, relações e `clientRef`;
- indicar se deseja apenas validar, materializar temporariamente ou persistir;
- enviar somente componentes aplicáveis;
- fornecer definições completas quando criar conteúdo novo;
- ler todos os `issues` e `warnings`;
- corrigir propostas conforme regras e erros acionáveis;
- manter estado local de sessões;
- usar apenas recursos presentes no snapshot;
- enviar resultados consolidados e logs estruturados;
- solicitar promoção quando um conteúdo temporário se tornar relevante.

## 5. Responsabilidades do backend

- persistir definições, variantes, instâncias, atores, itens e vínculos;
- manter materializações temporárias versionadas;
- localizar conteúdo equivalente e impedir duplicação semântica;
- calcular atributos, orçamentos, preços, qualidade e pontuações;
- validar componentes opcionais somente quando aplicáveis ou enviados;
- validar todos os problemas detectáveis na mesma resposta;
- resolver `clientRef`, códigos, versões e UUIDs;
- construir grafo de dependências;
- produzir plano de persistência;
- aplicar transações atômicas e rollback;
- gerar snapshots completos e deltas;
- controlar `stateVersion`, idempotência e versões de regras;
- retornar `referenceMap`, erros acionáveis e próximas operações;
- preservar IDs, vínculos, origem, versões e histórico.

## 6. Versões de regras

```json
{
  "ruleVersions": {
    "attributes": "attributes-v0.4",
    "combat": "combat-v0.2",
    "equipment": "equipment-v0.5",
    "actions": "actions-v0.1",
    "economy": "economy-v0.3",
    "skills": "skills-v0.1",
    "crafting": "crafting-v0.2",
    "actors": "actors-v0.1",
    "inventory": "inventory-v0.2",
    "operations": "operations-v0.2",
    "bundles": "bundles-v0.1",
    "integration": "integration-v0.9"
  }
}
```

Toda resolução, materialização, criação, revisão ou persistência informa as versões utilizadas.

## 7. Modos de criação

```text
VALIDATE_ONLY
MATERIALIZE_TEMPORARY
PERSIST_IF_VALID
PERSIST_REQUIRED
PROMOTE_TEMPORARY
```

### Validar

O backend resolve referências, calcula valores, valida regras e devolve problemas sem criar estado utilizável.

### Materializar temporariamente

O backend cria um snapshot utilizável em combate, Avaliação ou outra interação sem cadastrar tudo como conteúdo canônico.

### Persistir

O backend valida o pacote inteiro e, somente quando aprovado, cria e vincula tudo em uma transação.

### Promover

O backend transforma uma materialização temporária relevante em conteúdo persistente, preservando identidade e histórico quando possível.

## 8. Pacote completo

Um pacote pode conter:

```text
ator
atributos
recursos
perícias
proficiências
habilidades
magias
passivas
equipamentos
itens
inventário
dinheiro
drops
comportamento
relações
vínculos
```

Nem todos os componentes são obrigatórios para todos os atores.

Exemplos válidos:

- animal sem equipamento ou dinheiro;
- espírito sem inventário físico;
- NPC sem magia;
- comerciante sem perfil de combate;
- monstro com ações e drops, mas sem perícias sociais.

Se um componente estiver presente, deve passar integralmente pelas regras do domínio correspondente.

## 9. Reutilização e criação

Cada nó informa uma estratégia:

```text
REUSE_REQUIRED
REUSE_OR_CREATE
CREATE_NEW
CREATE_VARIANT
INLINE_TEMPORARY
UPDATE_EXISTING
REVIEW_EXISTING
```

O backend pode receber no mesmo pacote:

- referências a conteúdos existentes;
- novas definições;
- novas instâncias;
- relações entre ambos.

A resposta informa por `clientRef`:

```text
CREATED
REUSED
UPDATED
TEMPORARY
REVIEW_REQUIRED
```

## 10. Validação acumulativa

A validação ocorre por camadas:

```text
schema
→ referências
→ aplicabilidade
→ orçamentos
→ valores derivados
→ regras de domínio
→ vínculos cruzados
→ perfis de prontidão
→ plano de persistência
```

O backend deve devolver todos os erros independentes já detectáveis.

Exemplo:

```json
{
  "status": "REQUIRES_REVIEW",
  "issues": [
    {
      "clientRef": "actor:bandit-leader",
      "code": "PRIMARY_ATTRIBUTE_BUDGET_MISMATCH",
      "path": "/nodes/0/data/primaryAttributes",
      "expected": { "totalPrimaryPoints": 81 },
      "received": { "totalPrimaryPoints": 79 }
    },
    {
      "clientRef": "action:flame-slash",
      "code": "ACTION_COST_MISSING",
      "path": "/nodes/4/data/resourceCosts"
    }
  ]
}
```

## 11. Identidade

```text
UUID → identidade técnica
code → definição legível
version → versão utilizada
clientRef → referência local no pacote
sequence → ordenação ou número amigável
```

Relações persistentes usam UUID e chaves estrangeiras. O backend devolve um `referenceMap` convertendo cada `clientRef`.

## 12. Atomicidade

Criação de ator completo, grupo, materialização de combate e promoção usam por padrão:

```text
ALL_OR_NOTHING
```

Fluxo:

```text
validar tudo
→ construir plano
→ abrir transação
→ criar e vincular
→ validar estado final
→ commit
```

Com erro bloqueante:

```text
rollback total
```

## 13. Materialização de atores

```text
avistamento mínimo
→ interação provável
→ GPT monta ou referencia pacote
→ backend valida
→ reutiliza definições
→ cria conteúdo temporário faltante
→ retorna snapshot e prontidão
```

A materialização pode ser anônima e temporária.

Se o ator se tornar relevante:

```text
materialização temporária
→ PROMOTE_TEMPORARY
→ persistência sem duplicação
```

## 14. Perfis de prontidão

```text
ACTOR_COMPLETE
COMBAT_READY
TRADE_READY
CRAFT_READY
EVALUATION_READY
COMPANION_READY
PERSISTENCE_READY
```

Estados:

```text
READY
INCOMPLETE
INVALID
REQUIRES_MATERIALIZATION
REQUIRES_REVIEW
```

Um ator pode estar completo para diálogo e não estar pronto para combate.

`COMBAT_READY` exige, quando aplicável:

- atributos e recursos válidos;
- ações completas;
- custos declarados;
- armas e perfis de ataque válidos;
- equipamentos em slots permitidos;
- munição, consumíveis e cargas existentes;
- inventário materializado;
- facção e controle definidos.

## 15. Atores e componentes opcionais

O backend distingue:

```text
OPTIONAL_NOT_PROVIDED
NOT_APPLICABLE
REQUIRED_MISSING
PROVIDED_INVALID
```

Não deve exigir magia, equipamento, perícia ou inventário de atores aos quais esses componentes não se aplicam.

Também não deve omitir validação quando o GPT enviar esses componentes.

## 16. Identidade de item no snapshot

Todo item materializado informa:

```text
definitionId
definitionCode
definitionVersion
familyCode
variantCode
itemInstanceId
quality
resolvedPowerBudget
resolvedModifiers
resolvedBaseBuyPrice
resolvedBaseSellPrice
condition
durability
ruleVersions
```

Qualidade não cria definição duplicada.

## 17. Combate

O snapshot contém:

- participantes materializados;
- atributos e recursos;
- posições e linha do tempo;
- ações disponíveis;
- equipamentos e instâncias relevantes;
- munições, consumíveis, cargas e durabilidade;
- condições e recargas;
- política de rolagens.

O backend valida existência, consumo, alcance, tempo, rolagens, dano, condições e resultado.

Item consumido ou destruído não reaparece no saque.

## 18. Inventário e saque

O backend valida:

- existência e quantidade;
- propriedade e localização;
- compatibilidade de pilha;
- peso e capacidade;
- slots e requisitos;
- vínculos e reservas;
- ausência de duplicação.

Pacotes de ator podem criar e vincular inventário e equipamentos na mesma transação.

## 19. Comércio

O snapshot comercial contém:

- instâncias e pilhas;
- qualidade e condição;
- preços resolvidos;
- saldos;
- faixas de negociação;
- perícias, passivas e relação;
- reservas.

A transação altera propriedade e saldo atomicamente.

## 20. Fabricação

O snapshot contém:

- receita;
- definição e variante de saída;
- materiais e ferramentas reservados;
- perícia e profissão;
- chance de sucesso;
- limiares e teto de qualidade;
- perfil de distribuição;
- versões.

Cada sucesso cria uma instância, não uma definição por qualidade.

## 21. Criação e revisão de conteúdo

Antes de criar:

```text
searchContent
→ localizar equivalentes
→ manageContent VALIDATE_CONTENT_BUNDLE
→ corrigir problemas
→ PERSIST_CONTENT_BUNDLE ou MATERIALIZE_TEMPORARY
```

Revisões mantêm ID e código quando possível, criam nova versão, preservam vínculos e não recalculam eventos históricos.

## 22. Erros e recuperação

Cada erro informa:

```text
code
severity
blocking
clientRef
domain
path
ruleCode
message
expected
received
recoveryActions
```

O backend deve sugerir a próxima Action e operação quando possível.

## 23. Idempotência e concorrência

Toda operação crítica usa:

```text
requestId
idempotencyKey
baseStateVersion
```

Repetição idêntica devolve o mesmo resultado. Conteúdo diferente com a mesma chave retorna `IDEMPOTENCY_PAYLOAD_MISMATCH`.

Conflito de versão retorna `CONFLICT` com recarga, reaplicação segura ou cancelamento.

## 24. Operações conceituais

```text
loadGame
searchContent
getContent
manageContent
materializeActors
manageActor
manageRelationships
manageInventory
startOrLoadEncounter
checkpointEncounter
submitEncounterResolution
startOrLoadTrade
submitTrade
startOrLoadCraft
submitCraftResolution
startOrLoadSkillSession
submitSkillResolution
manageMission
applyProgression
advanceWorldTime
manageWorldState
cancelSession
```

## 25. Requisitos para o Codex

- motor único de validação para pacote temporário e persistente;
- validação acumulativa;
- schemas discriminados por nó e componente;
- UUID, `code`, `version` e `clientRef`;
- pesquisa e reutilização de conteúdo;
- grafo de dependências;
- transação e rollback;
- materialização temporária;
- promoção sem duplicação;
- perfis de prontidão;
- `referenceMap`;
- idempotência e controle de versão;
- erros acionáveis;
- testes com componentes opcionais;
- testes com vários erros simultâneos;
- testes de pacote misto com conteúdo reutilizado e novo;
- teste do limite máximo de 30 Actions.
# Decisões e Pendências

Este arquivo separa regras aceitas de propostas ainda sujeitas a testes.

## 1. Arquitetura

- O GPT conduz interações usando snapshots versionados.
- O GPT pode criar, corrigir, balancear e evoluir conteúdos por Actions válidas.
- O backend calcula, valida, persiste e aplica consequências autoritativas.
- Validação não significa imutabilidade.
- Recursos ausentes não podem ser acrescentados retroativamente a uma sessão em andamento.
- Regras ficam nos serviços de aplicação e domínio, não presas a REST, GraphQL ou frontend.
- O frontend futuro reutiliza os mesmos serviços, regras e validações.

## 2. Actions e contratos operacionais

- O GPT possui limite de 30 Actions expostas.
- O catálogo inicial planeja 22 Actions e reserva 8 vagas.
- REST + OpenAPI é a interface canônica do GPT.
- GraphQL pode ser avaliado para o frontend, mas não substitui as Actions.
- Uma Action representa domínio ou fluxo; microoperações usam enums tipados.
- Serviços e endpoints internos não contam como Actions.
- Novos sistemas reutilizam Actions existentes sempre que compartilharem invariantes.
- Toda mutação crítica usa idempotência e `stateVersion`.
- O OpenAPI deve falhar automaticamente acima de 30 `operationId`s.
- Não expor mutações arbitrárias ou payload sem schema.

Catálogo inicial:

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

## 3. Criação, validação e materialização em lote

### Decisão consolidada

```text
GPT monta pacote completo em memória
→ backend valida todos os nós e relações
→ reutiliza conteúdo existente
→ cria somente o que falta e está autorizado
→ materializa temporariamente ou persiste
→ devolve snapshot, IDs e mapa de referências
```

- O pacote pode conter atores, atributos, ações, habilidades, magias, perícias, passivas, equipamentos, itens, drops, inventário e vínculos.
- Nem todo ator precisa possuir todos os componentes.
- Ausência de componente não aplicável é válida.
- Componente enviado deve ser validado integralmente.
- A validação deve acumular todos os problemas independentes detectáveis.
- O backend não deve parar no primeiro erro.
- Cada erro deve indicar `clientRef`, domínio, caminho, regra, esperado, recebido e recuperação.
- Criação completa usa `ALL_OR_NOTHING` por padrão.
- Com erro bloqueante, nada é persistido.
- Pacotes podem ser apenas validados, materializados temporariamente, persistidos ou promovidos.
- Conteúdo temporário pode participar de combate e depois ser descartado ou promovido.
- Pacotes mistos podem reutilizar conteúdos existentes e criar conteúdos faltantes na mesma transação.

Modos:

```text
VALIDATE_ONLY
MATERIALIZE_TEMPORARY
PERSIST_IF_VALID
PERSIST_REQUIRED
PROMOTE_TEMPORARY
```

Estratégias por nó:

```text
REUSE_REQUIRED
REUSE_OR_CREATE
CREATE_NEW
CREATE_VARIANT
INLINE_TEMPORARY
UPDATE_EXISTING
REVIEW_EXISTING
```

## 4. Identificadores

- UUID é a identidade técnica principal de definições, instâncias, atores, sessões e vínculos.
- Definições reutilizáveis também possuem `code` e `version`.
- O GPT usa `clientRef` para relacionar nós antes de receber UUIDs.
- O backend devolve `referenceMap` após validar ou persistir.
- Sequências numéricas servem para ordem, versão e números amigáveis, não como identidade principal.
- Relações persistentes usam UUID e chaves estrangeiras.

## 5. Componentes opcionais de ator

Podem existir quando aplicáveis:

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

Exemplos válidos:

- animal sem equipamento;
- espírito sem inventário físico;
- NPC sem magia;
- comerciante sem perfil de combate;
- monstro sem perícias sociais.

## 6. Prontidão

Perfis:

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

## 7. Criação, revisão e versionamento

- Todo conteúdo pode ser revisado com justificativa coerente.
- Sugestões do jogador são avaliadas, não aplicadas automaticamente.
- Revisões preferem manter ID e código e criar nova versão.
- Toda revisão declara escopo.
- Eventos históricos não são recalculados automaticamente.
- O backend retorna erros acionáveis.

Escopos:

```text
DEFINITION_FUTURE_INSTANCES
CURRENT_INSTANCE
SELECTED_INSTANCES
ALL_COMPATIBLE_INSTANCES
```

## 8. Atributos

```text
Força
Agilidade
Destreza
Vitalidade
Inteligência
```

- Agilidade representa rapidez e mobilidade.
- Destreza representa precisão e controle.
- Cada nível concede 10 pontos primários.
- Pontos podem permanecer não distribuídos.
- O backend calcula o orçamento esperado por nível.
- Erro de distribuição deve informar total esperado, recebido e diferença.
- Especializações usam secundários, perícias, proficiências, profissões e passivas.

## 9. Atores

- Ator é o modelo universal para pessoas, animais, monstros, criaturas, espíritos, construtos, invocações e jogadores.
- Natureza, espécie, arquétipo, facção, papel e controle são separados.
- Entidades irrelevantes podem existir como avistamentos.
- Antes de interação mecânica, o ator é validado e materializado.
- Materialização pode ser temporária.
- Dados da sessão ficam congelados no snapshot.
- O GPT não inventa retroativamente ações, itens, dinheiro ou drops.
- Instâncias temporárias podem ser promovidas sem duplicação.
- Materialização usa semente determinística.

## 10. Itens, variantes, qualidades e instâncias

```text
Definição única com perfil Comum
→ variante somente quando houver diferença real
→ qualidade na instância
→ valores finais resolvidos e congelados
```

- Qualidade não cria novo cadastro.
- Cada unidade criada, encontrada ou comprada é uma instância.
- A instância guarda qualidade, orçamento, modificadores e preços resolvidos.
- Mudanças futuras não alteram silenciosamente instâncias existentes.
- Migração é explícita.

## 11. Equipamentos

- Todo equipamento declara modificadores, inclusive zeros.
- Nível e slot formam orçamento Comum de referência.
- Qualidade multiplica o orçamento da instância.
- Distribuição adicional segue o perfil da definição.
- Equipamentos podem conceder ações, passivas e proficiências.
- Qualidade não altera automaticamente peso, slots, material ou empunhadura.

## 12. Inventário, pilhas e saque

- Todo item possui localização e propriedade.
- A mesma unidade não pode estar em dois locais ou operações.
- Pilhas exigem compatibilidade completa.
- Consumíveis, munições e cargas são removidos conforme a ação.
- Itens consumidos não reaparecem no saque.
- Itens carregados, drops naturais e recompensas externas são fontes distintas.
- Pacotes de ator podem criar e vincular inventário na mesma transação.

## 13. Fabricação

- Toda fabricação usa receita conhecida.
- A receita referencia definição e variante de saída.
- Cada sucesso cria uma instância.
- Perícia melhora sucesso e qualidade dentro do teto.
- O backend resolve qualidade, orçamento, modificadores e preço.
- Consumo, criação e XP são autoritativos.
- Valores ficam congelados.

## 14. Economia

- A moeda é Coroa, código `CROWN`.
- A definição guarda preço Comum.
- A instância guarda preço-base resolvido.
- Mercado e negociação definem preço final.
- O backend controla saldo, estoque, propriedade e atomicidade.

## 15. Combate e ações

- Posições e alcances usam metros.
- O tempo usa linha contínua em ticks.
- Existe uma única rolagem de acerto.
- Ações possuem preparação, recuperação e tempo mínimo.
- Munições, consumíveis e equipamentos precisam existir no snapshot.
- Combate só inicia com participantes `COMBAT_READY`.

## 16. Perícias, profissões e passivas

- Atributos representam potencial.
- Perícias representam treinamento.
- Proficiências representam domínio específico.
- Profissões agrupam perícias, receitas, passivas e desbloqueios.
- O backend calcula pontuação efetiva, XP e níveis.

## 17. Contratos comuns

Solicitação:

```text
operation
requestId
idempotencyKey
baseStateVersion
dryRun
context
payload tipado
```

Resposta:

```text
status
previousStateVersion
newStateVersion
result
snapshot
snapshotDelta
referenceMap
warnings
issues
recoveryActions
availableNextOperations
```

## 18. Pendências de lotes e integração

- definir limite de nós, relações e tamanho do pacote;
- definir duração de materializações temporárias;
- definir preservação de UUID na promoção;
- definir catálogo final de `nodeType` e `relationType`;
- definir política de atomicidade parcial;
- definir retenção de idempotência;
- definir autenticação e autorização;
- definir formato final de `snapshotDelta`;
- criar exemplos OpenAPI completos;
- testar correção automática após múltiplos erros;
- testar rollback total;
- testar pacote misto com conteúdo reutilizado e novo.

## 19. Pendências de atributos

- validar 16 pontos livres com cinco atributos;
- definir limite por atributo;
- testar níveis 1, 5, 10, 20 e 50;
- calibrar secundários, Vida, Mana e Vigor.

## 20. Pendências de atores

- orçamento por nível e ameaça;
- espécies, arquétipos e variantes;
- nível de desafio;
- grupos;
- moral, fuga e rendição;
- domesticação e recrutamento;
- relações, impressões e memórias.

## 21. Pendências de equipamentos e inventário

- multiplicadores de qualidade e slot;
- arredondamento e custos;
- perfis de distribuição;
- durabilidade e reparo;
- limites de pilha e sobrecarga;
- divisão de saque;
- identificação de itens;
- expiração de cadáveres.

## 22. Pendências de fabricação e economia

- fórmula de sucesso;
- limiares de qualidade;
- política de falha;
- fabricação em lote;
- XP e retorno decrescente;
- cesta econômica;
- margens e negociação;
- mercados regionais;
- inflação.

## 23. Pendências de combate e progressão

- chance de acerto e influência do nível;
- Defesa Crítica e mitigação;
- fila de eventos;
- tempos de ações e movimento;
- cobertura, projéteis e reações;
- curva de XP;
- divisão de XP em grupo;
- nível máximo;
- recompensas de missão.

## 24. Próximas validações

1. Enviar NPC nível 5 com orçamento incorreto e verificar erro completo.
2. Enviar NPC sem magia e confirmar ausência válida.
3. Enviar NPC com magia incompleta e confirmar rejeição localizada.
4. Validar pacote com três erros independentes na mesma resposta.
5. Materializar grupo temporário e iniciar combate.
6. Promover um sobrevivente sem duplicar identidade.
7. Persistir pacote completo com conteúdos reutilizados e novos.
8. Forçar falha no último vínculo e confirmar rollback total.
9. Repetir a mesma `idempotencyKey` e impedir duplicação.
10. Garantir OpenAPI com no máximo 30 Actions.
# Decisões e Pendências

Este arquivo separa regras aceitas de propostas ainda sujeitas a testes.

## 1. Arquitetura

- O GPT conduz interações usando snapshots versionados.
- O GPT pode criar, corrigir, balancear e evoluir conteúdos por Actions válidas.
- O backend calcula, valida, persiste e aplica consequências autoritativas.
- Validação não significa imutabilidade.
- O frontend futuro usa o mesmo domínio e validações.
- Recursos ausentes não podem ser acrescentados retroativamente a uma sessão em andamento.
- As regras de negócio ficam em serviços de aplicação e domínio, não presas à interface REST, GraphQL ou frontend.

## 2. Actions e contratos operacionais

- O GPT possui limite de 30 Actions expostas.
- O catálogo inicial planeja 22 Actions e reserva 8 vagas.
- REST + OpenAPI é a interface canônica do GPT.
- GraphQL pode ser avaliado futuramente para o frontend, mas não substitui as Actions do GPT.
- Uma Action representa um domínio ou fluxo; microações usam enums internos tipados.
- Serviços, comandos e endpoints internos não contam como Actions.
- Novos sistemas devem reutilizar Actions existentes sempre que compartilharem domínio e invariantes.
- Nova Action exige ciclo de vida, transação, segurança ou contrato realmente diferente.
- Não expor `executeAnything`, `executeGraphQL`, atualização arbitrária de entidade ou payload sem schema.
- Toda mutação crítica usa idempotência e `stateVersion`.
- Toda resposta informa erros acionáveis, correções e próximas operações possíveis.
- O OpenAPI deve possuir teste automatizado que falhe acima de 30 `operationId`s.
- O alvo de até 22 Actions só pode ser alterado por decisão versionada.

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

## 3. Criação, revisão e versionamento

- Todo conteúdo pode ser revisado com justificativa coerente.
- Sugestões do jogador são avaliadas, não aplicadas automaticamente.
- O GPT consulta identidade, orçamento, vínculos e impacto.
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

## 4. Atributos

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
- Especializações usam secundários, perícias, proficiências, profissões e passivas.

## 5. Atores

- Ator é o modelo universal para pessoas, animais, monstros, criaturas, espíritos, construtos, invocações e jogadores.
- Natureza, espécie, arquétipo, facção, papel e controle são separados.
- Entidades irrelevantes podem existir como avistamentos.
- Antes de interação mecânica relevante, o ator é materializado com ficha completa.
- Dados materializados ficam congelados no snapshot.
- O GPT não inventa retroativamente ações, itens, dinheiro ou drops.
- Instâncias podem ser promovidas sem duplicação.
- Companheiro é papel e vínculo, não espécie.
- Materialização usa semente determinística.

## 6. Definições, variantes, qualidades e instâncias de item

### Regra consolidada

```text
Definição única com perfil Comum
→ variante somente quando houver diferença real
→ qualidade na instância
→ valores finais resolvidos e congelados
```

- Qualidade não cria novo cadastro.
- `elven-dagger` representa a Adaga Élfica em qualquer qualidade.
- A definição usa `COMMON` como referência de poder e preço.
- Cada unidade criada, encontrada ou comprada é uma instância.
- A instância guarda qualidade, orçamento, modificadores e preços resolvidos.
- Mudanças futuras nas regras não alteram silenciosamente instâncias existentes.
- Migração é explícita.

Não cria nova definição:

- qualidade diferente;
- condição ou durabilidade;
- fabricante ou proprietário;
- estado de roubo;
- nome personalizado sem efeito mecânico.

Pode criar variante ou definição:

- material estrutural diferente;
- elemento diferente;
- distribuição de poder diferente;
- ações ou passivas diferentes;
- receita ou função mecânica própria.

Antes de criar conteúdo, o backend deve retornar:

```text
USE_EXISTING_DEFINITION
USE_EXISTING_VARIANT
CREATE_NEW_VARIANT
CREATE_NEW_DEFINITION
REVIEW_REQUIRED
```

Mudança somente de qualidade sempre reutiliza definição ou variante existente.

## 7. Equipamentos

- Todo equipamento declara todos os modificadores, inclusive zeros.
- `primaryModifiers` inclui cinco atributos.
- Corpo e Peito são slots diferentes.
- Armas podem ocupar uma ou duas mãos.
- Qualidades: Inferior, Comum, Raro, Épico e Lendário.
- Nível e slot formam o orçamento Comum de referência.
- Qualidade multiplica o orçamento da instância.
- Distribuição adicional segue perfil da definição.
- Penalidades recuperam orçamento positivo dentro de limites.
- Equipamentos podem conceder ações, passivas e proficiências.
- Qualidade não altera automaticamente peso, slots, material ou empunhadura.
- Passivas só escalam quando houver perfil explícito.

## 8. Inventário, pilhas e saque

- Definição, variante, instância e pilha são diferentes.
- Todo item persistente possui localização e propriedade.
- A mesma unidade não pode estar em dois locais ou operações.
- Pilhas exigem compatibilidade de definição, variante, qualidade, condição e demais estados.
- Equipamentos com histórico ou durabilidade individual normalmente permanecem separados.
- O frontend pode agrupar visualmente instâncias iguais.
- Consumíveis, munições e cargas são removidos conforme a ação.
- Itens consumidos não reaparecem no saque.
- Itens carregados, drops naturais e recompensas externas são fontes distintas.
- Drops aleatórios usam semente persistida.
- Operações do domínio são agrupadas em `manageInventory`.

## 9. Fabricação

- Toda fabricação usa receita conhecida.
- A receita referencia definição e variante de saída.
- Cada sucesso cria uma nova instância.
- Cinco tentativas podem gerar várias instâncias de qualidades diferentes sem duplicar a definição.
- Receita, material, ferramenta, oficina e desbloqueios limitam a qualidade máxima.
- Perícia melhora sucesso e qualidade dentro do teto.
- O backend resolve qualidade, orçamento, modificadores e preço-base da instância.
- Consumo, criação, reparo, qualidade e XP são autoritativos.
- Valores resolvidos ficam congelados.

## 10. Economia

- A moeda é Coroa, código `CROWN`.
- Valores são inteiros.
- A definição guarda preço Comum de referência.
- A instância guarda preço-base resolvido por qualidade.
- Mercado e negociação definem o preço final.
- Passivas comerciais não alteram preços-base.
- O GPT negocia somente dentro das faixas.
- O backend controla saldo, estoque, propriedade e atomicidade.

## 11. Combate e ações

- Posições e alcances usam metros.
- O tempo usa linha contínua em ticks.
- Existe uma única rolagem de acerto.
- Precisão é comparada com Esquiva ou Resistência.
- Chance de acerto fica entre 1% e 99%.
- Após acerto entram crítico, Defesa Crítica e mitigação.
- Ações possuem preparação, recuperação e tempo mínimo.
- Movimento e ataque são separados, salvo ação composta.
- Ações podem ser interrompidas.
- Munições, consumíveis e equipamentos usados precisam existir no snapshot.

## 12. Perícias, profissões e passivas

- Atributos representam potencial geral.
- Perícias representam treinamento.
- Proficiências representam domínio específico.
- Profissões agrupam perícias, receitas, passivas e desbloqueios.
- Passivas alteram regras por efeitos estruturados.
- O backend calcula pontuação efetiva, XP e níveis.
- Atividades triviais concedem XP reduzido ou zero.

## 13. Contratos comuns de operação

Toda mutação persistente deve usar, quando aplicável:

```text
operation
requestId
idempotencyKey
baseStateVersion
dryRun
context
payload tipado
```

Toda resposta deve poder informar:

```text
status
previousStateVersion
newStateVersion
result
snapshotDelta
warnings
issues
recoveryActions
availableNextOperations
```

Status canônicos:

```text
SUCCESS
PARTIAL_SUCCESS
REQUIRES_ACTION
REQUIRES_MATERIALIZATION
REQUIRES_REVIEW
CONFLICT
REJECTED
CANCELLED
```

Antes de sessão complexa:

```text
READY
INCOMPLETE
INVALID
REQUIRES_MATERIALIZATION
REQUIRES_REVIEW
```

## 14. Pendências de Actions e integração

- validar o limite real no editor e no OpenAPI final;
- testar catálogo com até 22 Actions;
- revisar orçamento após cada novo sistema;
- definir autenticação e autorização;
- definir payload máximo, paginação e timeout;
- definir retenção de idempotência;
- definir atomicidade de lotes;
- definir formato final de `snapshotDelta`;
- definir códigos de erro por domínio;
- criar exemplos OpenAPI completos;
- testar escolha de Action pelo GPT;
- testar recuperação automática após erros;
- medir tokens e tamanho de snapshots;
- impedir que Actions internas sejam expostas acidentalmente.

## 15. Pendências de atributos

- validar 16 pontos livres com cinco atributos;
- definir limite por atributo;
- testar níveis 1, 5, 10, 20 e 50;
- calibrar secundários, Vida, Mana e Vigor;
- validar Agilidade contra Destreza.

## 16. Pendências de atores

- orçamento de criação por nível e ameaça;
- espécies, arquétipos e variantes;
- nível de desafio;
- grupos;
- moral, fuga e rendição;
- gatilhos de materialização e promoção;
- domesticação e recrutamento;
- relações, impressões e memórias;
- migração após revisão;
- validação temática por tags.

## 17. Pendências de equipamentos e qualidade

- calibrar multiplicadores de qualidade e slot;
- definir arredondamento;
- calibrar custos de modificadores;
- limite de recuperação negativa;
- perfis de distribuição por categoria;
- orçamento de ações e passivas;
- escalonamento de efeitos;
- equipamentos versáteis;
- migração de instâncias antigas;
- agrupamento visual.

## 18. Pendências de inventário e saque

- limites de pilha;
- compatibilidade de condição;
- sobrecarga;
- durabilidade e reparo;
- contêineres aninhados;
- tempo de coleta;
- divisão de saque em grupo;
- identificação de itens;
- expiração de cadáveres;
- prevenção de duplicação concorrente.

## 19. Pendências de fabricação

- fórmula de sucesso;
- limiares de qualidade;
- perfis de distribuição;
- política de falha e consumo;
- tempo de produção;
- reparo e conservação;
- encantamento;
- desmontagem e subprodutos;
- fabricação em lote;
- XP e prevenção de treinamento abusivo.

## 20. Pendências de economia

- cesta econômica;
- preços Comuns por categoria;
- multiplicadores de qualidade;
- taxa de recompra;
- margens e negociação;
- mercados regionais;
- itens roubados e vinculados;
- condição e durabilidade;
- inflação;
- migração de preços antigos.

## 21. Pendências de combate e ações

- base de 75% da chance de acerto;
- influência do nível;
- Defesa Crítica e mitigação;
- prioridades da fila;
- tempos de movimento, ataques e magias;
- engajamento, reações, cobertura e projéteis;
- testes de perseguição, interrupção e múltiplos inimigos;
- aleatoriedade auditável.

## 22. Pendências de progressão

- XP do personagem;
- divisão em grupo;
- diferença de nível;
- recompensas de missões e objetivos;
- nível máximo;
- relação entre personagem, perícia e profissão.

## 23. Próximas validações

1. Gerar OpenAPI com as 22 Actions planejadas.
2. Garantir teste automático com máximo de 30 `operationId`s.
3. Testar o GPT escolhendo Action e enum corretos em cenários ambíguos.
4. Testar erro acionável e recuperação automática.
5. Repetir mutação com a mesma `idempotencyKey` e impedir duplicação.
6. Simular conflito de `stateVersion` e recuperação.
7. Criar uma definição de Adaga Élfica Comum de referência.
8. Simular cinco fabricações com falhas e qualidades diferentes.
9. Confirmar uma definição e várias instâncias.
10. Continuar testes de atores, combate, comércio e fabricação.
11. Ajustar antes das versões `v1.0`.
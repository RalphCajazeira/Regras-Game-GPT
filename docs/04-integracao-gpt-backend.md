# Integração entre GPT, Backend e Frontend

**Versão da proposta:** `integration-v0.5`  
**Status:** em validação

## 1. Objetivo

Permitir que o GPT conduza interações com fluidez, crie e revise conteúdos quando necessário, enquanto o backend mantém autoridade sobre dados persistidos, regras versionadas, validações e consequências permanentes.

O padrão se aplica a atores, combate, comércio, perícias, profissões, fabricação e futuras sessões complexas.

## 2. Regra central

```text
Backend = autoridade de dados, regras, validação e persistência.
GPT = condutor, criador, revisor e resolvedor local dentro das regras carregadas.
Frontend = visualização e administração usando o mesmo domínio.
```

A autoridade do backend não torna os conteúdos imutáveis. O GPT pode propor, corrigir, balancear e evoluir qualquer conteúdo por meio das operações adequadas. O backend valida a proposta antes de persistir.

## 3. Responsabilidades

### GPT

- interpretar a intenção do jogador;
- carregar fichas, ações, perícias, receitas, estoques e conteúdos persistidos;
- usar regras e parâmetros versionados;
- manter estado local da sessão;
- calcular chances, danos, tempos, posições, preços e resultados permitidos;
- registrar rolagens e decisões;
- identificar quando uma entidade precisa ser materializada;
- propor definições novas quando não existir conteúdo compatível;
- revisar conteúdos incoerentes, incompletos ou desbalanceados;
- avaliar sugestões do jogador em vez de aplicá-las automaticamente;
- não inventar atributos, perícias, passivas, receitas, itens, custos ou recompensas durante uma resolução já iniciada;
- enviar histórico e resultado consolidado quando houver alteração persistente.

### Backend

- persistir atores, itens, ações, perícias, profissões, passivas, receitas, comerciantes e sessões;
- calcular atributos secundários e pontuações efetivas;
- calcular preços-base, faixas comerciais e limites de qualidade;
- fornecer regras, orçamentos e conteúdos necessários para criação e revisão;
- gerar snapshots completos;
- controlar `stateVersion` e versões de regras;
- validar e reproduzir resoluções;
- validar propostas de criação, correção, balanceamento e evolução;
- aplicar consumo, criação, XP, níveis, saldos e propriedade atomicamente;
- preservar IDs, vínculos, versões e histórico;
- retornar erros acionáveis;
- manter auditoria.

### Frontend futuro

- exibir fichas, perícias, profissões, passivas e receitas;
- mostrar equipamentos e ações concedidas;
- acompanhar combate, comércio e fabricação;
- apresentar chances, rolagens, tempos, qualidade e histórico;
- permitir revisão administrativa usando as mesmas operações e validações do GPT;
- apresentar versões, motivos de revisão e escopo de aplicação.

## 4. Padrão de sessão versionada

```text
1. GPT solicita criação ou carregamento.
2. Backend retorna snapshot completo e versionado.
3. GPT conduz escolhas e resoluções localmente.
4. GPT mantém log estruturado.
5. Checkpoints podem ser enviados.
6. GPT envia resultado consolidado.
7. Backend recalcula e valida.
8. Backend persiste o resultado autoritativo.
9. GPT apresenta o retorno final.
```

Campos comuns:

```text
sessionId
stateVersion
ruleVersions
lifecycleStatus
snapshot
resolutionPolicy
```

## 5. Versões de regras

```json
{
  "ruleVersions": {
    "attributes": "attributes-v0.4",
    "combat": "combat-v0.2",
    "equipment": "equipment-v0.4",
    "actions": "actions-v0.1",
    "economy": "economy-v0.1",
    "skills": "skills-v0.1",
    "crafting": "crafting-v0.1",
    "actors": "actors-v0.1",
    "integration": "integration-v0.5"
  }
}
```

Toda resolução ou revisão persistente informa as versões utilizadas.

## 6. Atores e materialização

O GPT não precisa materializar toda entidade observada.

Fluxo:

```text
avistamento mínimo
→ interação torna-se provável
→ GPT solicita materialização
→ backend fornece definição, orçamento e regras
→ GPT/backend criam a instância completa
→ backend valida
→ snapshot congelado é utilizado
```

O snapshot de ator materializado deve conter, quando aplicável:

- identidade, natureza, espécie e arquétipo;
- nível e categoria de ameaça;
- cinco atributos primários e secundários;
- recursos atuais e máximos;
- ações, magias, habilidades e passivas;
- equipamentos, inventário, consumíveis e dinheiro;
- XP, drops e materiais coletáveis;
- facção, comportamento, moral e objetivos;
- vínculos, domesticação, recrutamento e grupo;
- conhecimento disponível ao jogador;
- semente de geração e versões usadas.

Depois da materialização, o GPT não acrescenta retroativamente recursos que não existiam. Revisões estruturais seguem o fluxo de revisão e não alteram silenciosamente uma sessão em andamento.

## 7. Combate

O snapshot de combate contém:

- participantes materializados e relações;
- cinco atributos primários e secundários finais;
- posições em metros;
- ambiente, obstáculos e coberturas;
- `combatTime`, `nextReadyAt` e fila de eventos;
- ações disponíveis e ações em preparação;
- recursos, condições, recargas e engajamentos;
- política de rolagens.

O GPT mantém a linha do tempo localmente e envia declaração, resolução e estado antes/depois de cada evento relevante.

O backend valida alcance, tempos, custos, rolagens, dano, condições e resultado final.

## 8. Comércio

O snapshot comercial contém:

- saldo do jogador e comerciante;
- estoque e quantidades;
- categorias aceitas;
- preços de referência;
- faixas mínima e máxima;
- reputação e relação;
- perícia de Comércio e passivas aplicáveis;
- regras de negociação.

O GPT conduz diálogo e contrapropostas. O backend confirma preços, saldo, estoque, propriedade e transferência atômica.

## 9. Sessão de perícia

```json
{
  "skillSessionId": "skill-session-id",
  "stateVersion": 20,
  "ruleVersions": {
    "attributes": "attributes-v0.4",
    "skills": "skills-v0.1",
    "integration": "integration-v0.5"
  },
  "actor": {},
  "skill": {
    "code": "persuasion",
    "level": 6,
    "effectiveScore": 48
  },
  "challenge": {},
  "availableActions": [],
  "constraints": {},
  "resolutionPolicy": {}
}
```

Usada quando uma interação exige escolhas ou várias etapas, mas não precisa de chamadas a cada decisão.

O log registra:

- perícia selecionada;
- atributos e pesos aplicados;
- dificuldade;
- passivas e modificadores;
- rolagens;
- resultado;
- custos;
- XP reportado.

O backend calcula o XP autoritativo.

## 10. Sessão de fabricação

```json
{
  "craftSessionId": "craft-session-id",
  "stateVersion": 15,
  "ruleVersions": {
    "attributes": "attributes-v0.4",
    "equipment": "equipment-v0.4",
    "skills": "skills-v0.1",
    "crafting": "crafting-v0.1",
    "integration": "integration-v0.5"
  },
  "actor": {},
  "profession": {},
  "skill": {},
  "recipe": {},
  "materials": [],
  "tools": [],
  "workspace": {},
  "resolution": {
    "successChance": 0,
    "qualityThresholds": {},
    "maximumQuality": "COMMON"
  }
}
```

O GPT pode conduzir preparação, escolhas permitidas, rolagens e narrativa.

O backend valida:

1. sessão e versão;
2. receita conhecida;
3. perícia e profissão;
4. materiais, ferramentas e oficina;
5. limites de qualidade;
6. rolagens e resultado;
7. consumo e subprodutos;
8. item criado ou alterado;
9. XP e progressão;
10. atomicidade.

## 11. Criação de conteúdo

O GPT pode criar qualquer conteúdo necessário ao jogo quando não existir definição adequada:

- ator ou arquétipo;
- espécie;
- ação;
- habilidade;
- magia;
- passiva;
- equipamento;
- item;
- receita;
- comerciante;
- comportamento;
- regra de saque ou recrutamento.

Fluxo recomendado:

```text
GPT identifica necessidade
→ consulta conteúdos semelhantes e regras
→ backend retorna orçamento e restrições
→ GPT propõe definição completa
→ backend valida
→ GPT corrige se necessário
→ backend persiste versão aprovada
```

O backend deve informar exatamente por que uma proposta falhou e como pode ser corrigida.

## 12. Revisão de conteúdo

Todo conteúdo pode ser revisado. Existência prévia não é motivo suficiente para rejeição.

### 12.1 Motivos

```text
CORRECTION
BALANCE_REVISION
NARRATIVE_EVOLUTION
INSTANCE_STATE_CHANGE
SCOPE_PROMOTION
MIGRATION
```

### 12.2 Avaliação de sugestões

Quando o jogador pedir uma revisão ou sugerir uma mudança, o GPT deve:

1. consultar a definição e suas versões;
2. identificar o problema ou objetivo real;
3. verificar regras, orçamento e identidade temática;
4. avaliar consequências sobre conteúdos vinculados;
5. aceitar, adaptar ou recusar a sugestão;
6. explicar de forma objetiva quando a proposta não fizer sentido;
7. enviar ao backend uma revisão estruturada.

O GPT não deve obedecer a uma sugestão incoerente apenas porque foi solicitada.

### 12.3 Escopo

Toda revisão declara:

```text
DEFINITION_FUTURE_INSTANCES
CURRENT_INSTANCE
SELECTED_INSTANCES
ALL_COMPATIBLE_INSTANCES
```

### 12.4 Versionamento

Preferência:

- manter o mesmo ID e código;
- criar uma nova versão;
- registrar motivo e justificativa;
- preservar vínculos;
- declarar política para instâncias existentes;
- não recalcular eventos históricos;
- permitir migração explícita quando segura.

### 12.5 Coerência temática

O GPT usa conhecimento, fontes e instruções para avaliar significado. O backend valida tags, compatibilidade, orçamento e schema.

Uma combinação incomum pode ser aceita quando houver justificativa explícita. Uma incoerência acidental deve retornar erro acionável.

## 13. Resolução direta ou por snapshot

### Direta

Adequada para operação curta e altamente autoritativa. O GPT envia a intenção e o backend calcula e persiste imediatamente.

### Snapshot

Adequada quando existem escolhas, várias etapas, risco, preparação ou narrativa relevante. O GPT resolve localmente e envia o histórico final.

A escolha do modo pertence à operação ou política retornada pelo backend.

## 14. Checkpoints

Podem ser usados quando:

- a sessão durar muito;
- existir criação ou consumo importante;
- ocorrer mudança de fase;
- a conversa puder ser interrompida;
- houver consequência persistente relevante;
- uma entidade temporária for promovida;
- uma revisão precisar entrar em vigor antes da continuação.

Um checkpoint valida um trecho e retorna nova `stateVersion`.

## 15. Concorrência

Se:

```text
baseStateVersion != currentStateVersion
```

O backend rejeita a resolução e informa versão esperada, recebida, conflitos e ações de recuperação.

## 16. Aleatoriedade

Na primeira versão, o GPT pode gerar e registrar d100. O backend valida faixa e coerência, mas não prova aleatoriedade.

Modo auditável futuro:

- semente no snapshot;
- sequência determinística;
- pacote de rolagens;
- assinatura do snapshot.

## 17. Operações conceituais

```text
create/load/materialize/promote/reviseActor
create/reviseContentDefinition
reviewContentCoherence
create/load/checkpoint/submitEncounter
create/load/submitTradeSession
create/load/submitSkillSession
create/load/submitCraftSession
migrateContentInstances
cancelSession
```

Os nomes definitivos dependem da API, mas cada operação deve possuir finalidade única e retorno acionável.
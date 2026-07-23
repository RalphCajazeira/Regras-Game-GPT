# Integração entre GPT, Backend e Frontend

**Versão da proposta:** `integration-v0.6`  
**Status:** em validação

## 1. Objetivo

Permitir que o GPT conduza interações com fluidez, crie e revise conteúdos e resolva sessões localmente, enquanto o backend mantém autoridade sobre dados persistidos, regras versionadas, validações e consequências permanentes.

O padrão se aplica a:

- atores e materialização;
- combate;
- inventário e saque;
- comércio;
- perícias e profissões;
- fabricação;
- criação e revisão de conteúdo;
- futuras sessões complexas.

## 2. Regra central

```text
Backend = autoridade de dados, regras, validação e persistência.
GPT = condutor, criador, revisor e resolvedor local dentro das regras carregadas.
Frontend = visualização e administração usando o mesmo domínio.
```

A autoridade do backend não torna conteúdos imutáveis. O GPT pode propor, corrigir, balancear e evoluir qualquer conteúdo pelas operações adequadas. O backend valida a proposta antes de persistir.

Depois que uma sessão começa, o GPT não acrescenta retroativamente recursos que não estavam no snapshot. Mudanças válidas exigem operação de estado, evento materializado, revisão com escopo explícito ou migração segura.

## 3. Responsabilidades

### GPT

- interpretar a intenção do jogador;
- carregar fichas, inventários, ações, perícias, receitas, estoques e conteúdos persistidos;
- usar regras e parâmetros versionados;
- manter estado local da sessão;
- calcular chances, danos, tempos, posições, preços e resultados permitidos;
- registrar rolagens, operações e decisões;
- identificar quando uma entidade ou contêiner precisa ser materializado;
- propor definições novas quando não existir conteúdo compatível;
- revisar conteúdos incoerentes, incompletos ou desbalanceados;
- avaliar sugestões do jogador em vez de aplicá-las automaticamente;
- usar somente itens, quantidades, cargas, munições, dinheiro e recompensas existentes;
- enviar histórico e resultado consolidado quando houver alteração persistente.

### Backend

- persistir atores, itens, pilhas, localizações, propriedade, ações, perícias, profissões, passivas, receitas, comerciantes e sessões;
- calcular atributos secundários, peso, sobrecarga e pontuações efetivas;
- calcular preços-base, faixas comerciais e limites de qualidade;
- fornecer regras, orçamentos e conteúdos para criação e revisão;
- materializar atores, inventários, contêineres e drops reproduzíveis;
- gerar snapshots completos;
- controlar `stateVersion` e versões de regras;
- validar e reproduzir resoluções;
- validar propostas de criação, correção, balanceamento e evolução;
- controlar reservas de comércio e fabricação;
- aplicar consumo, criação, XP, níveis, saldos, propriedade e localização atomicamente;
- preservar IDs, vínculos, versões, origem e histórico;
- retornar erros acionáveis;
- manter auditoria.

### Frontend futuro

- exibir fichas, inventários, peso, slots, perícias, profissões, passivas e receitas;
- mostrar equipamentos, ações concedidas, condição, durabilidade e reservas;
- acompanhar combate, saque, comércio e fabricação;
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
    "economy": "economy-v0.2",
    "skills": "skills-v0.1",
    "crafting": "crafting-v0.1",
    "actors": "actors-v0.1",
    "inventory": "inventory-v0.1",
    "integration": "integration-v0.6"
  }
}
```

Toda resolução ou revisão persistente informa as versões utilizadas.

## 6. Atores e materialização

O GPT não precisa materializar toda entidade observada.

```text
avistamento mínimo
→ interação torna-se provável
→ GPT solicita materialização
→ backend fornece definição, orçamento e regras
→ GPT/backend criam a instância completa
→ backend valida
→ snapshot congelado é utilizado
```

O snapshot de ator materializado contém, quando aplicável:

- identidade, natureza, espécie e arquétipo;
- nível e categoria de ameaça;
- cinco atributos primários e secundários;
- recursos atuais e máximos;
- ações, magias, habilidades e passivas;
- equipamentos, inventário, consumíveis, munições e dinheiro;
- XP, drops e materiais coletáveis;
- facção, comportamento, moral e objetivos;
- vínculos, domesticação, recrutamento e grupo;
- conhecimento disponível ao jogador;
- semente de geração e versões usadas.

Depois da materialização, o GPT não acrescenta retroativamente recursos que não existiam. Revisões estruturais não alteram silenciosamente sessão em andamento.

## 7. Combate

O snapshot de combate contém:

- participantes materializados e relações;
- atributos e recursos finais;
- posições em metros;
- ambiente, obstáculos e coberturas;
- `combatTime`, `nextReadyAt` e fila de eventos;
- ações disponíveis e ações em preparação;
- equipamentos e inventário relevante;
- pilhas de munição, consumíveis, cargas e durabilidade;
- condições, recargas e engajamentos;
- política de rolagens.

O GPT mantém a linha do tempo localmente e registra declaração, resolução e estado antes/depois de cada evento relevante.

O backend valida:

- alcance e linha de visão;
- tempos e prioridades;
- recursos e custos;
- existência de munição, consumível ou carga;
- momento do consumo;
- rolagens, dano e condições;
- durabilidade e destruição quando aplicáveis;
- resultado final.

Um item consumido ou destruído durante o combate não pode reaparecer no saque.

## 8. Inventário, equipamento e saque

### 8.1 Snapshot de inventário

```json
{
  "inventorySessionId": "inventory-session-id",
  "stateVersion": 21,
  "ruleVersions": {
    "inventory": "inventory-v0.1",
    "equipment": "equipment-v0.4",
    "actors": "actors-v0.1",
    "integration": "integration-v0.6"
  },
  "actor": {},
  "inventory": {
    "stacks": [],
    "instances": [],
    "containers": [],
    "equipmentSlots": {},
    "currencyBalance": 0,
    "weightSummary": {}
  },
  "availableOperations": [],
  "constraints": {},
  "activeReservations": []
}
```

### 8.2 Estado local permitido

O GPT pode preparar localmente:

- seleção e comparação de itens;
- divisão ou união de pilhas;
- proposta de equipar ou desequipar;
- organização entre contêineres;
- consumo ou uso de cargas em sessão compatível;
- saque e coleta;
- descarte ou destruição permitida;
- separação para comércio;
- separação para fabricação.

Nenhuma propriedade ou localização é alterada persistentemente antes da confirmação do backend.

### 8.3 Operações consolidadas

```json
{
  "inventorySessionId": "inventory-session-id",
  "baseStateVersion": 21,
  "operations": [
    {
      "type": "TRANSFER_ITEM",
      "itemInstanceId": "item-id",
      "from": { "type": "CORPSE_LOOT", "referenceId": "loot-id" },
      "to": { "type": "ACTOR_INVENTORY", "referenceId": "player-id" },
      "quantity": 1
    }
  ]
}
```

### 8.4 Validação do backend

O backend confere:

1. sessão e versão;
2. existência da instância ou pilha;
3. quantidade;
4. propriedade e acesso;
5. localização atual;
6. compatibilidade de pilha;
7. capacidade e peso;
8. slots e requisitos;
9. vínculo e política de comércio;
10. reservas ativas;
11. ausência de duplicação;
12. resultado atômico.

### 8.5 Saque e drops

O snapshot de saque pode conter:

- equipamentos recuperáveis;
- inventário restante;
- dinheiro;
- itens ocultos ainda não revelados;
- fontes de coleta;
- regras de acesso;
- condições do corpo;
- semente e política de resolução.

O backend resolve ou reproduz drops usando a semente persistida. Recarregar a sessão não concede nova rolagem.

## 9. Reservas e prevenção de uso duplo

Itens podem ser reservados por:

```text
CRAFT_RESERVATION
TRADE_RESERVATION
QUEST_RESERVATION
SYSTEM_RESERVATION
```

Enquanto uma quantidade estiver reservada, ela não pode ser simultaneamente:

- consumida;
- vendida;
- transferida;
- equipada;
- destruída;
- reservada por operação incompatível.

Exemplo:

```json
{
  "reservationId": "reservation-id",
  "type": "CRAFT_RESERVATION",
  "allocations": [
    {
      "stackId": "iron-ingot-stack",
      "quantity": 3
    }
  ],
  "sessionId": "craft-session-id"
}
```

A conclusão consome ou libera a reserva conforme o resultado. Cancelamento seguro libera recursos que não foram consumidos.

## 10. Comércio

O snapshot comercial contém:

- saldo do jogador e comerciante;
- estoque como instâncias e pilhas reais;
- propriedade, condição, vínculo e estado de roubo;
- categorias aceitas;
- preços de referência;
- faixas mínima e máxima;
- reputação e relação;
- perícia de Comércio e passivas;
- regras de negociação;
- reservas ativas.

O GPT conduz diálogo e contrapropostas. O backend confirma preço, saldo, estoque, propriedade, quantidade e transferência atômica.

Ao iniciar confirmação de uma negociação, os itens podem ser colocados em `TRADE_RESERVATION` para impedir consumo ou venda simultânea.

## 11. Sessão de perícia

```json
{
  "skillSessionId": "skill-session-id",
  "stateVersion": 20,
  "ruleVersions": {
    "attributes": "attributes-v0.4",
    "skills": "skills-v0.1",
    "integration": "integration-v0.6"
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

O log registra perícia, atributos, pesos, dificuldade, passivas, rolagens, resultado, custos e XP reportado. O backend calcula XP autoritativo.

## 12. Sessão de fabricação

```json
{
  "craftSessionId": "craft-session-id",
  "stateVersion": 15,
  "ruleVersions": {
    "attributes": "attributes-v0.4",
    "equipment": "equipment-v0.4",
    "inventory": "inventory-v0.1",
    "skills": "skills-v0.1",
    "crafting": "crafting-v0.1",
    "integration": "integration-v0.6"
  },
  "actor": {},
  "profession": {},
  "skill": {},
  "recipe": {},
  "reservedMaterials": [],
  "reservedTools": [],
  "workspace": {},
  "resolution": {
    "successChance": 0,
    "qualityThresholds": {},
    "maximumQuality": "COMMON"
  }
}
```

O backend valida:

1. sessão e versão;
2. receita conhecida;
3. perícia e profissão;
4. materiais e ferramentas reservados;
5. oficina;
6. limites de qualidade;
7. rolagens e resultado;
8. consumo, desgaste e subprodutos;
9. item criado ou alterado;
10. XP e progressão;
11. atomicidade.

## 13. Criação de conteúdo

O GPT pode criar conteúdo necessário quando não existir definição adequada:

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
- regra de drop, saque ou recrutamento.

```text
GPT identifica necessidade
→ consulta conteúdos semelhantes e regras
→ backend retorna orçamento e restrições
→ GPT propõe definição completa
→ backend valida
→ GPT corrige se necessário
→ backend persiste versão aprovada
```

## 14. Revisão de conteúdo

Todo conteúdo pode ser revisado. Existência prévia não é motivo suficiente para rejeição.

Motivos:

```text
CORRECTION
BALANCE_REVISION
NARRATIVE_EVOLUTION
INSTANCE_STATE_CHANGE
SCOPE_PROMOTION
MIGRATION
```

Quando o jogador sugerir mudança, o GPT:

1. consulta definição e versões;
2. identifica problema ou objetivo;
3. verifica regras, orçamento, propriedade e identidade temática;
4. avalia impacto sobre vínculos e sessões;
5. aceita, adapta ou recusa;
6. explica objetivamente quando não fizer sentido;
7. envia revisão estruturada.

Escopos:

```text
DEFINITION_FUTURE_INSTANCES
CURRENT_INSTANCE
SELECTED_INSTANCES
ALL_COMPATIBLE_INSTANCES
```

Preferência:

- manter ID e código;
- criar nova versão;
- registrar motivo e justificativa;
- preservar vínculos;
- declarar política para instâncias existentes;
- não recalcular eventos históricos;
- permitir migração explícita quando segura.

## 15. Resolução direta ou por snapshot

### Direta

Adequada para operação curta e autoritativa, como mover uma unidade, consumir item simples ou equipar fora de encontro. O GPT envia intenção e o backend calcula e persiste.

### Snapshot

Adequada quando existem escolhas, várias etapas, risco, preparação ou narrativa relevante. O GPT resolve localmente e envia histórico final.

A política da operação define o modo.

## 16. Checkpoints

Podem ser usados quando:

- a sessão durar muito;
- houver criação, consumo ou transferência importante;
- ocorrer mudança de fase;
- a conversa puder ser interrompida;
- houver consequência persistente relevante.

Checkpoint valida um trecho e retorna nova `stateVersion`.

## 17. Concorrência

Se:

```text
baseStateVersion != currentStateVersion
```

O backend rejeita a submissão e informa:

- versão esperada e recebida;
- itens, pilhas ou atores alterados;
- reservas conflitantes;
- operações que podem ser reaplicadas;
- ações de recarregar, reconciliar ou cancelar com segurança.

## 18. Aleatoriedade

Na primeira versão, o GPT pode gerar e registrar d100. O backend valida faixa e coerência, mas não prova aleatoriedade.

Modo auditável futuro:

- semente no snapshot;
- sequência determinística;
- pacote de rolagens;
- assinatura do snapshot.

Drops materializados usam semente persistida mesmo antes do modo auditável completo.

## 19. Operações conceituais

```text
create/load/materialize/reviseActor
create/load/checkpoint/submitEncounter
create/load/submitInventorySession
add/remove/move/transfer/equip/consume/loot/harvestItem
reserve/releaseInventoryResources
create/load/submitTradeSession
create/load/submitSkillSession
create/load/submitCraftSession
create/revise/migrateContent
cancelSession
```

Os nomes definitivos dependem da API, mas cada operação possui finalidade única, validação previsível e retorno acionável.

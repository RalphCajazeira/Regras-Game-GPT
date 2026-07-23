# Integração entre GPT, Backend e Frontend

**Versão da proposta:** `integration-v0.3`  
**Status:** em validação

## 1. Objetivo

Permitir que o GPT conduza o jogo com fluidez, enquanto o backend mantém a autoridade sobre dados persistidos, regras versionadas, validações e consequências finais.

O backend não precisa resolver cada microação. A comunicação principal ocorre ao carregar uma interação, em checkpoints opcionais e ao concluir uma operação persistente.

Esse padrão é usado por combate, comércio e futuras interações complexas.

## 2. Regra central

```text
Backend = autoridade de dados, regras versionadas, validação e persistência.
GPT = condutor, interpretador e resolvedor local dentro do snapshot recebido.
Frontend = visualização e administração usando o mesmo domínio.
```

## 3. Responsabilidades

### GPT

O GPT deve:

- interpretar a intenção do jogador;
- consultar fichas, estoques e conteúdos persistidos;
- usar regras versionadas;
- manter estado local da interação;
- calcular chances, danos, tempos, posições, preços e modificadores permitidos;
- manter fila cronológica de eventos quando aplicável;
- gerar e registrar rolagens;
- narrar conforme os resultados mecânicos;
- impedir ações que o snapshot indique como inválidas;
- enviar histórico estruturado e resultado consolidado;
- não inventar custos, atributos, ações, itens, saldos, estoques, descontos, efeitos ou recompensas ausentes.

### Backend

O backend deve:

- persistir mundos, campanhas, atores, itens, ações, comerciantes, estoques e encontros;
- calcular atributos derivados;
- calcular preços-base e faixas comerciais;
- validar conteúdos criados pelo GPT;
- gerar snapshots completos;
- controlar `stateVersion` e versões de regras;
- validar resoluções e transações enviadas;
- reproduzir cálculos temporais e espaciais relevantes;
- persistir recursos, posições, condições, propriedade, saldos e consequências;
- calcular XP, subida de nível, recompensas e pontos não distribuídos;
- garantir operações atômicas;
- retornar erros acionáveis;
- manter histórico e auditoria.

### Frontend futuro

O frontend poderá:

- exibir fichas e recursos;
- mostrar equipamentos e ações concedidas;
- acompanhar posições em metros;
- exibir linha do tempo e eventos pendentes;
- apresentar chances, rolagens e histórico;
- exibir saldo, estoque, preços e ofertas;
- permitir revisão administrativa;
- compartilhar as mesmas regras e validações.

## 4. Padrão de sessão versionada

```text
1. GPT solicita criação ou carregamento da sessão.
2. Backend retorna snapshot completo e versionado.
3. GPT resolve decisões e diálogos localmente.
4. GPT mantém log estruturado.
5. Checkpoints podem ser enviados quando necessário.
6. GPT envia resultado consolidado.
7. Backend recalcula e valida.
8. Backend persiste o resultado autoritativo.
9. GPT apresenta o retorno final.
```

Toda sessão possui, quando aplicável:

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
    "attributes": "attributes-v0.3",
    "combat": "combat-v0.2",
    "equipment": "equipment-v0.3",
    "actions": "actions-v0.1",
    "economy": "economy-v0.1",
    "integration": "integration-v0.3"
  }
}
```

Toda resolução informa as versões utilizadas.

## 6. Fluxo do encontro de combate

```text
1. GPT solicita criação ou carregamento do encontro.
2. Backend retorna snapshot completo e versionado.
3. GPT inicializa combatTime, fila e disponibilidades.
4. GPT resolve decisões e eventos localmente.
5. GPT mantém log de cada declaração e resolução.
6. Checkpoints podem ser enviados quando necessário.
7. Ao encerrar, GPT envia a resolução consolidada.
8. Backend recalcula e reproduz os eventos.
9. Backend persiste o resultado autoritativo.
10. Backend aplica XP, níveis e demais consequências.
11. GPT apresenta o retorno final.
```

## 7. Snapshot de combate

```json
{
  "encounterId": "encounter-id",
  "stateVersion": 12,
  "ruleVersions": {
    "attributes": "attributes-v0.3",
    "combat": "combat-v0.2",
    "equipment": "equipment-v0.3",
    "actions": "actions-v0.1",
    "integration": "integration-v0.3"
  },
  "lifecycleStatus": "ACTIVE",
  "combatTime": 0,
  "participants": [],
  "relations": [],
  "environment": {
    "coordinateMode": "TWO_DIMENSIONAL",
    "widthMeters": 30,
    "heightMeters": 20,
    "terrain": [],
    "obstacles": [],
    "cover": []
  },
  "positions": [],
  "engagements": [],
  "pendingActions": [],
  "eventQueue": [],
  "conditions": [],
  "resolutionPolicy": {
    "randomMode": "GPT_D100",
    "minimumChance": 1,
    "maximumChance": 99,
    "ticksPerSecond": 10
  }
}
```

## 8. Participantes

Cada participante deve incluir:

- atributos finais e composição resumida;
- `initiative`;
- `movementSpeed`;
- `physicalActionSpeed`;
- `castingSpeed`;
- Vida, Mana e Vigor atuais e máximos;
- posição, raio ocupado e orientação quando relevante;
- `nextReadyAt`;
- equipamentos ativos;
- ações disponíveis completas;
- habilidades e magias;
- custos, recargas e condições;
- resistências e imunidades;
- reações disponíveis;
- recompensa-base de XP, quando aplicável;
- estado de derrota, fuga ou incapacidade.

Exemplo resumido:

```json
{
  "actorId": "player",
  "position": {
    "xMeters": 4,
    "yMeters": 8,
    "occupiedRadiusMeters": 0.5
  },
  "nextReadyAt": 0,
  "secondaryAttributes": {
    "initiative": 15,
    "movementSpeed": 4.5,
    "physicalActionSpeed": 8,
    "castingSpeed": 2
  },
  "availableActions": [
    { "code": "basic-dagger-attack" }
  ]
}
```

## 9. Fila de eventos

O snapshot pode iniciar vazio ou conter eventos já agendados.

```json
{
  "eventQueue": [
    {
      "sequence": 1,
      "eventType": "ACTION_RESOLVE",
      "scheduledAt": 20,
      "actorId": "mage",
      "actionInstanceId": "action-instance-id"
    }
  ]
}
```

O GPT mantém a fila ordenada por:

1. `scheduledAt`;
2. prioridade do tipo;
3. Iniciativa;
4. regra de desempate.

## 10. Ações pendentes

```json
{
  "pendingActions": [
    {
      "actionInstanceId": "action-instance-id",
      "actorId": "mage",
      "actionCode": "fire-arrow",
      "startedAt": 0,
      "resolveAt": 20,
      "nextReadyAt": 28,
      "target": {
        "actorId": "enemy-1"
      },
      "status": "WINDUP",
      "costsApplied": {
        "mana": 12
      }
    }
  ]
}
```

Estados conceituais:

```text
DECLARED
WINDUP
RESOLVED
INTERRUPTED
CANCELLED
RECOVERY
```

## 11. Estado local do combate

O GPT pode alterar localmente:

- `combatTime`;
- `nextReadyAt`;
- fila de eventos;
- ações pendentes;
- posições e distâncias;
- engajamentos;
- recursos atuais;
- condições;
- recargas;
- reações e dívida temporal;
- participantes derrotados;
- efeitos temporários.

O snapshot original permanece como base da validação final.

## 12. Log de combate

Cada entrada registra:

- sequência;
- tempo;
- tipo de evento;
- ator;
- alvo ou área;
- ação usada;
- posição antes e depois;
- parâmetros declarados antes da rolagem;
- tempos calculados;
- atributos consultados;
- fórmula e chance;
- rolagens;
- acerto, crítico e Defesa Crítica;
- dano, cura e mitigação;
- custos;
- interrupções;
- condições;
- estado antes e depois.

Exemplo:

```json
{
  "sequence": 14,
  "combatTime": 120,
  "eventType": "ACTION_RESOLVE",
  "actorId": "warrior",
  "actionCode": "heavy-strike",
  "timing": {
    "startedAt": 98,
    "resolveAt": 120,
    "nextReadyAt": 132
  },
  "result": {
    "hit": true,
    "finalDamage": 80
  }
}
```

## 13. Resolução final de combate

```json
{
  "encounterId": "encounter-id",
  "baseStateVersion": 12,
  "ruleVersions": {
    "attributes": "attributes-v0.3",
    "combat": "combat-v0.2",
    "equipment": "equipment-v0.3",
    "actions": "actions-v0.1",
    "integration": "integration-v0.3"
  },
  "outcome": "VICTORY",
  "finalCombatTime": 248,
  "events": [],
  "finalParticipants": [],
  "defeatedActorIds": [],
  "escapedActorIds": []
}
```

## 14. Validação do combate

O backend deve conferir:

1. encontro e snapshot;
2. `baseStateVersion`;
3. versões de regra;
4. participantes, relações e ações;
5. posições, distâncias e linha de visão;
6. tempos-base, ajustes, mínimos e fila;
7. ordem dos eventos;
8. custos e recargas;
9. alcance e revalidação em `resolveAt`;
10. chances e rolagens;
11. crítico, Defesa Crítica, mitigação e dano;
12. interrupções e dívida temporal de reações;
13. condições e recursos;
14. estados finais e resultado.

## 15. Fluxo da sessão comercial

```text
1. GPT solicita criação ou carregamento da sessão comercial.
2. Backend retorna saldos, estoque, ofertas, faixas e regras.
3. GPT conduz comparação, diálogo e negociação localmente.
4. GPT mantém log das propostas e decisões.
5. Ao concluir, GPT envia a transação consolidada.
6. Backend recalcula preços e valida saldo, estoque e propriedade.
7. Backend transfere itens e Coroas atomicamente.
8. GPT apresenta o resultado autoritativo.
```

## 16. Snapshot comercial

```json
{
  "tradeSessionId": "trade-session-id",
  "stateVersion": 8,
  "ruleVersions": {
    "equipment": "equipment-v0.3",
    "economy": "economy-v0.1",
    "integration": "integration-v0.3"
  },
  "lifecycleStatus": "ACTIVE",
  "currencyCode": "CROWN",
  "playerBalance": 500,
  "merchant": {},
  "merchantBalance": 1200,
  "stock": [],
  "buyOffers": [],
  "negotiationRules": {},
  "resolutionPolicy": {
    "allowLocalNegotiation": true,
    "requireFinalValidation": true
  }
}
```

O snapshot comercial contém:

- saldos;
- estoque e quantidades;
- categorias aceitas;
- preços de referência e faixas;
- condição dos itens;
- especialidade;
- reputação e relação;
- regras de negociação;
- versão do estado.

## 17. Estado local e log comercial

O GPT pode manter:

- itens selecionados;
- contrapropostas;
- descontos permitidos;
- trocas;
- rolagens sociais;
- valor líquido provisório.

O log registra:

- proposta inicial;
- itens e quantidades;
- referências e faixas;
- argumentos;
- modificadores;
- rolagens;
- contrapropostas;
- oferta final;
- motivo de recusa.

## 18. Transação consolidada

```json
{
  "tradeSessionId": "trade-session-id",
  "baseStateVersion": 8,
  "ruleVersions": {
    "equipment": "equipment-v0.3",
    "economy": "economy-v0.1",
    "integration": "integration-v0.3"
  },
  "purchases": [],
  "sales": [],
  "currencyTransferred": 120,
  "finalOffers": [],
  "negotiationLog": []
}
```

O backend valida sessão, saldos, estoque, propriedade, categorias, preços, faixas, total, ausência de duplicação e atomicidade.

## 19. Aleatoriedade

Na primeira versão, o GPT pode gerar d100 e registrar os valores.

O backend valida faixa e coerência, mas não prova aleatoriedade.

Modo auditável futuro:

- semente no snapshot;
- sequência determinística;
- pacote de rolagens;
- assinatura do snapshot.

## 20. Checkpoints

Checkpoints são opcionais quando:

- o combate durar muitos eventos;
- um participante importante for derrotado;
- ocorrer mudança de fase;
- a conversa puder ser interrompida;
- houver consequência persistente relevante;
- uma negociação longa precisar ser preservada.

O checkpoint retorna nova `stateVersion` e snapshot atualizado.

## 21. Concorrência

Se:

```text
baseStateVersion != currentStateVersion
```

O backend rejeita a submissão e informa:

- versão esperada;
- versão recebida;
- alterações conflitantes;
- opções de recarregar, reaplicar ou cancelar com segurança.

## 22. Operações conceituais

```text
createEncounter
loadEncounter
checkpointEncounter
submitEncounterResolution
cancelEncounter

createTradeSession
loadTradeSession
submitTradeResolution
cancelTradeSession
```

Os nomes definitivos dependem do backend.

## 23. Criação e revisão de conteúdo

O GPT pode propor e revisar:

- atores;
- criaturas;
- itens;
- ações;
- habilidades;
- magias;
- efeitos;
- comerciantes;
- recompensas.

O backend valida antes de persistir.

Revisões devem preferir manter o mesmo identificador e criar nova versão, preservando vínculos e histórico. Eventos antigos não são recalculados automaticamente.

## 24. Erros acionáveis

Exemplo:

```json
{
  "success": false,
  "error": {
    "code": "ENCOUNTER_STATE_VERSION_CONFLICT",
    "message": "A resolução foi criada a partir da versão 12, mas o encontro está na versão 13.",
    "details": {
      "expectedStateVersion": 13,
      "receivedStateVersion": 12
    },
    "recoveryActions": [
      "Recarregue o encontro.",
      "Reaplique somente os eventos ainda compatíveis.",
      "Cancele o encontro com segurança caso não seja possível reconciliar."
    ]
  }
}
```

## 25. Pendências

- definir esquema completo de ambiente e obstáculos;
- definir precisão espacial necessária;
- definir tamanho máximo de histórico sem checkpoint;
- definir política auditável de aleatoriedade;
- definir compactação de logs;
- validar reprodução da fila de eventos;
- definir integração visual da linha do tempo no frontend.
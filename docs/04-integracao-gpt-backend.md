# Integração entre GPT, Backend e Frontend

**Versão da proposta:** `integration-v0.2`  
**Status:** em validação

## 1. Objetivo

Permitir que o GPT conduza o jogo com fluidez, enquanto o backend mantém a autoridade sobre dados persistidos, validações e consequências finais.

O backend não precisa resolver cada microação. A comunicação principal ocorre ao carregar uma interação, em checkpoints opcionais e ao concluir uma operação persistente.

Esse padrão é usado por combate, comércio e futuras interações complexas.

## 2. Regra central

```text
Backend = autoridade de dados, regras versionadas, validação e persistência.
GPT = condutor, interpretador e resolvedor local dentro do snapshot recebido.
Frontend = visualização e administração usando o mesmo domínio.
```

## 3. Responsabilidades

### 3.1 GPT

O GPT deve:

- interpretar a intenção do jogador;
- consultar fichas, estoques e conteúdos persistidos;
- usar regras versionadas;
- manter estado local da interação;
- calcular chances, danos, preços e modificadores permitidos;
- gerar e registrar rolagens quando necessárias;
- narrar conforme os resultados mecânicos;
- impedir ações que o snapshot indique como inválidas;
- enviar ao backend histórico estruturado e resultado consolidado;
- não inventar custos, atributos, itens, saldos, estoques, descontos, efeitos ou recompensas ausentes.

### 3.2 Backend

O backend deve:

- persistir mundos, campanhas, atores, itens, comerciantes, estoques e encontros;
- calcular atributos derivados;
- calcular preços-base e faixas comerciais;
- validar conteúdos criados pelo GPT;
- gerar snapshots completos;
- controlar `stateVersion` e versões de regras;
- validar resoluções e transações enviadas;
- persistir recursos, condições, propriedade, saldos e consequências;
- calcular XP, subida de nível, recompensas e pontos não distribuídos;
- garantir operações atômicas;
- retornar erros acionáveis;
- manter histórico e auditoria.

### 3.3 Frontend futuro

O frontend poderá:

- exibir fichas e recursos;
- mostrar equipamentos e modificadores;
- acompanhar encontros;
- apresentar chances, rolagens e histórico;
- exibir saldo, estoque, preços e ofertas;
- permitir revisão administrativa;
- compartilhar as mesmas regras e validações do GPT e do backend.

## 4. Padrão de sessão versionada

Interações complexas utilizam:

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

Exemplo:

```json
{
  "ruleVersions": {
    "attributes": "attributes-v0.2",
    "combat": "combat-v0.1",
    "equipment": "equipment-v0.2",
    "economy": "economy-v0.1",
    "integration": "integration-v0.2"
  }
}
```

Uma resolução deve informar as versões utilizadas para poder ser reproduzida.

## 6. Fluxo do encontro de combate

```text
1. GPT solicita criação ou carregamento do encontro.
2. Backend retorna snapshot completo e versionado.
3. GPT resolve as decisões localmente.
4. GPT mantém log estruturado de cada ação.
5. Checkpoints podem ser enviados quando necessário.
6. Ao encerrar, GPT envia a resolução consolidada.
7. Backend recalcula e valida.
8. Backend persiste o resultado autoritativo.
9. Backend aplica XP, níveis e demais consequências.
10. GPT narra ou apresenta o retorno autoritativo final.
```

### 6.1 Snapshot de combate

```json
{
  "encounterId": "encounter-id",
  "stateVersion": 12,
  "ruleVersions": {
    "attributes": "attributes-v0.2",
    "combat": "combat-v0.1",
    "equipment": "equipment-v0.2"
  },
  "lifecycleStatus": "ACTIVE",
  "participants": [],
  "relations": [],
  "positions": [],
  "conditions": [],
  "availableActions": [],
  "environment": {},
  "resolutionPolicy": {
    "randomMode": "GPT_D100",
    "minimumChance": 1,
    "maximumChance": 99
  }
}
```

Cada participante deve incluir:

- atributos finais;
- composição resumida dos atributos;
- Vida, Mana e Vigor atuais e máximos;
- equipamentos ativos;
- habilidades e magias disponíveis;
- custos, recargas e condições;
- resistências e imunidades;
- recompensa-base de XP, quando aplicável;
- estado de derrota, fuga ou incapacidade.

### 6.2 Estado local do combate

O GPT pode alterar localmente:

- recursos atuais;
- posição e distância;
- condições;
- recargas;
- participantes derrotados;
- ordem de turnos;
- efeitos temporários.

O snapshot original permanece como base da validação final.

### 6.3 Log de combate

Cada ação deve registrar:

- turno;
- ator;
- alvo ou área;
- ação usada;
- parâmetros declarados antes da rolagem;
- atributos consultados;
- fórmula e chance calculada;
- rolagens;
- acerto, crítico e Defesa Crítica;
- dano e mitigação;
- custos;
- condições aplicadas ou removidas;
- estado antes e depois.

### 6.4 Resolução de combate

```json
{
  "encounterId": "encounter-id",
  "baseStateVersion": 12,
  "ruleVersions": {
    "attributes": "attributes-v0.2",
    "combat": "combat-v0.1",
    "equipment": "equipment-v0.2"
  },
  "outcome": "VICTORY",
  "actions": [],
  "finalParticipants": [],
  "defeatedActorIds": ["enemy-1"],
  "escapedActorIds": [],
  "notes": []
}
```

### 6.5 Validação de combate

O backend deve conferir:

1. encontro e snapshot;
2. `baseStateVersion`;
3. versões de regra;
4. atores, ações e alvos;
5. alcance e relações;
6. recursos e custos;
7. chances e rolagens;
8. crítico, Defesa Crítica, mitigação e dano;
9. condições e recargas;
10. estados finais e resultado.

## 7. Fluxo da sessão comercial

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

### 7.1 Snapshot comercial

```json
{
  "tradeSessionId": "trade-session-id",
  "stateVersion": 8,
  "ruleVersions": {
    "equipment": "equipment-v0.2",
    "economy": "economy-v0.1",
    "integration": "integration-v0.2"
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

O snapshot deve conter:

- saldo do jogador;
- saldo do comerciante;
- estoque e quantidades;
- itens que o comerciante aceita comprar;
- `referencePrice`, `minimumAllowedPrice` e `maximumAllowedPrice`;
- condição dos itens;
- especialidade e perfil comercial;
- reputação e relação aplicáveis;
- regras e flexibilidade de negociação;
- versão do estado.

### 7.2 Estado local da negociação

O GPT pode manter localmente:

- itens selecionados;
- contrapropostas;
- descontos ainda permitidos;
- itens oferecidos em troca;
- resultado de rolagens sociais;
- valor líquido provisório.

O GPT não altera saldos ou propriedade de forma persistente antes da confirmação do backend.

### 7.3 Log comercial

Cada negociação deve poder registrar:

- proposta inicial;
- itens e quantidades;
- preços de referência;
- faixas autorizadas;
- argumentos relevantes;
- modificadores aplicados;
- rolagens, quando existirem;
- contrapropostas;
- oferta final aceita;
- motivo de recusa, quando aplicável.

### 7.4 Transação consolidada

```json
{
  "tradeSessionId": "trade-session-id",
  "baseStateVersion": 8,
  "ruleVersions": {
    "equipment": "equipment-v0.2",
    "economy": "economy-v0.1"
  },
  "purchases": [],
  "sales": [],
  "currencyTransferred": 120,
  "finalOffers": [],
  "negotiationLog": []
}
```

### 7.5 Validação comercial

O backend deve conferir:

1. sessão e versão;
2. saldos;
3. estoque e propriedade;
4. categorias aceitas;
5. preços-base;
6. condição e mercado;
7. faixas autorizadas;
8. negociação;
9. total monetário;
10. ausência de duplicação;
11. transferência completa e atômica.

## 8. Aleatoriedade

Na primeira versão, o GPT pode gerar números d100 e registrá-los.

O backend consegue validar faixa e coerência, mas não provar que o número foi aleatório.

Modo auditável futuro:

- semente fornecida no snapshot;
- sequência determinística;
- pacote de rolagens pré-gerado;
- assinatura do snapshot.

Esse padrão pode ser usado tanto em combate quanto em negociação.

## 9. Checkpoints

Checkpoints são opcionais e podem ser usados quando:

- uma interação durar muitas etapas;
- ocorrer consequência persistente importante;
- um participante importante for derrotado;
- estoque ou saldo precisar ser reservado;
- ocorrer mudança de fase;
- a conversa estiver perto de ser interrompida.

Um checkpoint valida e persiste um trecho e retorna nova `stateVersion`.

## 10. Concorrência e divergência

Se o estado persistido mudar durante a resolução local:

```text
baseStateVersion != currentStateVersion
```

O backend deve rejeitar a submissão e informar:

- versão esperada;
- versão recebida;
- alterações conflitantes;
- possibilidade de recarregar, reaplicar ou cancelar com segurança.

Isso evita vender o mesmo item duas vezes, gastar saldo já utilizado ou concluir combate sobre um estado alterado.

## 11. Operações conceituais

### Combate

```text
createEncounter
loadEncounter
checkpointEncounter
submitEncounterResolution
cancelEncounter
```

### Comércio

```text
createTradeSession
loadTradeSession
checkpointTradeSession
submitTradeTransaction
cancelTradeSession
```

### Conteúdo e administração

```text
createContentDefinition
reviseContentDefinition
createMerchant
updateMerchantStock
grantCurrencyReward
```

Os nomes definitivos dependem do backend, mas cada operação deve ter finalidade única e resposta acionável.

## 12. Criação e revisão de conteúdo

O GPT pode propor:

- atores e criaturas;
- itens e equipamentos;
- habilidades, magias e efeitos;
- comerciantes e estoques iniciais;
- serviços;
- recompensas;
- preços-base e categorias econômicas.

O backend valida a estrutura e as regras antes de persistir.

Revisões devem preferir manter o mesmo identificador e criar nova versão da definição, preservando vínculos e histórico. Eventos e transações antigas não são recalculados automaticamente.

## 13. Erros acionáveis

Exemplo de conflito:

```json
{
  "success": false,
  "error": {
    "code": "STATE_VERSION_CONFLICT",
    "message": "A resolução foi criada a partir da versão 12, mas a sessão está na versão 13.",
    "details": {
      "expectedStateVersion": 13,
      "receivedStateVersion": 12
    },
    "recoveryActions": [
      "Recarregue a sessão.",
      "Reaplique somente as ações ainda compatíveis.",
      "Cancele com segurança caso não seja possível reconciliar."
    ]
  }
}
```

## 14. Regra final

O GPT possui liberdade narrativa e operacional dentro do snapshot. O backend define os limites, reproduz os cálculos e somente persiste resultados válidos.

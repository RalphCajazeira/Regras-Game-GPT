# Integração entre GPT, Backend e Frontend

**Versão da proposta:** `integration-v0.4`  
**Status:** em validação

## 1. Objetivo

Permitir que o GPT conduza interações com fluidez, enquanto o backend mantém autoridade sobre dados persistidos, regras versionadas, validações e consequências permanentes.

O padrão se aplica a combate, comércio, perícias, profissões, fabricação e futuras sessões complexas.

## 2. Regra central

```text
Backend = autoridade de dados, regras, validação e persistência.
GPT = condutor e resolvedor local dentro do snapshot recebido.
Frontend = visualização e administração usando o mesmo domínio.
```

## 3. Responsabilidades

### GPT

- interpretar a intenção do jogador;
- carregar fichas, ações, perícias, receitas, estoques e conteúdos persistidos;
- usar somente regras e parâmetros versionados;
- manter estado local da sessão;
- calcular chances, danos, tempos, posições, preços e resultados permitidos;
- registrar rolagens e decisões;
- não inventar atributos, perícias, passivas, receitas, itens, custos ou recompensas;
- enviar histórico e resultado consolidado quando houver alteração persistente.

### Backend

- persistir atores, itens, ações, perícias, profissões, passivas, receitas, comerciantes e sessões;
- calcular atributos secundários e pontuações efetivas;
- calcular preços-base, faixas comerciais e limites de qualidade;
- gerar snapshots completos;
- controlar `stateVersion` e versões de regras;
- validar e reproduzir resoluções;
- aplicar consumo, criação, XP, níveis, saldos e propriedade atomicamente;
- retornar erros acionáveis;
- manter histórico e auditoria.

### Frontend futuro

- exibir fichas, perícias, profissões, passivas e receitas;
- mostrar equipamentos e ações concedidas;
- acompanhar combate, comércio e fabricação;
- apresentar chances, rolagens, tempos, qualidade e histórico;
- permitir revisão administrativa usando as mesmas validações.

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
    "integration": "integration-v0.4"
  }
}
```

Toda resolução persistente informa as versões utilizadas.

## 6. Combate

O snapshot de combate contém:

- participantes e relações;
- cinco atributos primários e secundários finais;
- posições em metros;
- ambiente, obstáculos e coberturas;
- `combatTime`, `nextReadyAt` e fila de eventos;
- ações disponíveis e ações em preparação;
- recursos, condições, recargas e engajamentos;
- política de rolagens.

O GPT mantém a linha do tempo localmente e envia declaração, resolução e estado antes/depois de cada evento relevante.

O backend valida alcance, tempos, custos, rolagens, dano, condições e resultado final.

## 7. Comércio

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

## 8. Sessão de perícia

```json
{
  "skillSessionId": "skill-session-id",
  "stateVersion": 20,
  "ruleVersions": {
    "attributes": "attributes-v0.4",
    "skills": "skills-v0.1",
    "integration": "integration-v0.4"
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

## 9. Sessão de fabricação

```json
{
  "craftSessionId": "craft-session-id",
  "stateVersion": 15,
  "ruleVersions": {
    "attributes": "attributes-v0.4",
    "equipment": "equipment-v0.4",
    "skills": "skills-v0.1",
    "crafting": "crafting-v0.1",
    "integration": "integration-v0.4"
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

## 10. Resolução direta ou por snapshot

### Direta

Adequada para operação curta e altamente autoritativa. O GPT envia a intenção e o backend calcula e persiste imediatamente.

### Snapshot

Adequada quando existem escolhas, várias etapas, risco, preparação ou narrativa relevante. O GPT resolve localmente e envia o histórico final.

A escolha do modo pertence à operação ou política retornada pelo backend.

## 11. Checkpoints

Podem ser usados quando:

- a sessão durar muito;
- existir criação ou consumo importante;
- ocorrer mudança de fase;
- a conversa puder ser interrompida;
- houver consequência persistente relevante.

Um checkpoint valida um trecho e retorna nova `stateVersion`.

## 12. Concorrência

Se:

```text
baseStateVersion != currentStateVersion
```

O backend rejeita a resolução e informa versão esperada, recebida, conflitos e ações de recuperação.

## 13. Aleatoriedade

Na primeira versão, o GPT pode gerar e registrar d100. O backend valida faixa e coerência, mas não prova aleatoriedade.

Modo auditável futuro:

- semente no snapshot;
- sequência determinística;
- pacote de rolagens;
- assinatura do snapshot.

## 14. Operações conceituais

```text
create/load/checkpoint/submitEncounter
create/load/submitTradeSession
create/load/submitSkillSession
create/load/submitCraftSession
cancelSession
```

Os nomes definitivos dependem da API, mas cada operação deve possuir finalidade única e retorno acionável.

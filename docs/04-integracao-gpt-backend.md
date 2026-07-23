# Integração entre GPT, Backend e Frontend

**Versão da proposta:** `integration-v0.1`  
**Status:** em validação

## 1. Objetivo

Permitir que o GPT conduza o jogo com fluidez, enquanto o backend mantém a autoridade sobre dados persistidos, validações e consequências finais.

O backend não precisa resolver cada microação do combate. A comunicação principal ocorre no início, em checkpoints opcionais e no encerramento.

## 2. Responsabilidades

### 2.1 GPT

O GPT deve:

- interpretar a intenção do jogador;
- consultar fichas e conteúdos persistidos;
- usar as regras versionadas;
- manter um estado local do encontro;
- calcular chances e danos;
- gerar e registrar rolagens;
- narrar conforme os resultados mecânicos;
- impedir ações que o snapshot indique como inválidas;
- enviar ao backend um histórico estruturado;
- não inventar custos, atributos, itens, efeitos ou recompensas ausentes.

### 2.2 Backend

O backend deve:

- persistir mundos, campanhas, atores, itens e encontros;
- calcular atributos derivados;
- validar conteúdos criados pelo GPT;
- gerar snapshots completos;
- controlar `stateVersion` e versões de regras;
- validar a resolução enviada;
- persistir recursos finais, condições e consequências;
- calcular XP, subida de nível e pontos não distribuídos;
- retornar erros acionáveis;
- manter histórico e auditoria.

### 2.3 Frontend futuro

O frontend poderá:

- exibir fichas e recursos;
- mostrar equipamentos e modificadores;
- acompanhar encontros;
- apresentar chances, rolagens e histórico;
- permitir revisão administrativa;
- compartilhar as mesmas regras e validações do GPT e do backend.

## 3. Fluxo recomendado do encontro

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

## 4. Snapshot inicial

O snapshot deve conter tudo que o GPT precisa para não consultar o backend a cada turno.

```json
{
  "encounterId": "encounter-id",
  "stateVersion": 12,
  "ruleVersions": {
    "attributes": "attributes-v0.2",
    "combat": "combat-v0.1",
    "equipment": "equipment-v0.1"
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

## 5. Estado local do GPT

Durante o encontro, o GPT mantém uma cópia de trabalho derivada do snapshot.

Ele pode alterar localmente:

- recursos atuais;
- posição e distância;
- condições;
- recargas;
- participantes derrotados;
- ordem de turnos;
- efeitos temporários.

O snapshot original não é substituído. Ele permanece como base para a validação final.

## 6. Log de ações

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

O log permite que o backend reproduza e confira o resultado.

## 7. Resolução final

Exemplo resumido:

```json
{
  "encounterId": "encounter-id",
  "baseStateVersion": 12,
  "ruleVersions": {
    "attributes": "attributes-v0.2",
    "combat": "combat-v0.1",
    "equipment": "equipment-v0.1"
  },
  "outcome": "VICTORY",
  "actions": [],
  "finalParticipants": [
    {
      "actorId": "player",
      "currentHealth": 48,
      "currentMana": 12,
      "currentStamina": 30,
      "conditions": []
    }
  ],
  "defeatedActorIds": ["enemy-1"],
  "escapedActorIds": [],
  "notes": []
}
```

## 8. Validação final

O backend deve conferir:

1. se o encontro e o snapshot existem;
2. se `baseStateVersion` ainda é atual;
3. se as versões de regra são aceitas;
4. se atores, ações e alvos existiam;
5. se alcance e relações permitiam as ações;
6. se recursos e custos eram suficientes;
7. se as chances foram calculadas corretamente;
8. se as rolagens produzem os resultados informados;
9. se crítico, Defesa Crítica e dano estão corretos;
10. se condições, recargas e recursos finais são reproduzíveis;
11. se participantes derrotados realmente chegaram ao estado de derrota;
12. se o resultado final é compatível com o histórico.

## 9. Aleatoriedade

Na primeira versão, o GPT pode gerar números d100 e registrá-los.

O backend consegue validar a faixa e a coerência da resolução, mas não provar que o número foi aleatório.

Modo auditável futuro:

- semente fornecida no snapshot;
- sequência determinística;
- pacote de rolagens pré-gerado;
- assinatura do snapshot.

Qualquer uma dessas opções mantém uma única chamada inicial, sem exigir comunicação a cada ataque.

## 10. Checkpoints

Checkpoints são opcionais e podem ser usados quando:

- o encontro durar muitas rodadas;
- um participante importante for derrotado;
- ocorrer mudança de fase;
- a conversa estiver perto de ser interrompida;
- houver consequência persistente relevante.

Um checkpoint não encerra o combate. Ele valida e persiste um trecho do histórico e retorna uma nova `stateVersion`.

## 11. Concorrência e divergência

Se o estado persistido mudar durante a resolução local:

```text
baseStateVersion != currentStateVersion
```

O backend deve rejeitar a submissão com erro acionável e informar:

- versão esperada;
- versão recebida;
- alterações conflitantes;
- possibilidade de recarregar, reaplicar ou cancelar com segurança.

## 12. Operações conceituais

A API poderá expor operações equivalentes a:

```text
createEncounter
loadEncounter
checkpointEncounter
submitEncounterResolution
cancelEncounter
```

Os nomes definitivos dependem do backend, mas cada operação deve ter uma finalidade única e resposta acionável.

## 13. Criação e revisão de conteúdo

O GPT também pode propor:

- atores;
- criaturas;
- itens;
- habilidades;
- magias;
- efeitos;
- recompensas.

O backend valida a estrutura e as regras antes de persistir.

Revisões devem preferir manter o mesmo identificador e criar nova versão da definição, preservando vínculos e histórico. Eventos antigos não são recalculados automaticamente.

## 14. Erros acionáveis

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
      "Reaplique somente as ações ainda compatíveis.",
      "Cancele o encontro com segurança caso não seja possível reconciliar."
    ]
  }
}
```

## 15. Regra central

```text
Backend = autoridade de dados, validação e persistência.
GPT = condutor, interpretador e resolvedor local das regras.
Frontend = visualização e administração compartilhando o mesmo domínio.
```

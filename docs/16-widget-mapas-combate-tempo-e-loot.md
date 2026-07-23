# Widget de Mapas, Combate, Tempo e Loot

**Versão da proposta:** `widget-gameplay-v0.1`  
**Status:** em validação

## 1. Objetivo

Definir a experiência interativa principal dentro do ChatGPT, permitindo exploração, viagem, batalha, inventário e saque por interface visual sem retirar do GPT a liberdade narrativa.

## 2. Modos visuais

```text
CHARACTER_SHEET
EXPLORATION_MAP
LOCATION_VIEW
TACTICAL_BATTLE_MAP
INVENTORY
LOOT
TRADE
CRAFTING
MISSION_LOG
RELATIONSHIP_VIEW
```

O widget pode alternar de modo sem perder a sessão oficial.

## 3. Mapa de exploração

A primeira versão pode ser simples:

- pontos de interesse;
- rotas entre pontos;
- posição atual;
- destino selecionado;
- distância;
- tempo estimado;
- risco conhecido;
- locais conhecidos e ocultos;
- animação de deslocamento.

O mapa não precisa começar com geografia detalhada. Pode ser um grafo visual de locais e rotas.

### 3.1 Prévia de viagem

```json
{
  "originLocationId": "city-id",
  "destinationLocationId": "forest-id",
  "distanceKilometers": 8,
  "estimatedDurationMinutes": 120,
  "knownRisk": "MODERATE",
  "previewOnly": true,
  "baseStateVersion": 15,
  "ruleVersions": {}
}
```

### 3.2 Confirmação

```text
widget confirma destino
→ advanceWorldTime / resolveGameCommand
→ backend valida rota e recursos
→ avança o relógio
→ resolve encontros e eventos aplicáveis
→ retorna nova posição, horário e possível transição
```

## 4. Tempo

### 4.1 Tempo do mundo

```text
worldTime
```

Usado para:

- viagem;
- dia e noite;
- descanso;
- lojas;
- missões com prazo;
- fabricação;
- treinamento;
- eventos do mundo.

### 4.2 Tempo do encontro

```text
sessionTimeTicks
```

Proposta inicial:

```text
10 ticks = 1 segundo
```

O tempo de encontro consumido pode avançar o relógio do mundo ao final.

## 5. Entrada no modo de batalha

```text
EXPLORATION_MAP
→ encontro materializado e COMBAT_READY
→ TACTICAL_BATTLE_MAP
```

O widget recebe:

- participantes visíveis;
- posições;
- raio ocupado;
- obstáculos;
- cobertura;
- terreno;
- alcance das ações selecionadas;
- áreas de ameaça conhecidas;
- recursos;
- linha temporal;
- decisão atual.

Inimigos ocultos não são enviados como entidades visíveis. Informações exclusivas do Mestre permanecem fora da superfície do jogador.

## 6. Mapa tático

A primeira versão pode usar:

- plano 2D simples;
- pontos ou ícones;
- escala em metros;
- círculos de alcance;
- linhas de caminho;
- áreas de efeito;
- animações básicas;
- painel lateral com ações.

Não é necessário começar com sprites complexos.

## 7. Clique para movimentar

O jogador seleciona um ponto.

O widget calcula ou solicita prévia:

```text
distância
caminho
posição final
tempo em ticks
custo de Vigor
terreno atravessado
cobertura final
alcances após o movimento
reações possíveis conhecidas
```

A confirmação envia comando estruturado. O backend recalcula o caminho e confirma a posição oficial.

Pontos inválidos devem ser visualmente diferenciados.

## 8. Janela de decisão

O sistema mantém linha do tempo contínua, mas a interface apresenta decisões semelhantes a turnos.

```text
ator do jogador disponível
→ widget abre janela de decisão
→ jogador escolhe ação ou plano
→ backend resolve até a próxima decisão relevante
→ widget apresenta novo estado
```

O jogador não precisa acompanhar cada tick manualmente.

## 9. Plano de ações

O widget pode permitir uma sequência limitada:

```text
mover até alcance
→ atacar
→ recuar
```

Exemplo:

```json
{
  "command": "ACTION_PLAN",
  "actorId": "actor-id",
  "steps": [
    {
      "type": "MOVE_INTO_RANGE",
      "targetActorId": "enemy-id",
      "desiredRangeMeters": 1.8
    },
    {
      "type": "USE_ACTION",
      "actionCode": "basic-sword-attack",
      "targetActorId": "enemy-id"
    },
    {
      "type": "MOVE_TO_POINT",
      "point": { "x": 7, "y": 4 }
    }
  ],
  "resolutionBoundary": "NEXT_PLAYER_DECISION",
  "baseStateVersion": 15
}
```

O plano agenda intenções. Não garante que todos os passos ocorrerão.

Pode ser interrompido por:

```text
TARGET_INVALIDATED
ACTOR_INTERRUPTED
RESOURCE_DEPLETED
REACTION_TRIGGERED
UNEXPECTED_EVENT
PHASE_TRANSITION
PLAYER_DECISION_REQUIRED
```

## 10. Horizonte de planejamento

O backend informa:

```text
maximumPlanningTicks
maximumSteps
allowedCompositeCommands
```

Isso impede filas excessivas de ações futuras.

## 11. Esperar ou passar

Ação canônica:

```text
WAIT
```

Opções:

```text
WAIT_TICKS
WAIT_UNTIL_NEXT_EVENT
WAIT_UNTIL_ENEMY_ACTION
HOLD_POSITION
DEFENSIVE_WAIT
```

Esperar avança o tempo real do encontro; não é uma rodada abstrata gratuita.

## 12. Prévia de ação

Ao selecionar ação e alvo, mostrar quando conhecido:

```text
alcance válido
movimento necessário
chance estimada de acerto
dano estimado
custos
tempo de preparação
recuperação
posição final
riscos conhecidos
```

A prévia deve declarar que não é resultado oficial.

## 13. Resolução oficial

Uma chamada pode resolver:

- movimento;
- preparação;
- ataque;
- reação;
- efeitos imediatos;
- condições;
- ações automáticas de NPCs;
- avanço temporal;
- próxima decisão.

Exemplo de retorno:

```json
{
  "status": "SUCCESS",
  "resolvedEvents": [],
  "snapshotDelta": {},
  "nextDecision": {
    "actorId": "player-id",
    "sessionTimeTicks": 34,
    "availableCommands": []
  },
  "narrationDirective": {
    "level": "COMPACT"
  }
}
```

## 14. Ações criativas

O jogador continua podendo escrever livremente.

Exemplo:

```text
“Corro sobre a mesa, salto por cima do guarda e corto a corda do lustre.”
```

O GPT estrutura a intenção e chama `resolveCreativeIntent`. O backend materializa objetos mecânicos necessários, valida as etapas e resolve até uma fronteira segura.

O widget representa visualmente apenas o que possuir modelo visual compatível.

## 15. Controle dos NPCs

NPCs genéricos podem agir automaticamente conforme comportamento carregado.

NPCs importantes recebem intenção de alto nível do GPT, quando necessário.

O backend interrompe a resolução quando uma decisão narrativa relevante precisar do GPT ou do jogador.

## 16. Fim do combate

Resultado final deve conter:

```text
vencedores
derrotados
mortos
inconscientes
capturados
fugitivos
recursos gastos
condições restantes
equipamentos danificados
XP proposto
loot formado
consequências
resumo mecânico
narrationDirective
```

## 17. Loot clicável

Após o combate, entidades podem permanecer como:

```text
CORPSE
LOOT_CONTAINER
HARVEST_SOURCE
INCAPACITATED_ACTOR
```

Ao clicar, o widget abre os itens realmente existentes.

Ações:

```text
TAKE_ALL
TAKE_SELECTED
COMPARE
EQUIP_FROM_LOOT
LEAVE_ITEM
HARVEST
EVALUATE_ITEM
```

Nada novo é inventado ao abrir o painel.

## 18. Passagem do combate para loot

```text
COMBAT_COMPLETE
→ backend forma loot e coleta possíveis
→ widget muda para LOOT
→ jogador seleciona itens
→ manageInventory transfere propriedade
→ widget retorna a EXPLORATION_MAP ou LOCATION_VIEW
```

## 19. Narração seletiva

O widget não solicita texto do GPT para cada ação comum.

Solicitar narração em:

- crítico importante;
- derrota;
- rendição;
- descoberta;
- mudança de fase;
- consequência permanente;
- decisão narrativa;
- início e fim de encontro;
- ação criativa.

## 20. Sincronização do GPT

Ações diretas do widget geram resumos compactos:

```text
Desde a última narração:
- Kael moveu-se 8 m.
- Acertou o Líder Bandido por 17 de dano.
- Sofreu 8 de dano.
- O inimigo se rendeu.
```

Não enviar cada animação ou clique ao contexto do modelo.

## 21. Recuperação

Ao remontar o widget:

```text
loadGameContext
→ sessão ativa
→ loadWidgetView
→ restaura modo, posições, tempo e decisão atual
```

A interface deve funcionar após recarregar a conversa ou abrir outro widget.

## 22. Acessibilidade e dispositivos

O widget deve prever:

- teclado;
- leitor de tela;
- contraste;
- texto alternativo;
- controles não dependentes somente de arrastar;
- layout responsivo;
- modo compacto e expandido.

## 23. Primeiro recorte implementável

1. Mapa com dois pontos e deslocamento animado.
2. Passagem do tempo oficial.
3. Evento que inicia combate.
4. Mapa tático com pontos para jogador e inimigo.
5. Clique para mover.
6. Ataque básico.
7. Backend resolve até próxima decisão.
8. Fim do combate.
9. Corpo clicável.
10. Transferência de loot.
11. Narração final pelo GPT.

## 24. Critérios de aceitação

- o widget nunca altera sozinho o estado oficial;
- a posição visual corresponde ao snapshot oficial;
- a prévia informa versão e estado usados;
- uma ação repetida não duplica efeitos;
- o combate pode ser retomado após remontagem;
- o jogador pode usar chat e widget na mesma sessão;
- o GPT não precisa narrar cada ataque comum;
- loot reflete consumo e destruição ocorridos no combate.

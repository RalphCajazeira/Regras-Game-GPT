# Roadmap Atual — Extensão e Jogabilidade

**Versão:** `implementation-roadmap-v0.4`  
**Status:** canônico para a ordem atual de entrega

## 1. Objetivo

Orientar a evolução do produto a partir do estado já alcançado:

- backend autoritativo existente;
- OAuth e App MCP em staging;
- `GameSession` persistente;
- projeções de ficha, inventário, equipamentos e habilidades;
- primeira ação autenticada persistente;
- widget funcional como fallback;
- fundação Manifest V3 integrada.

Este roadmap não substitui o estado vivo do repositório executável. Ele define a sequência-alvo de capacidades.

## 2. Princípio

```text
entregar recorte vertical pequeno
→ testar manualmente
→ registrar problemas reais
→ corrigir
→ adicionar próxima capacidade
```

Não esperar o jogo completo para iniciar testes.

## 3. Linha de base concluída

Capacidades consideradas disponíveis como base técnica:

- identidade e autorização do App MCP;
- vínculo `ExternalIdentity → User → Player`;
- membership e controle de ator;
- projeções públicas allowlisted;
- `GameSession` com versão própria;
- seleção persistente de campanha e personagem;
- ficha, inventário, equipamentos e habilidades read-only;
- idempotência e concorrência básica;
- evento oficial de observação;
- pipeline canônico de staging;
- fundação da extensão com overlay e página própria.

O nível exato de validação deve ser consultado em `PROJECT_STATE.md` do repositório executável.

## 4. Fase E1 — Fundação da extensão

Objetivos:

- Manifest V3;
- botão flutuante;
- Shadow DOM;
- overlay/full-page;
- página própria;
- comunicação interna tipada;
- preferências locais;
- acessibilidade;
- build carregável como extensão descompactada.

Critérios:

- não ler chat, cookies ou tokens;
- permissões mínimas;
- shell único reutilizado;
- teste manual no Chromium;
- nenhum backend nesta fase.

## 5. Fase E2 — OAuth da extensão

Objetivos:

- login próprio da extensão;
- mesma identidade interna do App MCP;
- refresh seguro;
- logout;
- revogação;
- armazenamento mínimo de sessão;
- nenhum segredo no manifest.

Critérios:

- não reutilizar cookie do ChatGPT;
- não capturar token da página;
- token expirado recuperável;
- usuário suspenso/revogado falha fechado;
- User A não acessa User B.

## 6. Fase E3 — Leitura real do backend

Objetivos:

- remover fixture local das telas oficiais;
- carregar bootstrap e contexto;
- Resumo;
- Ficha;
- Inventário;
- Equipamentos;
- Habilidades;
- seleção e continuidade;
- estados vazios e erros.

Critérios:

- DTOs públicos compartilhados ou equivalentes;
- paginação;
- sem UUID interno desnecessário;
- sem `MASTER_ONLY`;
- cache apenas efêmero;
- reload reconstrói tudo pelo backend.

## 7. Fase E4 — Atualização automática

Objetivos:

- recarga após mutação;
- polling controlado de fallback;
- canal SSE ou WebSocket autenticado;
- heartbeat e reconexão quando aplicável;
- invalidação por `sessionVersion`;
- atualização das áreas afetadas.

Critérios:

- realtime não transporta estado completo sem necessidade;
- evento perdido não causa divergência permanente;
- reconexão força recarga oficial;
- múltiplas superfícies convergem para o mesmo estado;
- deploy/restart é recuperável.

## 8. Fase E5 — Primeira ação real na extensão

Ação inicial:

`Observar os arredores`

Objetivos:

- executar pela extensão;
- usar a mesma autorização e idempotência do backend;
- atualizar `GameSession`;
- exibir resultado oficial;
- atualizar automaticamente a extensão;
- disponibilizar contexto para o GPT narrar.

Critérios:

- clique duplo não duplica;
- conflito exige reload;
- retry de transporte preserva chave;
- bloqueio de domínio gera nova tentativa deliberada;
- HP, Mana, SP e inventário não mudam indevidamente.

## 9. Fase E6 — Movimento simples

Objetivos:

- representar localização atual;
- listar destinos autorizados;
- prévia de rota;
- confirmar deslocamento;
- persistir localização e tempo quando a regra permitir;
- notificar GPT e extensão.

Fora de escopo inicial:

- mapa tático;
- pathfinding complexo;
- encontros aleatórios avançados.

## 10. Fase E7 — Consumível

Objetivos:

- selecionar item consumível;
- mostrar prévia;
- validar posse e quantidade;
- aplicar efeito;
- atualizar recursos e inventário;
- persistir evento e progressão aplicável.

Critérios:

- exatamente um consumo;
- idempotência;
- conflito recuperável;
- item narrativo não pode ser consumido mecanicamente;
- efeito secreto não vaza.

## 11. Fase E8 — Habilidade simples

Objetivos:

- selecionar habilidade autorizada;
- alvo simples;
- custo;
- resolução oficial;
- atualização de recursos;
- evento e aprendizagem aplicável;
- narração de marco quando necessária.

Começar fora de encontro ou com um cenário controlado.

## 12. Fase E9 — Primeiro encontro jogável

Objetivos:

- um personagem;
- um inimigo;
- estado do encontro;
- janela de decisão;
- movimento simples;
- ataque básico;
- habilidade simples;
- recursos;
- fim do encontro;
- loot mínimo;
- retomada após interrupção.

Critérios:

- motor autoritativo;
- `NEXT_PLAYER_DECISION`;
- idempotência;
- conflito;
- atualização automática;
- recuperação em nova conversa/aba;
- nenhuma decisão do Mestre exposta.

## 13. Fases seguintes

Após o primeiro ciclo jogável:

- comércio;
- equipamentos mutáveis;
- treino;
- progressão por uso;
- Cansaço e sono;
- mapa de exploração;
- combate tático por ticks;
- relações e companheiros;
- missões;
- crafting;
- administração visual;
- criação inicial completa;
- migração final do GPT sem Actions.

## 14. GPT definitivo

Depois de equivalência suficiente:

1. preservar backup da configuração atual;
2. revisar Instructions;
3. revisar Knowledge;
4. remover Actions da configuração do GPT;
5. conectar somente o App MCP;
6. instruir uso da extensão como frontend principal;
7. testar criação, retomada, narrativa e mecânica;
8. manter rollback até validação.

## 15. Organização das tasks

Cada task deve ter:

- um objetivo vertical;
- change set próprio;
- critérios de aceite;
- fora de escopo;
- testes;
- atualização do estado do projeto;
- condição de parada;
- PR próprio quando houver separação técnica real.

Bugs encontrados em teste manual podem virar:

- correção da mesma task, quando causados pelo change set;
- nova task, quando forem independentes ou exigirem commit próprio.

## 16. Regra de prioridade

Priorizar:

1. segurança e isolamento;
2. integridade e idempotência;
3. continuidade;
4. jogabilidade ponta a ponta;
5. atualização visual;
6. amplitude de funcionalidades;
7. refinamento estético.

## 17. Estado vivo

A ordem acima é canônica como direção. O andamento real e as próximas tasks concretas vivem em:

```text
RalphCajazeira/cronicas-de-outro-mundo
docs/ai/project/PROJECT_STATE.md
docs/ai/project/ROADMAP.md
docs/ai/project/DECISIONS.md
```

Quando esses documentos divergirem deste roadmap por decisão aprovada mais recente, atualizar ambos os repositórios.
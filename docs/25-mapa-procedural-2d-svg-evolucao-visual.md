# Mapa Procedural 2D — SVG, Persistência e Evolução Visual

**Versão:** `procedural-map-v0.1`  
**Status:** decisão canônica aprovada para implementação futura

## 1. Objetivo

Definir uma primeira versão de mapa procedural que seja funcional, legível e barata de evoluir antes da adoção de pixel art, tilesets ou engine 2D completa.

O objetivo inicial não é produzir arte semelhante a um jogo pronto. É validar:

- coerência espacial;
- regiões e localidades;
- estradas, rios e pontes;
- cidades, vilas, lotes e estruturas;
- seleção e interação;
- localização e deslocamento;
- persistência e descoberta;
- integração com GPT, extensão e backend.

## 2. Decisão visual inicial

A primeira renderização será feita na `GameApp` React compartilhada usando SVG.

```text
React
→ estado e interação da interface

SVG
→ terreno, zonas, linhas, retângulos, textos e marcadores

Backend
→ geração oficial, validação, persistência e estado
```

Não instalar Phaser apenas para o primeiro mapa.

Phaser, PixiJS ou outra engine passam a ser considerados quando câmera avançada, grande volume de tiles, colisão em tempo real, animação, efeitos ou combate visual justificarem a dependência.

## 3. Representação simplificada

O primeiro mapa pode usar cores, formas e nomes.

Exemplos:

```text
[ Cidade de Arden ]
[ Vila Ribeirinha ]
[ Guilda dos Aventureiros ]
[ Ferreiro de Arden ]
[ Taverna Lua Dourada ]
```

Elementos iniciais:

| Elemento | Representação inicial |
| --- | --- |
| grama | área verde |
| floresta | área verde-escura com rótulo ou padrão |
| água | faixa ou polígono azul |
| estrada | linha ou faixa bege/marrom |
| ponte | faixa marrom cruzando água |
| área urbana | região delimitada |
| casa | retângulo nomeado ou numerado |
| guilda | retângulo com nome e categoria |
| ferreiro | retângulo com nome e categoria |
| taverna | retângulo com nome e categoria |
| templo | retângulo com nome e categoria |
| ruína | forma cinza com estado |
| jogador | marcador com rótulo ou símbolo |
| NPC | marcador com nome ou iniciais |
| inimigo | marcador distinto com rótulo |

A informação nunca deve depender apenas da cor. Texto, símbolo, padrão ou borda devem complementar a codificação visual.

## 4. Três escalas de mapa

### 4.1 Mapa regional

Mostra:

- cidades;
- vilas;
- rios;
- estradas;
- florestas;
- montanhas;
- ruínas;
- dungeons;
- rotas;
- locais descobertos;
- posição aproximada do jogador.

É uma visão estratégica e pode ser mais abstrata.

### 4.2 Mapa local

Mostra:

- ruas;
- lotes;
- casas;
- guildas;
- lojas;
- ferreiros;
- tavernas;
- templos;
- portões;
- pontes;
- NPCs;
- entradas e pontos de interação.

É a primeira escala indicada para o MVP SVG.

### 4.3 Mapa tático

Mostra:

- personagens;
- inimigos;
- obstáculos;
- alcance;
- área de efeito;
- movimento;
- cobertura;
- zonas perigosas.

Essa escala pode permanecer simplificada até o combate visual justificar uma engine dedicada.

## 5. Separação entre modelo e renderização

O modelo espacial não pode depender de SVG, Phaser, sprites ou tilesets.

Entidades conceituais:

```text
WorldMap
MapRegion
MapChunk
Settlement
MapStructure
Route
River
Bridge
PointOfInterest
MapMutation
ExplorationState
```

Exemplo de estrutura:

```ts
interface MapStructure {
  id: string;
  type:
    | 'HOUSE'
    | 'ADVENTURERS_GUILD'
    | 'BLACKSMITH'
    | 'TAVERN'
    | 'SHOP'
    | 'TEMPLE'
    | 'RUIN';
  name: string;
  bounds: {
    x: number;
    y: number;
    width: number;
    height: number;
  };
  entrances: Array<{ x: number; y: number }>;
  status: 'OPEN' | 'CLOSED' | 'LOCKED' | 'DESTROYED';
}
```

Hoje esse objeto pode produzir um retângulo SVG. No futuro, o mesmo contrato pode produzir sprite, tilemap ou prefab sem alterar a regra de domínio.

## 6. Geração procedural determinística

A base deve ser reproduzível por:

```text
worldSeed
+ generatorVersion
+ coordenadas ou identidade da região
+ parâmetros públicos da geração
```

Fluxo inicial:

```text
seed
→ região base
→ água/rios
→ estrada principal
→ ponte quando estrada cruza rio
→ área urbana
→ praça ou eixo central
→ lotes
→ estruturas obrigatórias
→ casas e estruturas secundárias
→ validação de entradas e acessibilidade
```

Regras iniciais de composição:

- guilda próxima de praça, portão ou eixo principal;
- ferreiro próximo de estrada e zona de serviço;
- taverna próxima de praça, portão ou guilda;
- templo em área acessível e reconhecível;
- casas ocupam lotes residenciais restantes;
- estrada não termina em estrutura sem entrada;
- ponte aparece quando rota válida cruza água;
- toda estrutura pública deve possuir caminho acessível.

## 7. Persistência e chunks

O backend continua sendo a autoridade.

```text
seed
→ gera base determinística

banco
→ persiste alterações e descobertas
```

Mudanças persistentes incluem:

- ponte destruída ou reconstruída;
- casa construída;
- loja fechada;
- floresta queimada;
- árvore removida;
- baú saqueado;
- ruína descoberta;
- estrada bloqueada;
- cidade expandida.

Para mapas maiores, dividir em chunks:

```text
worldSeed
generatorVersion
chunkX
chunkY
generatedData ou checksum
persistentMutations
```

O frontend recebe somente regiões e chunks autorizados e necessários à visualização atual.

## 8. Papel do GPT

O GPT define intenção narrativa e significado, não coordenadas arbitrárias.

Exemplo:

```json
{
  "locationType": "VILLAGE",
  "theme": "ribeirinha",
  "importance": "minor",
  "near": "current-player-region"
}
```

O backend:

- procura ou cria local válido;
- escolhe posição coerente;
- aplica regras de geração;
- persiste o resultado;
- publica projeção pública;
- notifica a extensão.

O GPT narra o local confirmado.

## 9. Interação inicial

O primeiro mapa não precisa exigir movimento contínuo.

Fluxo possível:

```text
jogador seleciona Guilda dos Aventureiros
→ painel mostra detalhes
→ jogador escolhe ir até o local
→ backend valida rota, tempo e riscos
→ localização oficial é atualizada
→ mapa e GPT recebem o resultado
```

Movimento célula a célula pode ser introduzido depois, especialmente em mapa tático.

## 10. Etapas de implementação

### M1 — Mock funcional React + SVG

- mapa local pequeno;
- formas e cores;
- estruturas nomeadas;
- seleção por clique;
- painel de detalhes;
- fixture local.

### M2 — Geração determinística

- seed fixa;
- cidade ou vila;
- rio;
- estrada;
- ponte;
- lotes;
- guilda;
- ferreiro;
- taverna;
- casas;
- testes de determinismo e acessibilidade.

### M3 — Movimento e interação

- localização atual;
- seleção de destino;
- prévia de rota;
- confirmação;
- atualização oficial;
- eventos durante deslocamento.

### M4 — Persistência e chunks

- entidades de mapa;
- chunks;
- mutations;
- descoberta;
- reconstrução após reload;
- concorrência e idempotência.

### M5 — Regional, local e tático

- transição entre escalas;
- mapa regional para viagem;
- mapa local para exploração;
- mapa tático para encontro.

### M6 — Evolução visual

```text
SVG
→ ícones
→ sprites simples
→ tilesets
→ engine 2D quando necessária
```

## 11. Critérios para adotar Phaser

Adotar Phaser somente quando houver necessidade comprovada de vários destes pontos:

- câmera e zoom contínuos complexos;
- milhares de tiles visíveis;
- descarte eficiente de elementos fora da viewport;
- animações frequentes;
- colisão em tempo real;
- movimentação contínua;
- efeitos visuais;
- spritesheets;
- combate tático animado;
- áudio espacial ou gerenciamento de cenas.

Até esse ponto, React + SVG reduz dependências e acelera testes de domínio e UX.

## 12. Primeiro recorte de aceite

O primeiro protótipo será aceito quando, com uma seed fixa, produzir:

- mapa pequeno;
- uma cidade ou vila;
- uma estrada;
- um rio;
- uma ponte;
- Guilda dos Aventureiros;
- Ferreiro;
- Taverna;
- algumas casas;
- nomes legíveis;
- seleção por clique;
- ausência de dependência de arte externa;
- mesma renderização disponível no host web e na extensão.

## 13. Fora de escopo inicial

- pixel art final;
- mapa 3D;
- geração continental completa;
- economia urbana completa;
- simulação populacional avançada;
- pathfinding complexo;
- combate tático completo;
- instalação preventiva de Phaser;
- editor visual administrativo completo.

## 14. Regra de evolução

A arte melhora a apresentação, mas não substitui o modelo espacial.

```text
modelo semântico estável
→ primeira renderização simples
→ validação de gameplay
→ refinamento visual incremental
```

Nenhum refinamento visual pode mover autoridade mecânica para o frontend.
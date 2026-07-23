# Sistema de Perícias, Profissões e Passivas

**Versão da proposta:** `skills-v0.2`  
**Status:** em validação

## 1. Objetivo

Representar especialização prática sem multiplicar atributos primários.

```text
Atributos → potencial geral
Perícias → prática ampla
Proficiências → domínio de categoria
Habilidades e magias → técnicas específicas
Profissões → progressão temática e produtiva
Passivas → alterações estruturadas de regras
```

A progressão oficial por uso está definida em:

`docs/19-progressao-niveis-experiencia-treinamento-e-dominio.md`

## 2. Camadas

### Atributo primário

Capacidade universal:

```text
Força
Agilidade
Destreza
Vitalidade
Inteligência
```

### Atributo secundário

Valor derivado diretamente usado em resolução, como Precisão, Esquiva, ATK, DEF ou Velocidade.

### Perícia

Experiência prática em área ampla:

```text
Ferraria
Piromancia
Comércio
Furtividade
Primeiros Socorros
Sobrevivência
```

### Proficiência

Domínio de arma, ferramenta, armadura, material ou categoria:

```text
Adagas
Espadas
Arcos
Cajados
Armaduras Pesadas
Ferramentas de Ferreiro
Aço Élfico
```

### Habilidade ativa

Ação declarada e definida pelo sistema de ações.

### Habilidade passiva

Efeito automático, modificador ou gatilho estruturado.

### Profissão

Conjunto organizado de perícias, proficiências, receitas, passivas, especializações e marcos.

## 3. Perícias

Estrutura:

```json
{
  "code": "blacksmithing",
  "name": "Ferraria",
  "level": 8,
  "experience": 430,
  "experienceToNextLevel": 600,
  "attributeContributions": [
    { "attribute": "strength", "weight": 0.25 },
    { "attribute": "dexterity", "weight": 0.35 },
    { "attribute": "intelligence", "weight": 0.20 }
  ],
  "tags": ["CRAFTING", "METALWORK"],
  "ruleVersion": "skills-v0.2"
}
```

Pontuação efetiva:

```text
contribuição do nível da perícia
+ atributos
+ proficiências
+ ferramentas
+ ambiente
+ passivas
+ condições
+ modificadores temporários
```

Pesos e fórmulas são versionados.

## 4. Exemplos de perícias

### Produção e coleta

- Ferraria;
- Alquimia;
- Encantamento;
- Culinária;
- Mineração;
- Herbalismo;
- Curtimento;
- Carpintaria.

### Combate e magia

- Piromancia;
- Criomancia;
- Cura;
- Combate Desarmado;
- Tática;
- Controle Mágico.

### Exploração e suporte

- Furtividade;
- Sobrevivência;
- Investigação;
- Primeiros Socorros;
- Rastreamento;
- Armadilhas.

### Sociais e comerciais

- Comércio;
- Persuasão;
- Enganação;
- Intimidação;
- Etiqueta;
- Avaliação de Itens.

## 5. Proficiências

Uma proficiência possui progressão própria quando representar prática recorrente relevante.

```json
{
  "code": "daggers",
  "name": "Adagas",
  "level": 18,
  "experience": 320,
  "experienceToNextLevel": 410,
  "tags": ["WEAPON", "FINESSE"]
}
```

Ataques básicos universais normalmente evoluem a proficiência da categoria, não uma barra separada para cada golpe comum.

Habilidades nomeadas e magias específicas podem possuir domínio individual além da perícia e proficiência.

## 6. Profissões

```json
{
  "code": "blacksmith",
  "name": "Ferreiro",
  "level": 6,
  "experience": 820,
  "skillCodes": ["blacksmithing", "metal-evaluation"],
  "proficiencyCodes": ["smith-hammer", "ironworking"],
  "knownRecipeCodes": ["iron-dagger", "iron-short-sword"],
  "passiveCodes": ["material-efficiency-1", "quality-control-1"]
}
```

A profissão não substitui as perícias.

Ela organiza:

- identidade profissional;
- receitas;
- materiais;
- ferramentas;
- passivas;
- especializações;
- marcos;
- requisitos;
- possibilidade de ensinar.

Exemplos:

- Ferreiro;
- Alquimista;
- Encantador;
- Cozinheiro;
- Curandeiro;
- Caçador;
- Mercador.

## 7. Passivas

Passivas usam efeitos estruturados e versionados.

```json
{
  "code": "merchant-discount-1",
  "name": "Desconto I",
  "type": "PASSIVE",
  "effects": [
    {
      "target": "purchasePriceModifierPercent",
      "operation": "ADD",
      "value": -5
    }
  ]
}
```

Outro exemplo:

```json
{
  "code": "better-resale-1",
  "name": "Boa Revenda I",
  "type": "PASSIVE",
  "effects": [
    {
      "target": "salePriceModifierPercent",
      "operation": "ADD",
      "value": 10
    }
  ]
}
```

Passivas não alteram silenciosamente os valores-base persistidos das definições.

## 8. Acúmulo e limites

Modificadores são consolidados por categoria:

```text
passiva
+ equipamento
+ reputação
+ relação
+ ambiente
+ condição
= modificador agregado
```

Depois o backend aplica:

- limites;
- incompatibilidades;
- prioridade;
- regra de não acumulação;
- versão vigente.

## 9. Progressão pelo uso

Toda tentativa mecanicamente executada pode gerar XP.

```text
uso válido
→ backend confirma execução, custos e consequências
→ gera eventos de progressão
→ concede XP às trilhas aplicáveis
→ verifica níveis e marcos
→ persiste atomicamente
```

### Contextos válidos

- combate real;
- treino livre;
- árvore ou alvo estático;
- boneco de treino;
- sparring;
- estudo;
- meditação;
- atividade profissional;
- tentativa criativa resolvida pelo backend.

### Regras

- falha válida pode ensinar;
- ação interrompida pode gerar XP parcial;
- comando rejeitado antes de executar não gera XP;
- repetição válida não recebe zero apenas por ser repetição;
- Cansaço, dificuldade, contexto e resultado ajustam a quantidade;
- custos e tempo permanecem reais;
- backend calcula XP oficial;
- idempotência impede duplicação.

## 10. Ataque, defesa e resistência

### Ataque

Pode treinar:

- ação ou habilidade;
- proficiência da arma;
- perícia de combate;
- atributos declarados.

### Esquiva

Exige tentativa real de esquiva.

Ser atingido sem tentar esquivar não concede XP de Esquiva.

### Bloqueio e aparo

Exigem tentativa correspondente.

### Dano recebido

Pode treinar:

- Vitalidade;
- tolerância à dor;
- resiliência;
- proficiência de armadura quando houve absorção aplicável.

Também pode causar incapacidade, ferimento ou morte.

## 11. Domínio individual

Habilidades e magias nomeadas possuem domínio por ator:

```text
actor
+ contentDefinition
+ contentVersion
+ masteryLevel
+ masteryXp
+ statistics
+ selectedSpecializations
```

A definição declara quais parâmetros melhoram e em quais marcos.

## 12. Sessões em lote

Treinos longos reutilizam os serviços gerais, sem chamada por repetição.

Exemplos:

```text
TRAIN_ACTION_REPETITION
TRAIN_FOR_DURATION
TRAIN_UNTIL_RESOURCE_LIMIT
SPARRING_SESSION
STUDY_SESSION
MEDITATION_SESSION
```

O backend resolve cada uso necessário, aplica condições de parada e devolve um resumo consolidado.

## 13. Widget

O widget mostra:

- nível;
- XP;
- domínio;
- marcos;
- contribuições de atributos;
- efeitos aplicáveis;
- custo e duração de treino;
- Cansaço previsto;
- resumo confirmado.

O widget não calcula XP autoritativa.

## 14. Responsabilidades

### GPT

- interpretar a atividade;
- escolher perícia e técnica coerentes;
- estruturar treino criativo;
- narrar resultados e marcos;
- não conceder XP ou bônus diretamente.

### Backend

- calcular pontuação efetiva;
- resolver ação ou sessão;
- aplicar custos;
- criar eventos de progressão;
- calcular XP;
- validar efeitos e acúmulos;
- aplicar níveis, marcos e desbloqueios;
- persistir atomicamente.

## 15. Regra central

```text
Poucos atributos universais.
Especialização por perícias, proficiências, domínio, profissões e passivas.
Toda prática executada pode ensinar.
Backend resolve e persiste.
GPT interpreta e narra.
Widget apresenta e coleta comandos.
```

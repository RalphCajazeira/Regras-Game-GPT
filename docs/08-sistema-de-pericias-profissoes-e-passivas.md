# Sistema de Perícias, Profissões e Passivas

**Versão da proposta:** `skills-v0.1`  
**Status:** em validação

## 1. Objetivo

Manter poucos atributos primários universais e representar especializações por perícias, proficiências, profissões e habilidades passivas.

Os atributos indicam o potencial geral do ator. As perícias indicam o que ele aprendeu a fazer. As passivas modificam regras existentes sem criar novos atributos primários.

## 2. Camadas do domínio

```text
Atributos primários
→ atributos secundários
→ perícias e proficiências
→ habilidades ativas e passivas
→ profissões e receitas
→ resolução validada pelo backend
```

### 2.1 Atributo primário

Capacidade universal: Força, Agilidade, Destreza, Vitalidade ou Inteligência.

### 2.2 Atributo secundário

Valor derivado usado diretamente nas regras, como Precisão, Esquiva, ATK, DEF ou Velocidade.

### 2.3 Perícia

Experiência prática progressiva em uma atividade, como Ferraria, Comércio, Alquimia ou Primeiros Socorros.

### 2.4 Proficiência

Domínio de uma ferramenta, arma ou categoria específica, como Adagas, Arcos, Espadas, Martelos de Ferreiro ou Armaduras Pesadas.

### 2.5 Habilidade ativa

Ação declarada pelo ator e definida no sistema de ações.

### 2.6 Habilidade passiva

Modificador ou gatilho automático que altera uma regra existente sem consumir uma ação normal, salvo quando sua definição declarar custo ou ativação.

### 2.7 Profissão

Conjunto organizado de perícias, proficiências, passivas, receitas e progressão temática.

## 3. Perícias

Estrutura recomendada:

```json
{
  "code": "blacksmithing",
  "name": "Ferraria",
  "level": 8,
  "experience": 430,
  "experienceToNextLevel": 600,
  "unspentSkillPoints": 0,
  "attributeContributions": [
    { "attribute": "strength", "weight": 0.25 },
    { "attribute": "dexterity", "weight": 0.35 },
    { "attribute": "intelligence", "weight": 0.20 }
  ],
  "tags": ["CRAFTING", "METALWORK"]
}
```

O backend calcula e retorna uma pontuação efetiva:

```text
Pontuação Efetiva =
contribuição do nível da perícia
+ contribuições dos atributos
+ proficiências
+ ferramentas
+ ambiente ou oficina
+ passivas
+ modificadores temporários
```

Os pesos e fórmulas são versionados por perícia. O GPT não escolhe pesos depois de conhecer a rolagem.

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

### Combate e uso de equipamentos

- Adagas;
- Espadas;
- Arcos;
- Cajados;
- Escudos;
- Armaduras Pesadas;
- Combate Desarmado.

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

## 5. Profissões

Uma profissão agrupa progressões relacionadas.

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

A profissão não substitui as perícias. Ela organiza desbloqueios, receitas, passivas e identidade de progressão.

Exemplos:

- Ferreiro;
- Alquimista;
- Encantador;
- Cozinheiro;
- Curandeiro;
- Caçador;
- Mercador.

## 6. Habilidades passivas

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

Essas passivas modificam a transação atual. Elas não alteram `baseBuyPrice` ou `baseSellPrice` da definição do item.

## 7. Acúmulo e limites

Modificadores da mesma categoria são consolidados antes da aplicação:

```text
modificador da passiva
+ reputação
+ relação
+ negociação
= modificador agregado
```

Depois o backend aplica:

- limite da categoria;
- faixa mínima e máxima do comerciante;
- incompatibilidades;
- prioridade de efeitos;
- regra de não acumulação, quando existir.

Uma passiva nunca autoriza o GPT a ultrapassar limites persistidos.

## 8. Progressão pelo uso

Uma perícia pode ganhar experiência por tentativa válida ou resultado relevante.

```text
uso válido
→ backend confirma requisitos e consumo
→ concede XP da perícia
→ verifica aumento de nível
→ aplica desbloqueios
```

Regras contra exploração:

- atividades triviais concedem XP reduzido ou zero;
- cancelar antes da resolução não concede XP;
- tentativas precisam respeitar custos reais;
- falhas deliberadas não devem ser treinamento eficiente;
- repetições podem sofrer retorno decrescente;
- o backend calcula o XP autoritativo.

## 9. Resolução de perícia

Uma resolução pode ser direta no backend ou local pelo GPT.

### 9.1 Resolução direta

O GPT envia a intenção e o backend calcula, rola, persiste e retorna o resultado.

Adequada para operações curtas e altamente autoritativas.

### 9.2 Sessão por snapshot

O backend retorna:

```json
{
  "skillSessionId": "skill-session-id",
  "stateVersion": 15,
  "ruleVersion": "skills-v0.1",
  "actor": {},
  "skill": {
    "code": "blacksmithing",
    "level": 8,
    "effectiveScore": 72
  },
  "availableActions": [],
  "constraints": {},
  "resolutionPolicy": {}
}
```

O GPT pode conduzir escolhas, rolagens e narrativa localmente enquanto o estado-base permanecer válido. Ao final, envia um log estruturado para reprodução e persistência.

## 10. Responsabilidades

### GPT

- escolher a perícia compatível com a intenção;
- usar somente parâmetros carregados;
- conduzir escolhas e narrativa;
- registrar modificadores e rolagens;
- não inventar níveis, passivas, receitas ou bônus;
- enviar resolução consolidada quando houver alteração persistente.

### Backend

- persistir níveis, XP, desbloqueios e passivas;
- calcular pontuações efetivas;
- validar efeitos e acúmulos;
- controlar custos, limites e versões;
- conceder XP autoritativo;
- retornar erros acionáveis;
- aplicar alterações persistentes atomicamente.

## 11. Regra central

```text
Poucos atributos universais.
Especialização por perícias, proficiências, profissões e passivas.
GPT conduz dentro do snapshot.
Backend calcula, valida e persiste consequências permanentes.
```

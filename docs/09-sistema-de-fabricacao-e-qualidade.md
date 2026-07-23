# Sistema de Fabricação e Qualidade

**Versão da proposta:** `crafting-v0.1`  
**Status:** em validação

## 1. Objetivo

Definir como personagens fabricam, reparam, melhoram, desmontam e encantam itens usando atributos, perícias, profissões, receitas, materiais, ferramentas, oficinas, passivas e rolagens.

A fabricação deve permitir progressão pelo uso sem transformar materiais comuns em itens excepcionais sem requisitos compatíveis.

## 2. Princípios

1. Toda fabricação parte de uma receita ou procedimento conhecido.
2. Atributos indicam potencial; perícias indicam treinamento.
3. Receita, material, ferramenta e oficina limitam a qualidade máxima.
4. A perícia melhora sucesso e qualidade dentro desse limite.
5. Toda tentativa persistente consome ou reserva recursos reais.
6. O backend é autoridade sobre consumo, criação, qualidade e XP.
7. O GPT pode conduzir uma sessão local quando recebe todos os parâmetros.
8. Repetições triviais concedem pouco ou nenhum XP.
9. Qualidade do item e condição atual do item são conceitos diferentes.
10. Fórmulas e limiares são versionados.

## 3. Operações

```text
CRAFT
REPAIR
UPGRADE
ENCHANT
DISMANTLE
REFINE
```

Cada operação possui sua própria receita, custos, riscos e resultados permitidos.

## 4. Receita

```json
{
  "code": "iron-long-sword",
  "name": "Espada Longa de Ferro",
  "operation": "CRAFT",
  "requiredProfession": "blacksmith",
  "requiredSkill": "blacksmithing",
  "minimumSkillLevel": 5,
  "itemLevel": 5,
  "difficulty": 60,
  "baseTimeTicks": 6000,
  "materials": [],
  "requiredToolTags": ["SMITH_HAMMER"],
  "requiredWorkspaceTags": ["FORGE"],
  "allowedQualities": ["INFERIOR", "COMMON", "RARE"],
  "maximumQuality": "RARE"
}
```

A receita deve declarar antes da tentativa:

- resultado possível;
- nível e dificuldade;
- perícia e profissão requeridas;
- materiais e quantidades;
- ferramentas;
- oficina ou ambiente;
- tempo;
- qualidades permitidas;
- efeitos especiais permitidos;
- política de falha.

## 5. Fatores da fabricação

A resolução pode considerar:

```text
nível da perícia
+ atributos relevantes
+ proficiências
+ qualidade dos materiais
+ ferramentas
+ oficina
+ passivas
+ condições
+ dificuldade da receita
+ rolagem
```

Exemplo de Ferraria:

- Força ajuda a trabalhar materiais resistentes;
- Destreza ajuda na precisão e acabamento;
- Inteligência ajuda em temperatura, ligas, projeto e técnica;
- Ferraria representa experiência prática;
- ferramentas e oficina ampliam o teto e a consistência.

Os pesos exatos pertencem à definição versionada da perícia ou receita.

## 6. Pontuação efetiva

Modelo conceitual:

```text
Pontuação de Fabricação =
pontuação efetiva da perícia
+ bônus da receita conhecida
+ ferramenta
+ oficina
+ materiais
+ passivas
+ modificadores situacionais
```

A dificuldade é comparada a essa pontuação para formar:

- chance de sucesso;
- risco de perda;
- faixa de qualidade;
- tempo final;
- experiência possível.

A fórmula numérica definitiva permanece pendente de simulações.

## 7. Limite de qualidade

A qualidade final nunca pode ultrapassar o menor teto aplicável entre:

```text
receita
material principal
ferramenta
oficina
conhecimento ou desbloqueio
regra da operação
```

Exemplos:

```text
Receita simples + ferro comum → máximo Raro
Receita avançada + material raro → máximo Épico
Receita lendária + material lendário + oficina compatível → pode alcançar Lendário
```

Um personagem muito habilidoso aumenta a chance de alcançar o teto disponível, mas não ignora esse teto.

## 8. Qualidades

O sistema usa as qualidades canônicas:

```text
INFERIOR
COMMON
RARE
EPIC
LEGENDARY
```

A qualidade determina o orçamento de poder do item conforme o sistema de equipamentos.

A fabricação não distribui atributos livremente depois da rolagem. A receita, especialização, materiais e escolhas declaradas antes da resolução definem quais distribuições são válidas.

## 9. Limiar de qualidade

Exemplo de resposta carregada:

```json
{
  "successChance": 86,
  "qualityThresholds": {
    "INFERIOR": 1,
    "COMMON": 30,
    "RARE": 70,
    "EPIC": 94
  },
  "maximumQuality": "EPIC"
}
```

A rolagem e a política da receita determinam o resultado. Os limiares devem ser conhecidos antes da rolagem.

Possíveis resultados:

- falha sem item;
- falha com subproduto;
- item Inferior;
- item Comum;
- item Raro;
- item Épico;
- item Lendário, quando permitido.

## 10. Falha e consumo

A receita declara a política de falha:

```json
{
  "failurePolicy": {
    "consumeMaterialsPercent": 50,
    "damageTools": true,
    "createScrap": true,
    "grantReducedSkillXp": true
  }
}
```

O GPT não decide livremente quais materiais foram preservados. O backend valida o resultado conforme a política carregada.

## 11. Tempo

Atividades usam tempo persistente do mundo.

```text
Tempo Final =
tempo base da receita
modificado por perícia, ferramenta, oficina, passivas e complexidade
```

Velocidades de combate não reduzem automaticamente horas de trabalho artesanal. Uma passiva ou regra de profissão deve declarar esse efeito explicitamente.

## 12. Experiência da perícia

O backend calcula XP considerando:

- dificuldade relativa;
- operação concluída;
- resultado;
- novidade;
- repetição;
- recursos consumidos;
- diferença entre nível da perícia e receita.

Proposta conceitual:

```text
receita próxima ou acima da perícia → XP normal ou elevado
receita muito abaixo → XP reduzido
receita trivial → 0 XP
```

Falhas válidas podem conceder XP reduzido. Cancelamentos e tentativas inválidas não concedem XP.

## 13. Sessão de fabricação

```json
{
  "craftSessionId": "craft-session-id",
  "stateVersion": 15,
  "ruleVersions": {
    "attributes": "attributes-v0.4",
    "equipment": "equipment-v0.4",
    "skills": "skills-v0.1",
    "crafting": "crafting-v0.1"
  },
  "actor": {},
  "profession": {},
  "skill": {
    "code": "blacksmithing",
    "level": 8,
    "effectiveScore": 72
  },
  "recipe": {},
  "materials": [],
  "tools": [],
  "workspace": {},
  "resolution": {
    "successChance": 86,
    "qualityThresholds": {},
    "maximumQuality": "EPIC"
  }
}
```

O GPT pode conduzir preparação, escolhas permitidas, rolagens e narrativa. O backend confirma o consumo e cria o item autoritativamente.

## 14. Resultado consolidado

```json
{
  "craftSessionId": "craft-session-id",
  "baseStateVersion": 15,
  "recipeCode": "iron-long-sword",
  "declaredOptions": {},
  "rolls": [],
  "reportedResult": {
    "success": true,
    "quality": "RARE"
  }
}
```

O backend:

1. confere a versão;
2. recalcula pontuações e limites;
3. valida materiais, ferramentas e oficina;
4. valida as rolagens;
5. consome recursos;
6. cria ou altera o item;
7. concede XP;
8. registra histórico;
9. retorna o resultado autoritativo.

## 15. Integração com economia

Fabricação pode gerar valor, mas o preço-base continua sendo calculado pelas regras econômicas e de equipamento.

Custos relevantes:

- materiais;
- combustível;
- aluguel de oficina;
- desgaste de ferramentas;
- taxas;
- serviços auxiliares;
- tempo do personagem.

Itens fabricados não possuem garantia de venda. Comerciantes continuam limitados por especialidade, demanda, estoque e saldo.

## 16. Pendências de validação

- fórmula definitiva de sucesso;
- pesos de atributos por profissão;
- limiares de qualidade;
- consumo em falhas;
- durabilidade de ferramentas;
- tempo de produção;
- reparos e conservação;
- encantamento e efeitos especiais;
- desmontagem e subprodutos;
- XP e retorno decrescente;
- fabricação em lote.

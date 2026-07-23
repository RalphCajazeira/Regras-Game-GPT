# Sistema de Fabricação e Qualidade

**Versão da proposta:** `crafting-v0.2`  
**Status:** em validação

## 1. Objetivo

Definir como personagens fabricam, reparam, melhoram, desmontam, refinam e encantam itens usando definições existentes, receitas, materiais, ferramentas, oficinas, atributos, perícias, profissões, passivas e rolagens.

A fabricação cria ou altera **instâncias**. Ela não cria uma nova definição toda vez que a qualidade muda.

## 2. Princípios

1. Toda fabricação parte de uma receita ou procedimento conhecido.
2. A receita referencia uma definição de item ou variante já existente.
3. Qualidade pertence à instância produzida.
4. A referência de poder e preço da definição é a qualidade Comum.
5. Receita, material, ferramenta, oficina e desbloqueios limitam a qualidade máxima.
6. Perícia e atributos melhoram sucesso e qualidade dentro do teto permitido.
7. Toda tentativa persistente consome ou reserva recursos reais.
8. O backend é autoridade sobre consumo, criação, qualidade, valores resolvidos e XP.
9. O GPT pode conduzir a sessão localmente usando parâmetros carregados.
10. Repetições triviais concedem pouco ou nenhum XP.
11. Valores resolvidos ficam congelados na instância criada.

## 3. Operações

```text
CRAFT
REPAIR
UPGRADE
ENCHANT
DISMANTLE
REFINE
```

## 4. Receita

```json
{
  "code": "craft-elven-dagger",
  "version": 1,
  "name": "Forjar Adaga Élfica",
  "operation": "CRAFT",
  "outputDefinitionCode": "elven-dagger",
  "outputDefinitionVersion": 1,
  "outputVariantCode": "BASE",
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
  "maximumQuality": "RARE",
  "failurePolicy": {}
}
```

A receita declara antes da tentativa:

- definição e variante produzidas;
- nível e dificuldade;
- profissão e perícia requeridas;
- materiais e quantidades;
- ferramentas e oficina;
- tempo;
- qualidades permitidas;
- teto de qualidade;
- opções de distribuição permitidas;
- efeitos especiais possíveis;
- política de falha.

## 5. Definição versus instância

Fluxo correto:

```text
Receita: Forjar Adaga Élfica
→ definição existente: elven-dagger
→ tentativa de fabricação
→ sucesso ou falha
→ qualidade resolvida
→ nova instância criada
```

Cinco tentativas podem resultar em:

```text
1. Falha
2. Adaga Élfica Comum
3. Adaga Élfica Rara
4. Falha
5. Adaga Élfica Comum
```

Persistência:

```text
1 definição: elven-dagger
3 instâncias:
- item-001 COMMON
- item-002 RARE
- item-003 COMMON
```

O GPT e o backend não recriam a definição `elven-dagger` em cada tentativa.

## 6. Fatores da fabricação

A resolução pode considerar:

```text
nível da perícia
+ atributos relevantes
+ proficiências
+ conhecimento da receita
+ qualidade dos materiais
+ ferramentas
+ oficina
+ passivas
+ condições
- dificuldade
+ rolagem
```

Exemplo de Ferraria:

- Força ajuda a trabalhar materiais resistentes;
- Destreza ajuda na precisão e acabamento;
- Inteligência ajuda em projeto, ligas, temperatura e técnica;
- Ferraria representa experiência prática;
- ferramentas e oficina ampliam consistência e teto.

Os pesos são versionados e calculados pelo backend.

## 7. Pontuação efetiva

Modelo conceitual:

```text
Pontuação de Fabricação =
pontuação efetiva da perícia
+ receita conhecida
+ ferramenta
+ oficina
+ materiais
+ passivas
+ modificadores situacionais
```

A comparação com a dificuldade produz:

- chance de sucesso;
- risco de consumo ou perda;
- chances e limiares de qualidade;
- tempo final;
- XP possível.

## 8. Limite de qualidade

A qualidade final não pode ultrapassar o menor teto aplicável entre:

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

Perícia elevada melhora a chance de alcançar o teto, mas não o ignora.

## 9. Resolução de qualidade

```json
{
  "successChance": 86,
  "qualityThresholds": {
    "INFERIOR": 1,
    "COMMON": 30,
    "RARE": 70
  },
  "maximumQuality": "RARE"
}
```

Os limiares devem ser conhecidos antes da rolagem.

Possíveis resultados:

- falha sem item;
- falha com subproduto;
- item Inferior;
- item Comum;
- item Raro;
- item Épico;
- item Lendário, quando permitido.

## 10. Criação da instância

Quando houver sucesso, o backend:

1. localiza a definição e variante;
2. confirma a versão usada;
3. resolve a qualidade;
4. calcula o orçamento da instância;
5. distribui o orçamento conforme o perfil da definição e escolhas permitidas;
6. calcula modificadores finais;
7. calcula preços-base resolvidos;
8. resolve ações ou passivas escaláveis;
9. cria a instância;
10. consome materiais e aplica desgaste;
11. concede XP;
12. registra proveniência e histórico.

Exemplo:

```json
{
  "definitionCode": "elven-dagger",
  "definitionVersion": 1,
  "quality": "RARE",
  "resolvedPowerBudget": {
    "referenceCommonPoints": 10,
    "qualityMultiplier": 1.4,
    "maximumPoints": 14,
    "usedPoints": 14
  },
  "resolvedModifiers": {
    "physicalAttack": 11,
    "physicalAccuracy": 3
  },
  "craftedByActorId": "player-1"
}
```

## 11. Distribuição de poder

A qualidade não libera distribuição arbitrária depois da rolagem.

A distribuição é limitada por:

- perfil da definição;
- variante;
- receita;
- especialização do fabricante;
- materiais escolhidos;
- opções declaradas antes da resolução;
- orçamento final.

Uma Adaga Élfica não recebe bônus aleatório de Mana ou Defesa Mágica sem regra ou identidade compatível.

## 12. Variante durante a fabricação

Uma receita pode produzir:

- a definição-base;
- uma variante preexistente;
- uma nova variante proposta e validada quando houver diferença real.

Não é variante:

- qualidade diferente;
- condição diferente;
- fabricante diferente;
- nome personalizado sem efeito mecânico.

Pode ser variante:

- material estrutural diferente;
- elemento diferente;
- ação ou passiva diferente;
- distribuição de poder diferente;
- receita e função próprias.

## 13. Falha e consumo

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

O GPT não decide livremente quais materiais foram preservados. O backend valida conforme a política carregada.

## 14. Tempo

```text
Tempo Final =
tempo base da receita
modificado por perícia, ferramenta, oficina, passivas e complexidade
```

Velocidades de combate não reduzem automaticamente horas de trabalho artesanal.

## 15. Experiência

O backend calcula XP considerando:

- dificuldade relativa;
- operação concluída;
- resultado;
- novidade;
- repetição;
- recursos consumidos;
- diferença entre perícia e receita.

```text
Receita equivalente ou difícil → XP normal ou elevado
Receita muito inferior → XP reduzido
Receita trivial → 0 XP
```

## 16. Sessão de fabricação

```json
{
  "craftSessionId": "craft-session-id",
  "stateVersion": 15,
  "ruleVersions": {
    "equipment": "equipment-v0.5",
    "inventory": "inventory-v0.2",
    "skills": "skills-v0.1",
    "crafting": "crafting-v0.2"
  },
  "actor": {},
  "profession": {},
  "skill": {},
  "recipe": {},
  "outputDefinition": {},
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

O GPT conduz escolhas permitidas, rolagens e narrativa. O backend confirma o resultado e cria a instância autoritativamente.

## 17. Resultado consolidado

```json
{
  "craftSessionId": "craft-session-id",
  "baseStateVersion": 15,
  "recipeCode": "craft-elven-dagger",
  "declaredOptions": {},
  "rolls": [],
  "reportedResult": {
    "success": true,
    "quality": "RARE"
  }
}
```

## 18. Congelamento e revisão

A instância criada grava:

- definição e versão;
- qualidade;
- orçamento resolvido;
- modificadores resolvidos;
- preços-base resolvidos;
- versões das regras;
- fabricante;
- semente e proveniência.

Mudanças futuras em multiplicadores ou definição não alteram silenciosamente a instância. Migração é explícita.

O GPT pode revisar definição, variante, receita ou instância quando houver motivo coerente, usando escopo e versionamento.

## 19. Integração econômica

Fabricação transforma recursos e tempo em instâncias. Não cria dinheiro diretamente.

Custos possíveis:

- materiais;
- combustível;
- desgaste de ferramentas;
- oficina;
- taxas;
- serviços auxiliares;
- tempo.

A venda usa os preços resolvidos da instância e as regras de mercado.

## 20. Pendências

- fórmula definitiva de sucesso;
- limiares de qualidade;
- arredondamento do orçamento;
- perfis de distribuição;
- consumo em falhas;
- durabilidade de ferramentas;
- tempo de produção;
- reparos e conservação;
- encantamento e efeitos especiais;
- desmontagem e subprodutos;
- XP e retorno decrescente;
- fabricação em lote;
- migração de instâncias após revisão.

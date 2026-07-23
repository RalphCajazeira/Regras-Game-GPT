# Decisões e Pendências

Este arquivo separa regras aceitas de propostas ainda sujeitas a testes.

## 1. Arquitetura

- O GPT conduz interações usando snapshots versionados.
- O GPT pode criar, corrigir, balancear e evoluir conteúdos por Actions válidas.
- O backend calcula, valida, persiste e aplica consequências autoritativas.
- Validação não significa imutabilidade.
- O frontend futuro usa o mesmo domínio e validações.
- Recursos ausentes não podem ser acrescentados retroativamente a uma sessão em andamento.

## 2. Criação, revisão e versionamento

- Todo conteúdo pode ser revisado com justificativa coerente.
- Sugestões do jogador são avaliadas, não aplicadas automaticamente.
- O GPT consulta identidade, orçamento, vínculos e impacto.
- Revisões preferem manter ID e código e criar nova versão.
- Toda revisão declara escopo.
- Eventos históricos não são recalculados automaticamente.
- O backend retorna erros acionáveis.

Escopos:

```text
DEFINITION_FUTURE_INSTANCES
CURRENT_INSTANCE
SELECTED_INSTANCES
ALL_COMPATIBLE_INSTANCES
```

## 3. Atributos

```text
Força
Agilidade
Destreza
Vitalidade
Inteligência
```

- Agilidade representa rapidez e mobilidade.
- Destreza representa precisão e controle.
- Cada nível concede 10 pontos primários.
- Pontos podem permanecer não distribuídos.
- Especializações usam secundários, perícias, proficiências, profissões e passivas.

## 4. Atores

- Ator é o modelo universal para pessoas, animais, monstros, criaturas, espíritos, construtos, invocações e jogadores.
- Natureza, espécie, arquétipo, facção, papel e controle são separados.
- Entidades irrelevantes podem existir como avistamentos.
- Antes de interação mecânica relevante, o ator é materializado com ficha completa.
- Dados materializados ficam congelados no snapshot.
- O GPT não inventa retroativamente ações, itens, dinheiro ou drops.
- Instâncias podem ser promovidas sem duplicação.
- Companheiro é papel e vínculo, não espécie.
- Materialização usa semente determinística.

## 5. Definições, variantes, qualidades e instâncias de item

### Regra consolidada

```text
Definição única com perfil Comum
→ variante somente quando houver diferença real
→ qualidade na instância
→ valores finais resolvidos e congelados
```

- Qualidade não cria novo cadastro.
- `elven-dagger` representa a Adaga Élfica em qualquer qualidade.
- A definição usa `COMMON` como referência de poder e preço.
- Cada unidade criada, encontrada ou comprada é uma instância.
- A instância guarda qualidade, orçamento, modificadores e preços resolvidos.
- Mudanças futuras nas regras não alteram silenciosamente instâncias existentes.
- Migração é explícita.

Não cria nova definição:

- qualidade diferente;
- condição ou durabilidade;
- fabricante ou proprietário;
- estado de roubo;
- nome personalizado sem efeito mecânico.

Pode criar variante ou definição:

- material estrutural diferente;
- elemento diferente;
- distribuição de poder diferente;
- ações ou passivas diferentes;
- receita ou função mecânica própria.

Antes de criar conteúdo, o backend deve retornar um dos resultados:

```text
USE_EXISTING_DEFINITION
USE_EXISTING_VARIANT
CREATE_NEW_VARIANT
CREATE_NEW_DEFINITION
REVIEW_REQUIRED
```

Mudança somente de qualidade sempre reutiliza definição ou variante existente.

## 6. Equipamentos

- Todo equipamento declara todos os modificadores, inclusive zeros.
- `primaryModifiers` inclui cinco atributos.
- Corpo e Peito são slots diferentes.
- Armas podem ocupar uma ou duas mãos.
- Qualidades: Inferior, Comum, Raro, Épico e Lendário.
- Nível e slot formam o orçamento Comum de referência.
- Qualidade multiplica o orçamento da instância.
- Distribuição adicional segue perfil da definição.
- Penalidades recuperam orçamento positivo dentro de limites.
- Equipamentos podem conceder ações, passivas e proficiências.
- Qualidade não altera automaticamente peso, slots, material ou empunhadura.
- Passivas só escalam quando houver perfil explícito.

## 7. Inventário, pilhas e saque

- Definição, variante, instância e pilha são diferentes.
- Todo item persistente possui localização e propriedade.
- A mesma unidade não pode estar em dois locais ou operações.
- Pilhas exigem compatibilidade de definição, variante, qualidade, condição e demais estados.
- Equipamentos com histórico ou durabilidade individual normalmente permanecem separados.
- O frontend pode agrupar visualmente instâncias iguais.
- Consumíveis, munições e cargas são removidos conforme a ação.
- Itens consumidos não reaparecem no saque.
- Itens carregados, drops naturais e recompensas externas são fontes distintas.
- Drops aleatórios usam semente persistida.

## 8. Fabricação

- Toda fabricação usa receita conhecida.
- A receita referencia definição e variante de saída.
- Cada sucesso cria uma nova instância.
- Cinco tentativas podem gerar várias instâncias de qualidades diferentes sem duplicar a definição.
- Receita, material, ferramenta, oficina e desbloqueios limitam a qualidade máxima.
- Perícia melhora sucesso e qualidade dentro do teto.
- O backend resolve qualidade, orçamento, modificadores e preço-base da instância.
- Consumo, criação, reparo, qualidade e XP são autoritativos.
- Valores resolvidos ficam congelados.

## 9. Economia

- A moeda é Coroa, código `CROWN`.
- Valores são inteiros.
- A definição guarda preço Comum de referência.
- A instância guarda preço-base resolvido por qualidade.
- Mercado e negociação definem o preço final.
- Passivas comerciais não alteram preços-base.
- O GPT negocia somente dentro das faixas.
- O backend controla saldo, estoque, propriedade e atomicidade.

## 10. Combate e ações

- Posições e alcances usam metros.
- O tempo usa linha contínua em ticks.
- Existe uma única rolagem de acerto.
- Precisão é comparada com Esquiva ou Resistência.
- Chance de acerto fica entre 1% e 99%.
- Após acerto entram crítico, Defesa Crítica e mitigação.
- Ações possuem preparação, recuperação e tempo mínimo.
- Movimento e ataque são separados, salvo ação composta.
- Ações podem ser interrompidas.
- Munições, consumíveis e equipamentos usados precisam existir no snapshot.

## 11. Perícias, profissões e passivas

- Atributos representam potencial geral.
- Perícias representam treinamento.
- Proficiências representam domínio específico.
- Profissões agrupam perícias, receitas, passivas e desbloqueios.
- Passivas alteram regras por efeitos estruturados.
- O backend calcula pontuação efetiva, XP e níveis.
- Atividades triviais concedem XP reduzido ou zero.

## 12. Pendências de atributos

- validar 16 pontos livres com cinco atributos;
- definir limite por atributo;
- testar níveis 1, 5, 10, 20 e 50;
- calibrar secundários, Vida, Mana e Vigor;
- validar Agilidade contra Destreza.

## 13. Pendências de atores

- orçamento de criação por nível e ameaça;
- espécies, arquétipos e variantes;
- nível de desafio;
- grupos;
- moral, fuga e rendição;
- gatilhos de materialização e promoção;
- domesticação e recrutamento;
- relações, impressões e memórias;
- migração após revisão;
- validação temática por tags.

## 14. Pendências de equipamentos e qualidade

- calibrar multiplicadores de qualidade e slot;
- definir arredondamento;
- calibrar custos de modificadores;
- limite de recuperação negativa;
- perfis de distribuição por categoria;
- orçamento de ações e passivas;
- escalonamento de efeitos;
- equipamentos versáteis;
- migração de instâncias antigas;
- agrupamento visual.

## 15. Pendências de inventário e saque

- limites de pilha;
- compatibilidade de condição;
- sobrecarga;
- durabilidade e reparo;
- contêineres aninhados;
- tempo de coleta;
- divisão de saque em grupo;
- identificação de itens;
- expiração de cadáveres;
- prevenção de duplicação concorrente.

## 16. Pendências de fabricação

- fórmula de sucesso;
- limiares de qualidade;
- perfis de distribuição;
- política de falha e consumo;
- tempo de produção;
- reparo e conservação;
- encantamento;
- desmontagem e subprodutos;
- fabricação em lote;
- XP e prevenção de treinamento abusivo.

## 17. Pendências de economia

- cesta econômica;
- preços Comuns por categoria;
- multiplicadores de qualidade;
- taxa de recompra;
- margens e negociação;
- mercados regionais;
- itens roubados e vinculados;
- condição e durabilidade;
- inflação;
- migração de preços antigos.

## 18. Pendências de combate e ações

- base de 75% da chance de acerto;
- influência do nível;
- Defesa Crítica e mitigação;
- prioridades da fila;
- tempos de movimento, ataques e magias;
- engajamento, reações, cobertura e projéteis;
- testes de perseguição, interrupção e múltiplos inimigos;
- aleatoriedade auditável.

## 19. Pendências de progressão

- XP do personagem;
- divisão em grupo;
- diferença de nível;
- recompensas de missões e objetivos;
- nível máximo;
- relação entre personagem, perícia e profissão.

## 20. Próximas validações

1. Criar uma definição de Adaga Élfica Comum de referência.
2. Simular cinco fabricações com falhas e qualidades diferentes.
3. Confirmar uma definição e várias instâncias.
4. Validar orçamento e preços resolvidos por qualidade.
5. Agrupar duas instâncias Comuns e uma Rara na interface sem mesclar estados individuais.
6. Tentar criar cadastro duplicado por qualidade e exigir reutilização.
7. Criar variante real e comparar com simples mudança de qualidade.
8. Alterar multiplicador futuro e confirmar congelamento das instâncias antigas.
9. Continuar testes de atores, combate, comércio e fabricação.
10. Ajustar antes das versões `v1.0`.

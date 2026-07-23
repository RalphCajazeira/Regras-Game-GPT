# Decisões e Pendências

Este arquivo separa regras aceitas de propostas ainda sujeitas a testes.

## 1. Decisões consolidadas

### Arquitetura

- O GPT conduz interações localmente usando snapshots versionados.
- O GPT pode criar, corrigir, balancear e evoluir conteúdos por meio das Actions adequadas.
- O backend calcula, valida, persiste e aplica consequências autoritativas.
- Validação do backend não significa imutabilidade dos conteúdos.
- Combate, comércio, perícias, fabricação e materialização de atores não exigem chamada a cada microação.
- O frontend futuro usa o mesmo domínio e as mesmas validações.

### Criação e revisão

- Todo conteúdo pode ser revisado quando houver justificativa coerente.
- Sugestões do jogador são avaliadas pelo GPT; não são aplicadas automaticamente.
- O GPT deve consultar regras, identidade temática, orçamento e vínculos antes de propor uma alteração.
- Revisões preferem manter ID e código, criar nova versão e preservar vínculos.
- Toda revisão declara escopo para futuras instâncias, instância atual, instâncias selecionadas ou todas as compatíveis.
- Eventos históricos não são recalculados automaticamente.
- O backend deve retornar erros acionáveis, não apenas negar a operação.

### Atributos

Os cinco atributos primários são:

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
- Especializações não criam novos atributos primários automaticamente.

### Atores

- Ator é o modelo universal para pessoas, animais, monstros, criaturas, espíritos, construtos, invocações e personagens do jogador.
- Natureza, espécie, arquétipo, facção, papel e controle são conceitos separados.
- Entidades irrelevantes podem existir apenas como avistamentos.
- Antes de interação mecânica relevante, o ator é materializado com ficha completa.
- A ficha materializada define atributos, recursos, ações, passivas, equipamentos, inventário, dinheiro e recompensas aplicáveis.
- Dados materializados ficam congelados no snapshot da sessão.
- O GPT não pode inventar retroativamente poções, magias, dinheiro, equipamentos ou drops.
- Instâncias podem ser promovidas de encontro para campanha ou mundo sem duplicação.
- Companheiro é papel e vínculo, não espécie ou natureza.
- A ficha real e o conhecimento do jogador são estados separados.
- Materialização usa semente determinística para permitir reprodução.

### Combate

- Posições e alcances usam metros.
- O tempo usa linha do tempo contínua em ticks.
- Existe uma única rolagem de acerto.
- Precisão é comparada com Esquiva ou Resistência.
- A chance de acerto fica entre 1% e 99%.
- Após o acerto entram crítico, Defesa Crítica e mitigação.
- Ações possuem preparação, recuperação e tempo mínimo.
- Movimento e ataque são separados, salvo ação composta.
- Ações podem ser interrompidas.

### Equipamentos

- Todo equipamento declara todos os modificadores, inclusive os zerados.
- `primaryModifiers` inclui os cinco atributos.
- Corpo e Peito são slots diferentes.
- Armas podem ocupar uma ou duas mãos.
- Qualidades: Inferior, Comum, Raro, Épico e Lendário.
- Nível, qualidade e slot formam o orçamento.
- Penalidades recuperam orçamento positivo.
- Equipamentos podem conceder ações, passivas e proficiências válidas.
- Qualidade não altera automaticamente o peso.

### Economia

- A moeda única é a Coroa, código `CROWN`.
- Valores monetários são inteiros.
- Equipamento define preços-base; mercado e negociação definem preço final.
- Passivas comerciais modificam transações, não preços-base.
- O GPT negocia somente dentro das faixas fornecidas.
- O backend controla saldo, estoque, propriedade e transferência atômica.

### Perícias, profissões e passivas

- Atributos representam potencial geral.
- Perícias representam treinamento progressivo.
- Proficiências representam domínio de arma, ferramenta ou categoria.
- Profissões agrupam perícias, receitas, passivas e desbloqueios.
- Passivas alteram regras por efeitos estruturados e versionados.
- O backend calcula pontuação efetiva, XP e níveis.
- Atividades triviais concedem XP reduzido ou zero.

### Fabricação

- Toda fabricação usa receita ou procedimento conhecido.
- Receita, material, ferramenta, oficina e desbloqueios limitam a qualidade máxima.
- Perícia melhora sucesso e qualidade sem ignorar o teto.
- Consumo, criação, reparo, qualidade e XP são autoritativos no backend.
- O GPT pode conduzir uma sessão de fabricação a partir de snapshot completo.

## 2. Simulação anterior inválida

A primeira simulação de Mago nível 5 contra Guerreiro nível 5 usou orçamentos primários diferentes e não deve ser usada como validação.

## 3. Pendências de atributos

- validar os 16 pontos livres da criação com cinco atributos;
- definir limite de investimento por atributo;
- testar níveis 1, 5, 10, 20 e 50;
- calibrar fórmulas secundárias;
- validar Vida, Mana e Vigor;
- validar Agilidade contra Destreza em arquétipos diferentes.

## 4. Pendências de atores

- definir orçamento de criação por nível e categoria de ameaça;
- definir espécies, arquétipos e variantes;
- definir cálculo de nível de desafio;
- definir composição e materialização de grupos;
- definir moral, fuga, rendição e comportamento;
- definir critérios de materialização automática;
- definir critérios de promoção para campanha e mundo;
- definir domesticação, recrutamento e controle de companheiros;
- definir relações, impressões e memórias completas;
- integrar inventário, loot e coleta;
- definir migração de instâncias após revisão de definição;
- ampliar validação temática por tags.

## 5. Pendências de combate e ações

- validar base de 75% da chance de acerto;
- calibrar influência do nível;
- validar Defesa Crítica e mitigação;
- definir prioridades da fila de eventos;
- calibrar tempos de movimento, ataques e magias;
- definir engajamento, reações, cobertura e projéteis;
- testar arqueiro contra perseguidor, conjuração interrompida e múltiplos inimigos;
- definir política auditável de aleatoriedade.

## 6. Pendências de equipamentos

- validar multiplicadores de slot;
- calibrar custos dos modificadores;
- definir limite de recuperação negativa;
- definir orçamento de efeitos, passivas e ações concedidas;
- validar materiais, conservação e reparos;
- definir equipamentos versáteis;
- validar preços-base junto à economia;
- definir regras de coerência temática e revisão entre tags, nomes e efeitos.

## 7. Pendências de economia

- validar cesta econômica inicial;
- completar preços-base por categoria;
- calibrar nível, qualidade, material e poder utilizado;
- validar margens, recompra e negociação;
- integrar perícia Comércio e passivas comerciais;
- definir mercados regionais, escassez e eventos;
- definir itens roubados, vinculados e não comercializáveis;
- testar inflação em campanhas longas.

## 8. Pendências de perícias e profissões

- definir curva de XP das perícias;
- definir nível máximo;
- definir pesos de atributos por perícia;
- separar perícia, proficiência e profissão no domínio;
- definir aquisição e evolução de passivas;
- definir acúmulo, exclusividade e limites de efeitos;
- definir retorno decrescente por repetição;
- validar perícias sociais, de combate, coleta e produção.

## 9. Pendências de fabricação

- fórmula definitiva de sucesso;
- limiares de qualidade;
- teto por receita, material, ferramenta e oficina;
- política de falhas e consumo;
- tempo de produção;
- reparos, durabilidade e conservação;
- encantamento e efeitos especiais;
- desmontagem e subprodutos;
- fabricação em lote;
- XP e prevenção de treinamento abusivo.

## 10. Pendências de criação, revisão e versionamento

- definir schemas universais de proposta e revisão;
- definir impacto e dependências antes da alteração;
- definir políticas de migração por domínio;
- definir regras de compatibilidade retroativa;
- definir histórico, autor, justificativa e aprovação;
- definir revisão local de instância versus definição global;
- definir política para sessões em andamento;
- definir validações semânticas extensíveis;
- permitir revisão administrativa pelo frontend usando o mesmo domínio.

## 11. Pendências de progressão geral

- curva de XP do personagem;
- divisão de XP em grupo;
- diferença de nível;
- recompensa por missões e objetivos;
- nível máximo;
- relação entre nível do personagem, perícia e profissão.

## 12. Próximas validações

1. Materializar Lobo Alfa, bandido individual e grupo de bandidos.
2. Confirmar que inventário, equipamentos, ações e saque ficam definidos antes da interação.
3. Promover um ator anônimo sobrevivente para ator persistente sem duplicação.
4. Revisar um conteúdo tematicamente incoerente e validar versionamento e escopo.
5. Criar Guerreiro, Gatuno, Arqueiro, Mago e híbrido com o mesmo orçamento.
6. Testar os cinco atributos nos níveis 1, 5, 10, 20 e 50.
7. Simular combate temporal e medir duração, acerto e frequência de ações.
8. Simular Comércio com passivas e limites de preço.
9. Simular Ferraria em receitas triviais, equivalentes e difíceis.
10. Ajustar antes de publicar versões `v1.0`.
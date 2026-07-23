# Decisões e Pendências

Este arquivo separa as regras aceitas das propostas que ainda precisam de testes.

## Decisões consolidadas

### Arquitetura

- O GPT conduz a narrativa e resolve interações localmente usando snapshots versionados.
- O backend persiste, recalcula, valida e aplica consequências autoritativas.
- Combate e comércio não exigem chamada a cada microação ou frase.
- O GPT envia histórico e estado final ao encerrar ou em checkpoint opcional.
- O frontend futuro utilizará o mesmo domínio e as mesmas validações.

### Atributos

- Primários: Força, Destreza, Inteligência e Vitalidade.
- Cada nível concede 10 pontos primários.
- Pontos podem permanecer não distribuídos.
- Secundários são derivados e retornados pelo backend.

### Combate

- Existe somente uma rolagem de acerto.
- Precisão é comparada com Esquiva ou Resistência.
- Esquiva é rating de oposição, não uma segunda chance percentual.
- A chance de acerto fica entre 1% e 99%.
- Após o acerto entram crítico ofensivo, Defesa Crítica e defesa passiva.
- Defesa Crítica pode anular completamente o ataque.
- Ataques localizados trocam precisão por crítico ou penetração.
- Magias declaram qual oposição utilizam.

### Equipamentos

- Todo equipamento declara todos os modificadores, inclusive os zerados.
- Corpo e Peito são slots diferentes.
- Armas podem ocupar uma ou duas mãos.
- Qualidades: Inferior, Comum, Raro, Épico e Lendário.
- Nível, qualidade e slot formam o orçamento de poder.
- Penalidades negativas recuperam pontos para bônus positivos.
- Todo item comercializável possui preço-base de compra, preço-base de venda, moeda e peso.
- Qualidade não altera automaticamente o peso.
- Equipamento define preços-base; mercado e negociação definem preço final.

### Economia e comércio

- Existe uma moeda única chamada Coroa, persistida como `CROWN`.
- Valores monetários são inteiros.
- Todo preço final deriva de uma referência econômica validável.
- O GPT pode negociar somente dentro das faixas fornecidas.
- O backend controla saldo, estoque, propriedade, limites e transferência.
- Comerciantes possuem especialidade, estoque e dinheiro limitados.
- Transações são atômicas.
- Qualidade e condição do item são conceitos separados.
- Recompensas e gastos devem ser calibrados para preservar o valor da moeda.

## Simulação anterior inválida

A primeira simulação de Mago nível 5 contra Guerreiro nível 5 usou orçamentos diferentes:

```text
Guerreiro: 48 pontos primários
Mago: 40 pontos primários
```

A taxa de vitória daquela simulação não deve ser usada como validação.

## Pendências de atributos

- Confirmar os 16 pontos livres da criação.
- Definir limite de investimento por atributo.
- Testar as fórmulas nos níveis 1, 5, 10, 20 e 50.
- Confirmar a fórmula definitiva de ATK Físico.
- Definir unidade e efeito de Velocidade e Capacidade de Carga.
- Calibrar Vida, Mana e Vigor.

## Pendências de combate

- Validar a base de 75% da chance de acerto.
- Calibrar a influência do nível.
- Validar Defesa Crítica e mitigação passiva.
- Definir economia de ações e movimento.
- Definir duas armas, distâncias, condições, cura e recuo.
- Definir política de aleatoriedade e auditoria.

## Pendências de equipamentos

- Validar multiplicadores de slot.
- Calibrar custos dos modificadores.
- Definir limites para trocas com penalidades negativas.
- Definir orçamento de efeitos especiais.
- Definir materiais, fabricação, conservação e reparos.
- Definir requisitos e equipamentos versáteis.
- Validar preços-base por categoria em conjunto com a economia.

## Pendências de economia

- Validar a cesta econômica inicial.
- Definir tabela completa de preços-base por categoria.
- Calibrar fatores de nível, qualidade, material e poder utilizado.
- Validar faixas de margem, recompra e negociação.
- Definir competências sociais e suas fórmulas.
- Definir mercados regionais, escassez e eventos econômicos.
- Definir preços de consumíveis, materiais, serviços e hospedagem.
- Definir regras para itens roubados, vinculados, únicos e não comercializáveis.
- Calibrar reparos, conservação e durabilidade.
- Testar inflação e acúmulo de riqueza em campanhas longas.

## Pendências de progressão

- Definir curva de XP.
- Definir diferença de nível e divisão em grupo.
- Definir recompensa por objetivos não baseados em derrota.
- Definir nível máximo e recuperação ao subir de nível.
- Relacionar recompensas monetárias ao nível, risco, duração e dificuldade.

## Próximas validações

### Atributos e combate

1. Fechar as fórmulas de atributos.
2. Criar Guerreiro, Gatuno, Mago e híbrido com o mesmo orçamento.
3. Testar níveis 1, 5, 10 e 50.
4. Simular encontros de mesmo nível e de diferença extrema.
5. Medir chance de acerto, dano, duração e taxa de vitória.

### Economia

1. Montar cesta de itens, serviços e missões de nível 1.
2. Simular um personagem jogando e comprando durante várias sessões.
3. Medir entrada e saída de Coroas.
4. Testar lojas gerais e especialistas.
5. Testar negociação nos limites mínimo e máximo.
6. Ajustar antes de publicar uma versão `v1.0`.

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
- O atributo genérico `speed` foi substituído por:
  - `movementSpeed`;
  - `physicalActionSpeed`;
  - `castingSpeed`.
- `movementSpeed` é expresso em metros por segundo.

### Combate

- Existe somente uma rolagem de acerto.
- Precisão é comparada com Esquiva ou Resistência.
- Esquiva é rating de oposição, não uma segunda chance percentual.
- A chance de acerto fica entre 1% e 99%.
- Após o acerto entram crítico ofensivo, Defesa Crítica e defesa passiva.
- Defesa Crítica pode anular completamente o ataque.
- Ataques localizados trocam precisão por crítico ou penetração.
- Magias declaram qual oposição utilizam.
- O combate utiliza posições e alcances em metros.
- O combate usa linha do tempo contínua, não rodadas rígidas.
- Cada ator possui `nextReadyAt`.
- Ações possuem preparação e recuperação.
- O próximo evento é aquele com menor tempo agendado.
- Movimento e ataque são ações separadas, salvo ação composta.
- Ações em preparação podem ser interrompidas.
- Reações geram dívida temporal.
- Personagens rápidos podem agir mais vezes antes de ações lentas terminarem.
- O GPT não força acidentes para anular vantagens legítimas de mobilidade.

### Ações, habilidades e magias

- Existe uma estrutura universal para ataques, habilidades, magias, movimento, reações, consumíveis, cura e fuga.
- Alcance e áreas são expressos em metros.
- Áreas podem usar círculo, cone, linha e raio ao redor do usuário.
- Toda ação declara alvo, alcance, tempo, custo, acerto, dano, cura, efeitos e interrupção aplicáveis.
- Toda ação possui tempos mínimos.
- Declarar uma ação não significa resolvê-la imediatamente.
- Alcance e linha de visão são revalidados em `resolveAt`.
- Custos declaram se são consumidos no início, resolução ou sucesso.
- Armas e equipamentos podem conceder ações completas.

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
- Equipamentos podem modificar movimento, ação física e conjuração.
- Armas, escudos e focos podem conceder ações por código.

### Economia e comércio

- Existe uma moeda única chamada Coroa, persistida como `CROWN`.
- Valores monetários são inteiros.
- Todo preço final deriva de uma referência econômica validável.
- O GPT pode negociar somente dentro das faixas fornecidas.
- O backend controla saldo, estoque, propriedade, limites e transferência.
- Comerciantes possuem especialidade, estoque e dinheiro limitados.
- Transações são atômicas.
- Qualidade e condição do item são conceitos separados.
- Recompensas e gastos devem preservar o valor da moeda.

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
- Testar fórmulas nos níveis 1, 5, 10, 20 e 50.
- Confirmar a fórmula definitiva de ATK Físico.
- Calibrar Vida, Mana e Vigor.
- Calibrar `movementSpeed`, `physicalActionSpeed` e `castingSpeed`.
- Definir unidade e efeito final de Capacidade de Carga.

## Pendências de combate temporal

- Validar `10 ticks = 1 segundo`.
- Calibrar disponibilidade inicial por Iniciativa.
- Definir prioridade de eventos com o mesmo tempo.
- Validar retorno decrescente de velocidade.
- Definir regras de ações simultâneas.
- Definir custo de Vigor por corrida.
- Definir engajamento, desengajamento e reações.
- Definir linha de visão, cobertura, obstáculos e terreno.
- Definir projéteis com tempo de deslocamento.
- Validar perseguições e recuos em áreas abertas e fechadas.
- Validar Defesa Crítica e mitigação passiva.
- Definir política de aleatoriedade e auditoria.

## Pendências de ações, habilidades e magias

- Calibrar tempos de ataques básicos por arma.
- Calibrar tempos e custos de magias.
- Definir fórmula de concentração.
- Definir interrupção, cancelamento e reembolso de recursos.
- Definir recargas e cargas.
- Definir movimento permitido durante preparação.
- Definir ataques compostos e investidas.
- Validar formatos e seleção de áreas.
- Definir cura, sobrecura e ressurreição.
- Criar sistema de condições e efeitos.

## Pendências de equipamentos

- Validar multiplicadores de slot.
- Calibrar custos dos modificadores.
- Calibrar modificadores temporais.
- Definir limites para trocas com penalidades negativas.
- Definir orçamento de efeitos especiais.
- Definir materiais, fabricação, conservação e reparos.
- Definir requisitos e equipamentos versáteis.
- Validar preços-base por categoria em conjunto com a economia.
- Definir penalidades de peso e carga sobre movimento.

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

### Atributos, ações e combate

1. Criar Guerreiro, Gatuno, Mago, Arqueiro e híbrido com o mesmo orçamento.
2. Definir ações básicas de adaga, espada pesada, arco e magia inicial.
3. Testar níveis 1, 5, 10, 20 e 50.
4. Simular encontros de mesmo nível e diferença extrema.
5. Simular perseguição de arqueiro contra combatente lento.
6. Simular personagem rápido interrompendo conjuração.
7. Simular jogador contra inimigo rápido e inimigo lento simultaneamente.
8. Medir acerto, dano, tempo entre ações, duração e taxa de vitória.

### Economia

1. Montar cesta de itens, serviços e missões de nível 1.
2. Simular um personagem jogando e comprando durante várias sessões.
3. Medir entrada e saída de Coroas.
4. Testar lojas gerais e especialistas.
5. Testar negociação nos limites mínimo e máximo.
6. Ajustar antes de publicar uma versão `v1.0`.
# Decisões e Pendências

Este arquivo separa as regras aceitas das propostas que ainda precisam de testes.

## Decisões consolidadas

### Arquitetura

- O GPT conduz a narrativa e resolve o encontro localmente.
- O backend persiste, recalcula, valida e aplica consequências autoritativas.
- O combate usa um snapshot versionado e não exige chamada a cada ação.
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
- Todo item possui preço de compra, preço de venda e peso.
- Qualidade não altera automaticamente o peso.

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
- Definir materiais, preços-base, moeda e conservação.
- Definir requisitos e equipamentos versáteis.

## Pendências de progressão

- Definir curva de XP.
- Definir diferença de nível e divisão em grupo.
- Definir recompensa por objetivos não baseados em derrota.
- Definir nível máximo e recuperação ao subir de nível.

## Próxima validação

1. Fechar as fórmulas de atributos.
2. Criar Guerreiro, Gatuno, Mago e híbrido com o mesmo orçamento.
3. Testar níveis 1, 5, 10 e 50.
4. Simular encontros de mesmo nível e de diferença extrema.
5. Medir chance de acerto, dano, duração e taxa de vitória.
6. Ajustar antes de publicar uma versão `v1.0`.

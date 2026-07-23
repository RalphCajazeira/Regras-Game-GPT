# Sistema de Economia e Comércio

**Versão da proposta:** `economy-v0.2`  
**Status:** em validação

## 1. Objetivo

Definir uma economia única e previsível, permitindo que o GPT interprete comerciantes, mercados e negociações sem criar preços arbitrários.

O backend mantém preços-base, saldos, estoques, limites e validações. O GPT conduz a experiência comercial dentro dos parâmetros carregados.

## 2. Princípios

1. Existe uma única moeda em todo o jogo.
2. Valores monetários são inteiros.
3. Todo item comercializável possui preço-base de compra e venda.
4. Preço-base não é necessariamente o preço final.
5. Mercado, condição, comerciante, reputação, perícias e passivas podem modificar a transação.
6. O GPT varia preços somente dentro das faixas autorizadas.
7. O backend controla saldo, estoque, propriedade e resultado final.
8. Transações são atômicas.
9. Comerciantes possuem especialidade, estoque e dinheiro limitados.
10. Entradas e saídas de moeda devem preservar o valor econômico do jogo.

## 3. Moeda única

```text
Nome: Coroa
Plural: Coroas
Código: CROWN
```

```json
{
  "currencyCode": "CROWN",
  "amount": 125
}
```

O saldo é persistido como `currencyBalance`.

Regras:

- saldo não pode ser negativo;
- dinheiro não possui peso na primeira versão;
- recompensas, lojas e serviços usam a mesma moeda;
- moedas regionais narrativas são convertidas antes de alterar o estado.

## 4. Escala de referência

| Referência | Preço aproximado |
|---|---:|
| Refeição simples | `5` |
| Refeição boa | `12` |
| Hospedagem simples | `20` |
| Kit básico de viagem | `35` |
| Poção comum inicial | `30` |
| Adaga comum nível 1 | `40` |
| Espada curta comum nível 1 | `70` |
| Espada longa comum nível 1 | `110` |
| Peitoral leve comum nível 1 | `70` |
| Peitoral médio comum nível 1 | `110` |
| Peitoral pesado comum nível 1 | `160` |
| Reparo simples | `10` a `30` |
| Missão inicial curta | `50` a `150` |

A cesta é uma âncora de calibração e ainda precisa de simulações.

## 5. Tipos de preço

### Preço-base de compra — `baseBuyPrice`

Valor normal do item novo ou em condição padrão, em mercado estável.

### Preço-base de venda — `baseSellPrice`

Referência normal de recompra por comerciante geral.

```text
baseSellPrice = arredondar_para_baixo(baseBuyPrice × 0,40)
```

### Preço de referência — `referencePrice`

Valor calculado para aquele mercado e comerciante antes da negociação final.

### Preço final — `transactionPrice`

Valor efetivamente transferido. Não altera automaticamente os preços-base.

## 6. Formação do preço

```text
Preço-base =
preço da categoria
× nível
× qualidade
× material
× fabricação
× poder utilizado
```

```text
Preço de referência =
preço-base
× condição
× escassez
× demanda
× especialidade
× margem comercial
```

```text
Preço final =
preço de referência
modificado por perícias, passivas, reputação e negociação
limitado pela faixa autorizada
```

O backend retorna:

```text
referencePrice
minimumAllowedPrice
maximumAllowedPrice
```

## 7. Compra e venda

Faixas iniciais propostas:

| Situação | Faixa aproximada |
|---|---:|
| Loja vendendo ao jogador | `90%` a `140%` da referência |
| Loja comprando do jogador | `25%` a `60%` da referência |
| Especialista comprando categoria relevante | até `70%` |
| Desconto normal negociado | até `15%` |
| Desconto excepcional | até `25%` |

Essas faixas ainda precisam de simulações.

## 8. Comerciante

```json
{
  "merchantType": "BLACKSMITH",
  "buyCategories": ["WEAPON", "ARMOR", "METAL"],
  "sellCategories": ["WEAPON", "ARMOR"],
  "specialtyBuyMultiplier": 1.2,
  "defaultBuyMultiplier": 0.4,
  "availableFunds": 1500,
  "negotiationFlexibilityPercent": 15
}
```

O comerciante pode recusar categoria, item roubado, item danificado ou transação acima de seu saldo.

## 9. Perícia de Comércio

Comércio é uma perícia, não um atributo primário.

Pode considerar:

- nível da perícia;
- Destreza para leitura e execução técnica;
- Inteligência para avaliação e estratégia;
- reputação;
- relação com o comerciante;
- passivas;
- argumento apresentado;
- rolagem.

Os pesos são carregados pelo backend. O GPT não inventa a contribuição dos atributos.

## 10. Passivas comerciais

Exemplos:

```text
Desconto → reduz preço de compra em X%
Boa Revenda → aumenta valor recebido em X%
Avaliador → melhora informação sobre preço e condição
Cliente Frequente → melhora faixa com comerciante específico
```

Passivas modificam a transação, nunca `baseBuyPrice` ou `baseSellPrice`.

### Acúmulo

```text
passivas
+ perícia
+ reputação
+ relação
+ negociação
= modificador agregado
```

O backend aplica limites e faixa autorizada depois da consolidação.

Exemplo:

```text
Preço de referência: 100
Passiva: -10%
Negociação: -15%
Preço bruto: 75
Preço mínimo autorizado: 80
Preço final: 80
```

## 11. Fabricação e economia

Itens fabricados usam os mesmos preços-base, mas seu custo econômico pode incluir:

- materiais;
- combustível;
- ferramentas;
- oficina;
- taxas;
- serviços auxiliares;
- tempo.

Qualidade maior pode aumentar preço, mas a venda continua sujeita a demanda, especialidade, estoque e saldo do comerciante.

Fabricação não cria dinheiro diretamente. Ela transforma recursos e tempo em itens que ainda precisam ser vendidos ou usados.

## 12. Recompensas e saídas de moeda

Fontes:

- missões;
- contratos;
- tesouros;
- venda de itens;
- serviços;
- exploração.

Saídas:

- consumíveis;
- reparos;
- hospedagem;
- transporte;
- treinamento;
- fabricação;
- ferramentas;
- oficinas;
- encantamentos;
- informações;
- taxas.

Recompensas devem considerar nível, risco, duração e importância.

## 13. Sessão comercial

O snapshot contém:

- saldos;
- estoque;
- categorias aceitas;
- preços-base e de referência;
- faixas autorizadas;
- condição dos itens;
- perícia de Comércio;
- passivas;
- reputação e relação;
- flexibilidade de negociação;
- `stateVersion`.

O GPT conduz diálogo e contrapropostas localmente. Ao final, envia a transação consolidada.

## 14. Validação do backend

- sessão e versão;
- saldo;
- estoque e propriedade;
- categorias aceitas;
- preços-base;
- mercado e condição;
- perícias e passivas;
- faixas autorizadas;
- total monetário;
- ausência de duplicação;
- transferência atômica.

## 15. Responsabilidades

### GPT

- interpretar o comerciante;
- apresentar preços carregados;
- usar perícias e passivas aplicáveis;
- conduzir negociação;
- manter preços dentro dos limites;
- registrar propostas e rolagens;
- enviar transação consolidada.

### Backend

- calcular preços e faixas;
- persistir saldos, estoques e relações;
- calcular pontuação efetiva de Comércio;
- validar passivas e acúmulos;
- transferir itens e Coroas;
- manter histórico;
- impedir abusos econômicos.

## 16. Pendências

- validar cesta de preços;
- calibrar margens e recompra;
- definir pesos da perícia Comércio;
- calibrar passivas comerciais;
- definir mercados regionais;
- itens roubados e vinculados;
- conservação e durabilidade;
- inflação em campanhas longas;
- custos completos de fabricação e serviços.

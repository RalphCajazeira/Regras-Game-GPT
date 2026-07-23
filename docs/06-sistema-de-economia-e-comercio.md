# Sistema de Economia e Comércio

**Versão da proposta:** `economy-v0.3`  
**Status:** em validação

## 1. Objetivo

Definir uma economia previsível para que o GPT conduza comércio e negociação sem criar preços arbitrários.

O sistema distingue:

- preço Comum de referência da definição;
- preço-base resolvido da instância;
- preço de referência do mercado;
- preço final da transação.

## 2. Princípios

1. Existe uma única moeda.
2. Valores monetários são inteiros.
3. Definições usam qualidade Comum como referência econômica.
4. Qualidade da instância resolve seu próprio preço-base.
5. Mercado, condição, comerciante, reputação, perícias e passivas alteram a transação.
6. O GPT negocia somente dentro das faixas autorizadas.
7. O backend controla saldo, estoque, propriedade e resultado final.
8. Transações são atômicas.
9. Comerciantes possuem especialidade, estoque e dinheiro limitados.
10. Fabricar um item cria valor potencial, não dinheiro automático.

## 3. Moeda

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

O saldo é `currencyBalance` e não pode ficar negativo.

## 4. Escala econômica inicial

| Referência | Preço aproximado |
|---|---:|
| Refeição simples | `5` |
| Refeição boa | `12` |
| Hospedagem simples | `20` |
| Kit de viagem | `35` |
| Poção comum inicial | `30` |
| Adaga comum nível 1 | `40` |
| Espada curta comum nível 1 | `70` |
| Espada longa comum nível 1 | `110` |
| Peitoral leve comum nível 1 | `70` |
| Peitoral médio comum nível 1 | `110` |
| Peitoral pesado comum nível 1 | `160` |
| Reparo simples | `10` a `30` |
| Missão inicial curta | `50` a `150` |

A cesta ainda precisa de simulações.

## 5. Camadas de preço

### 5.1 Definição

A definição guarda a referência Comum:

```text
commonBaseBuyPrice
commonBaseSellPrice
currencyCode
```

Exemplo:

```json
{
  "definitionCode": "elven-dagger",
  "referenceQuality": "COMMON",
  "commonBaseBuyPrice": 120,
  "commonBaseSellPrice": 48,
  "currencyCode": "CROWN"
}
```

### 5.2 Instância

A instância guarda os valores-base resolvidos para sua qualidade:

```text
resolvedBaseBuyPrice
resolvedBaseSellPrice
```

```json
{
  "definitionCode": "elven-dagger",
  "quality": "RARE",
  "resolvedBaseBuyPrice": 240,
  "resolvedBaseSellPrice": 96
}
```

### 5.3 Mercado

```text
referencePrice
```

É calculado para região, comerciante e condição atual.

### 5.4 Transação

```text
transactionPrice
```

É o valor efetivamente transferido depois de negociação e limites.

## 6. Formação do preço Comum

```text
Preço Comum da Definição =
preço da categoria
× fator de nível
× material
× fabricação
× fator de poder Comum utilizado
```

A definição não inclui multiplicador de qualidade diferente de Comum.

## 7. Resolução de preço por qualidade

Multiplicadores iniciais:

| Qualidade | Multiplicador de preço |
|---|---:|
| Inferior | `0,50` |
| Comum | `1,00` |
| Raro | `2,00` |
| Épico | `4,00` |
| Lendário | `8,00` |

```text
resolvedBaseBuyPrice =
commonBaseBuyPrice
× multiplicador da qualidade
× ajustes explícitos da instância
```

```text
resolvedBaseSellPrice =
arredondar_para_baixo(
  resolvedBaseBuyPrice × taxa-base de recompra
)
```

Taxa inicial de referência:

```text
40%
```

Os valores resolvidos ficam congelados na instância com as versões de regra usadas.

## 8. Preço de referência do mercado

```text
referencePrice =
preço-base resolvido da instância
× condição
× escassez
× demanda
× especialidade
× margem comercial
```

O backend retorna:

```text
referencePrice
minimumAllowedPrice
maximumAllowedPrice
```

## 9. Preço final

```text
transactionPrice =
referencePrice
modificado por perícias, passivas, reputação e negociação
limitado pela faixa autorizada
```

A negociação nunca altera permanentemente o preço da definição ou da instância.

## 10. Compra e venda

Faixas iniciais:

| Situação | Faixa aproximada |
|---|---:|
| Loja vendendo | `90%` a `140%` da referência |
| Loja comprando | `25%` a `60%` da referência |
| Especialista comprando categoria relevante | até `70%` |
| Desconto normal | até `15%` |
| Desconto excepcional | até `25%` |

## 11. Comerciante

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

O comerciante pode recusar categoria, item roubado, vinculado, quebrado ou transação acima de seu saldo.

## 12. Perícia de Comércio

Comércio é perícia, não atributo primário.

Pode considerar:

- nível da perícia;
- Destreza;
- Inteligência;
- reputação;
- relação;
- passivas;
- argumento;
- rolagem.

Os pesos são calculados pelo backend.

## 13. Passivas comerciais

```text
Desconto → reduz compra
Boa Revenda → aumenta venda
Avaliador → melhora informação sobre preço e condição
Cliente Frequente → melhora faixa com comerciante específico
```

Passivas modificam apenas a transação.

Exemplo:

```text
Preço de referência: 100
Passiva: -10%
Negociação: -15%
Preço bruto: 75
Mínimo autorizado: 80
Preço final: 80
```

## 14. Qualidade, condição e variante

```text
Qualidade = potencial de fabricação da instância
Condição = conservação atual
Variante = diferença real de identidade ou mecânica
```

- qualidade muda preço-base resolvido;
- condição muda preço de mercado;
- variante pode possuir outra referência Comum;
- qualidade não cria cadastro econômico separado.

## 15. Fabricação e economia

Uma receita produz instância vinculada a definição e variante existentes.

O backend resolve:

- qualidade;
- modificadores;
- preço-base da instância;
- proveniência;
- custos consumidos.

Custos possíveis:

- materiais;
- combustível;
- ferramentas;
- oficina;
- taxas;
- serviços;
- tempo.

## 16. Sessão comercial

O snapshot contém:

- saldos;
- estoque com instâncias e pilhas;
- qualidade, condição e preços resolvidos;
- categorias aceitas;
- preços de referência;
- faixas autorizadas;
- perícia de Comércio;
- passivas;
- reputação e relação;
- reservas;
- `stateVersion`.

O GPT conduz diálogo e contrapropostas. O backend valida e transfere atomicamente.

## 17. Responsabilidades

### GPT

- apresentar qualidade e condição corretas;
- usar o preço resolvido da instância;
- conduzir negociação dentro da faixa;
- não criar um preço por qualidade sem regra;
- registrar propostas e resultado.

### Backend

- calcular referências Comuns;
- resolver preços por qualidade;
- congelar valores na instância;
- calcular mercado e faixas;
- validar passivas e negociação;
- controlar saldo, estoque e propriedade;
- impedir duplicação e arbitragem indevida.

## 18. Pendências

- calibrar cesta de preços;
- calibrar multiplicadores de qualidade;
- definir taxa de recompra por categoria;
- calibrar margens e negociação;
- mercados regionais;
- itens roubados e vinculados;
- condição e durabilidade;
- inflação;
- migração de preços resolvidos antigos.

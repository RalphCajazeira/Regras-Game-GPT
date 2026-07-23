# Sistema de Economia e Comércio

**Versão da proposta:** `economy-v0.1`  
**Status:** em validação

## 1. Objetivo

Definir uma economia única e previsível para o jogo, permitindo que o GPT interprete comerciantes, mercados e negociações sem criar preços arbitrários.

O backend mantém preços-base, saldos, estoques, limites e validações. O GPT conduz a experiência comercial dentro dos parâmetros carregados.

## 2. Princípios

1. Existe uma única moeda em todo o jogo.
2. Todos os valores monetários são inteiros.
3. Todo item comercializável possui preço-base de compra e preço-base de venda.
4. Preço-base não é necessariamente o preço final da transação.
5. O preço final pode variar por mercado, conservação, comerciante, reputação e negociação.
6. O GPT pode variar preços somente dentro das faixas autorizadas.
7. O backend é a autoridade sobre saldo, estoque, propriedade dos itens e resultado final.
8. Toda transação é atômica: ou todos os itens e valores são transferidos, ou nada é alterado.
9. Comerciantes possuem especialidade, estoque e dinheiro limitados.
10. Entradas e saídas de moeda devem ser calibradas para preservar o valor econômico do jogo.

## 3. Moeda única

Nome exibido:

```text
Coroa
```

Código persistido:

```text
CROWN
```

Plural:

```text
Coroas
```

Não existem conversões obrigatórias entre cobre, prata e ouro. Valores fracionários não são utilizados.

```json
{
  "currencyCode": "CROWN",
  "amount": 125
}
```

O saldo do ator pode ser persistido como:

```text
currencyBalance
```

Regras:

- saldo nunca pode ser negativo;
- dinheiro não possui peso na primeira versão;
- recompensas, serviços, lojas e negociações utilizam a mesma moeda;
- moedas regionais podem existir narrativamente, mas devem ser convertidas para Coroas antes de afetar o estado persistido.

## 4. Escala econômica de referência

A economia deve possuir uma cesta canônica que sirva como âncora para o GPT.

Valores iniciais propostos:

| Referência | Preço normal aproximado |
|---|---:|
| Refeição simples | `5` |
| Refeição de boa qualidade | `12` |
| Hospedagem simples por noite | `20` |
| Kit básico de viagem | `35` |
| Poção comum inicial | `30` |
| Adaga comum de nível 1 | `40` |
| Espada curta comum de nível 1 | `70` |
| Espada longa comum de nível 1 | `110` |
| Peitoral leve comum de nível 1 | `70` |
| Peitoral médio comum de nível 1 | `110` |
| Peitoral pesado comum de nível 1 | `160` |
| Reparo simples | `10` a `30` |
| Missão inicial curta | `50` a `150` |

Esses valores são referências de calibração, não uma tabela imutável de estoque. Devem ser revisados junto com recompensas, equipamentos e custos recorrentes.

## 5. Tipos de preço

### 5.1 Preço-base de compra — `baseBuyPrice`

Valor normal para comprar o item novo ou em condição padrão, em um mercado estável e sem negociação especial.

### 5.2 Preço-base de venda — `baseSellPrice`

Valor normal oferecido ao proprietário por um comerciante geral, antes de especialidade, conservação, demanda e negociação.

Proposta inicial:

```text
baseSellPrice = arredondar_para_baixo(baseBuyPrice × 0,40)
```

### 5.3 Preço de referência da sessão — `referencePrice`

Preço calculado para aquele mercado e comerciante antes da negociação final.

### 5.4 Preço final — `transactionPrice`

Valor efetivamente transferido após todos os modificadores e limites.

O preço final não altera automaticamente os preços-base da definição do item.

## 6. Formação do preço-base dos itens

Para equipamentos e outros itens escaláveis:

```text
Preço-base de compra =
preço-base da categoria
× fator de nível
× multiplicador de qualidade
× multiplicador de material
× multiplicador de fabricação
× fator de poder utilizado
```

Fator de nível proposto:

```text
1 + ((nível - 1) × 0,20)
```

Multiplicadores de qualidade propostos:

| Qualidade | Multiplicador de preço |
|---|---:|
| Inferior | `0,50` |
| Comum | `1,00` |
| Raro | `2,00` |
| Épico | `4,00` |
| Lendário | `8,00` |

O preço cresce mais rapidamente que o poder do item porque também representa escassez, técnica, materiais e dificuldade de substituição.

Fator de poder utilizado proposto:

```text
fatorPoder = 0,75 + ((pontosUtilizados ÷ pontosMáximos) × 0,25)
```

Limites:

```text
0,75 ≤ fatorPoder ≤ 1,00
```

Itens sem orçamento de poder utilizam `1,00` ou uma regra específica da categoria.

## 7. Preço comercial

### 7.1 Loja vendendo ao jogador

```text
Preço de referência =
baseBuyPrice
× condição
× escassez local
× margem do comerciante
× modificadores especiais
```

### 7.2 Loja comprando do jogador

```text
Oferta de referência =
baseSellPrice
× condição
× interesse da especialidade
× demanda local
× modificadores especiais
```

Depois da negociação:

```text
Preço final = limitar(
  preço negociado,
  preço mínimo permitido,
  preço máximo permitido
)
```

Todos os resultados são arredondados para números inteiros.

## 8. Multiplicadores comerciais

Faixas iniciais propostas:

### 8.1 Condição do item

| Condição | Multiplicador |
|---|---:|
| Excelente ou novo | `1,00` |
| Boa | `0,90` |
| Usada | `0,75` |
| Danificada | `0,50` |
| Quebrada | `0,10` ou não comercializável |

A qualidade do item e sua condição são conceitos diferentes. Um item lendário danificado continua lendário, mas vale menos até ser reparado.

### 8.2 Escassez ou demanda local

| Situação | Multiplicador sugerido |
|---|---:|
| Excesso de oferta | `0,75` a `0,90` |
| Mercado normal | `0,90` a `1,10` |
| Escasso | `1,10` a `1,30` |
| Muito escasso | `1,30` a `1,50` |

Valores fora dessas faixas exigem evento econômico explícito e persistido.

### 8.3 Margem de venda da loja

Faixa normal:

```text
0,90 a 1,40 do preço-base ajustado
```

O backend retorna o valor exato ou a faixa permitida para a sessão.

### 8.4 Recompra pelo comerciante

| Situação | Faixa sobre o valor de referência |
|---|---:|
| Comerciante geral | `25%` a `50%` |
| Comerciante interessado | `40%` a `60%` |
| Especialista buscando o item | até `70%` |

Um comerciante não deve normalmente comprar um item por valor igual ou superior ao preço pelo qual conseguiria revendê-lo, salvo evento, contrato ou colecionismo explícito.

## 9. Faixas autorizadas

Toda oferta carregada para o GPT deve informar:

```json
{
  "referencePrice": 100,
  "minimumAllowedPrice": 82,
  "maximumAllowedPrice": 125,
  "currencyCode": "CROWN"
}
```

O GPT pode:

- começar acima ou abaixo do preço de referência quando o perfil permitir;
- conceder descontos;
- fazer contrapropostas;
- melhorar ofertas de compra;
- recusar negociação.

O GPT não pode finalizar um valor fora da faixa autorizada.

## 10. Comerciantes

Todo comerciante persistido deve possuir um perfil comercial.

```json
{
  "merchantType": "BLACKSMITH",
  "currencyBalance": 1500,
  "buyCategories": ["WEAPON", "ARMOR", "METAL"],
  "sellCategories": ["WEAPON", "ARMOR"],
  "specialtyCategories": ["WEAPON", "ARMOR"],
  "defaultBuyMultiplier": 0.40,
  "specialtyBuyMultiplier": 0.55,
  "sellMarginMultiplier": 1.10,
  "negotiationFlexibility": 0.15,
  "acceptsStolenGoods": false
}
```

O perfil pode definir:

- categorias compradas e vendidas;
- categorias de especialidade;
- margem normal;
- flexibilidade de negociação;
- saldo disponível;
- relação com o jogador;
- restrições legais ou morais;
- interesse temporário em itens específicos;
- possibilidade de troca, encomenda ou crédito, quando existir regra própria.

## 11. Estoque e disponibilidade

O estoque pertence ao backend.

Cada entrada deve informar:

```json
{
  "itemDefinitionId": "iron-dagger",
  "itemInstanceId": "optional-instance-id",
  "quantity": 2,
  "referencePrice": 40,
  "minimumAllowedPrice": 34,
  "maximumAllowedPrice": 48,
  "available": true
}
```

Regras:

- itens únicos utilizam instância específica;
- itens empilháveis utilizam quantidade;
- o GPT não pode vender item fora do estoque carregado;
- o comerciante não pode comprar acima do próprio saldo;
- o jogador não pode comprar acima do próprio saldo;
- reserva de estoque durante uma negociação pode possuir prazo ou `stateVersion`.

## 12. Negociação

Como não existe Carisma entre os quatro atributos primários, negociação deve ser tratada por competências, reputação e contexto.

Possíveis competências futuras:

```text
Comércio
Persuasão
Intimidação
Enganação
```

A negociação pode considerar:

```text
competência social
+ reputação
+ relação com o comerciante
+ necessidade do item
+ qualidade do argumento
+ circunstâncias
+ rolagem
```

A interpretação do argumento é competência do GPT. A amplitude econômica permitida é competência do backend.

Faixas iniciais:

| Resultado | Efeito sugerido |
|---|---:|
| Falha grave | piora pequena ou fim da negociação |
| Falha | preço permanece |
| Sucesso | até `10%` de melhora |
| Sucesso alto | até `15%` de melhora |
| Resultado excepcional | até `25%`, respeitando o limite do comerciante |

Um bom argumento não permite ultrapassar `minimumAllowedPrice` ou `maximumAllowedPrice`.

## 13. Trocas e pagamentos mistos

Uma transação pode conter dinheiro e itens.

```text
Valor líquido =
valor das compras
- valor dos itens aceitos pelo comerciante
```

O backend deve avaliar cada item separadamente e garantir que a troca completa seja válida.

Na primeira versão, crédito, dívida e parcelamento permanecem fora do escopo, salvo se forem implementados como contratos persistidos.

## 14. Recompensas e entrada de moeda

Fontes de Coroas:

- missões e contratos;
- recompensas oficiais;
- venda de itens;
- tesouros;
- serviços e trabalhos;
- exploração;
- atividades econômicas específicas.

Recompensas devem considerar:

```text
nível recomendado
× dificuldade
× risco
× duração esperada
× importância narrativa
```

Referência de design:

- uma missão curta e segura paga consumíveis, alimentação ou hospedagem;
- uma missão comum paga uma parte relevante de um equipamento apropriado ao nível;
- uma missão difícil pode financiar um equipamento completo ou uma aquisição rara;
- uma missão inicial não deve financiar equipamento lendário sem justificativa excepcional.

O backend calcula e concede a recompensa autoritativa. O GPT pode apresentar ou negociar uma recompensa dentro das faixas recebidas.

## 15. Saídas de moeda

Para preservar o valor da moeda, o jogo deve possuir gastos recorrentes e opcionais:

- consumíveis;
- hospedagem e alimentação;
- transporte;
- reparos;
- treinamento;
- fabricação e melhoria;
- serviços mágicos;
- informações;
- contratação de ajudantes;
- taxas, licenças ou pedágios;
- despesas narrativas relevantes.

A existência desses gastos não deve transformar o jogo em manutenção excessiva. Eles servem para criar escolhas e controlar inflação.

## 16. Competência do GPT

O GPT deve:

- interpretar personalidade, interesse e comportamento do comerciante;
- apresentar estoque e preços carregados;
- conduzir diálogo, barganha e contrapropostas;
- avaliar a coerência narrativa do argumento do jogador;
- aplicar somente modificadores e limites recebidos;
- calcular ofertas locais reproduzíveis;
- registrar rolagens e decisões de negociação;
- manter estado local da sessão comercial;
- enviar a transação consolidada;
- propor preços-base de novos conteúdos usando as tabelas versionadas;
- nunca criar saldo, estoque, desconto ou recompensa sem base mecânica.

## 17. Competência do backend

O backend deve:

- persistir saldos de jogadores e comerciantes;
- persistir estoques, quantidades e propriedade;
- calcular ou validar preços-base;
- calcular faixas mínimas e máximas;
- persistir mercado, demanda, conservação, reputação e relação;
- validar o resultado da negociação;
- impedir saldo negativo e duplicação de itens;
- impedir venda de item inexistente ou compra acima do estoque;
- executar a transferência de itens e Coroas de forma atômica;
- registrar histórico e motivo dos modificadores;
- conceder recompensas;
- retornar erros acionáveis;
- versionar as regras econômicas.

## 18. Sessão comercial

A sessão pode ser carregada em uma única chamada:

```json
{
  "tradeSessionId": "trade-session-id",
  "stateVersion": 8,
  "ruleVersion": "economy-v0.1",
  "currencyCode": "CROWN",
  "playerBalance": 500,
  "merchant": {},
  "merchantBalance": 1200,
  "stock": [],
  "buyOffers": [],
  "negotiationRules": {}
}
```

Durante a conversa, o GPT pode comparar itens, negociar e montar a proposta sem chamar o backend a cada frase.

Ao concluir, envia:

```json
{
  "tradeSessionId": "trade-session-id",
  "baseStateVersion": 8,
  "purchases": [],
  "sales": [],
  "currencyTransferred": 120,
  "finalOffers": [],
  "negotiationLog": []
}
```

O backend recalcula tudo e persiste somente se a operação inteira for válida.

## 19. Validações do backend

O backend deve conferir:

1. existência e versão da sessão;
2. saldos disponíveis;
3. estoque e propriedade;
4. categorias aceitas pelo comerciante;
5. estado e conservação dos itens;
6. preço-base e modificadores;
7. limites mínimos e máximos;
8. resultado de negociação;
9. ausência de duplicação;
10. resultado monetário líquido;
11. transferência integral de todos os itens;
12. versões de regras utilizadas.

Exemplo de erro:

```json
{
  "success": false,
  "error": {
    "code": "TRADE_PRICE_OUT_OF_ALLOWED_RANGE",
    "message": "O preço final de 70 Coroas está abaixo do mínimo permitido de 82.",
    "details": {
      "referencePrice": 100,
      "minimumAllowedPrice": 82,
      "receivedPrice": 70,
      "currencyCode": "CROWN"
    },
    "recoveryActions": [
      "Use um preço entre 82 e 125 Coroas.",
      "Remova o desconto não autorizado.",
      "Recarregue a sessão caso as condições de mercado tenham mudado."
    ]
  }
}
```

## 20. Operações conceituais

```text
createTradeSession
loadTradeSession
submitTradeTransaction
cancelTradeSession
createMerchant
updateMerchantStock
grantCurrencyReward
```

Os nomes definitivos dependem do backend.

## 21. Integração com outros documentos

- Equipamentos definem nível, qualidade, material, poder e preços-base.
- Economia define preços de sessão, mercado, comerciante e transação.
- Progressão define recompensas de XP e poderá influenciar recompensas monetárias.
- Integração GPT/backend define snapshots, versões, validação e persistência.
- Frontend futuro exibirá saldo, estoque, faixas, ofertas e histórico.

## 22. Pendências

- validar a cesta econômica inicial;
- definir preços-base por categoria completa;
- calibrar recompensas por nível e dificuldade;
- definir conservação, reparos e durabilidade;
- definir competências sociais e suas fórmulas;
- validar margens, recompra e negociação;
- definir comportamento de mercados regionais;
- definir preço e orçamento de consumíveis, materiais e serviços;
- definir política para itens roubados, vinculados ou não comercializáveis;
- testar inflação em campanhas longas.

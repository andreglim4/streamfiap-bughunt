# Análise do Projeto

Durante a análise do projeto, foram encontrados alguns problemas relacionados às regras de negócio e também alguns pontos que poderiam ser melhorados seguindo boas práticas de POO e Clean Code.

Abaixo estão os principais problemas encontrados em cada classe.

## Conteudo.java

Na classe `Conteudo` encontrei alguns pontos que podem causar problemas no funcionamento do sistema.

### Encapsulamento

O atributo `duracaoMinutos` está como `public`:

```java
public int duracaoMinutos;
```

Isso permite que qualquer parte do sistema altere o valor diretamente. O mais adequado seria deixar o atributo como `private` e controlar seu acesso através de métodos.

### Problema com o cálculo do preço

O método `calcularPrecoAluguel()` retorna diretamente `9.90` na classe mãe.

Isso acaba sendo um problema porque nem todo tipo de conteúdo possui o mesmo preço. Por exemplo, o documentário deveria ser gratuito e a série possui um cálculo baseado na quantidade de temporadas.

O ideal seria deixar o método como `abstract` na classe `Conteudo`, fazendo com que cada classe filha seja responsável pelo seu próprio cálculo.

Outro ponto é o uso de `instanceof Promocionavel` dentro da classe pai. Isso faz com que `Conteudo` precise conhecer detalhes das classes filhas, o que acaba prejudicando o uso de polimorfismo.

### Validação da duração

Também foi identificada uma possível falha na regra de cadastro.

O sistema deveria impedir conteúdos com duração menor ou igual a zero, porém não foi encontrada uma validação para isso no atributo `duracaoMinutos`.

Uma possibilidade seria utilizar uma validação como `@Positive`, caso o projeto esteja utilizando Jakarta Validation.

---

# Documentario.java

Na classe `Documentario` foi encontrado um problema relacionado ao preço.

### Preço do documentário

De acordo com a regra do sistema, documentários devem ser gratuitos, ou seja, o preço deve ser **R$ 0,00**.

Porém, a classe não sobrescreve o método `calcularPrecoAluguel()`.

Com isso, ela acaba herdando o método da classe `Conteudo`, que retorna `R$ 9,90`.

O correto seria implementar o próprio método na classe:

```java
@Override
public double calcularPrecoAluguel() {
    return 0.0;
}
```

### Promoção

Nesse ponto, o comportamento está correto.

Documentários não devem participar de promoções e a classe não implementa `Promocionavel`.

### Relação com o problema da classe Conteudo

Esse erro também mostra por que seria melhor tornar `calcularPrecoAluguel()` abstrato na classe `Conteudo`.

Se o método fosse abstrato, o Java obrigaria a classe `Documentario` a implementar seu próprio cálculo, evitando que o preço de R$ 9,90 fosse herdado por engano.

---

# Filme.java

Na classe `Filme` foi encontrado um erro no cálculo da promoção.

### Erro no desconto

A regra diz que filmes devem ter **20% de desconto**.

Porém, o código está utilizando:

```java
preco * 1.2
```

Isso aumenta o preço em 20%, em vez de aplicar um desconto.

O correto seria:

```java
preco * 0.8
```

Assim, um filme de R$ 10,00, por exemplo, passaria a custar R$ 8,00.

### Preço de estreia

A parte responsável por adicionar R$ 5,00 quando o filme está em estreia está correta e atende à regra definida.

### Magic Numbers

Também existem alguns valores fixos diretamente no código, como:

```java
9.90
5.00
1.2
```

Seria melhor transformar esses valores em constantes. Por exemplo:

```java
private static final double TAXA_ESTREIA = 5.00;
private static final double DESCONTO_PROMOCIONAL = 0.8;
```

Isso deixa o código mais fácil de entender e de alterar no futuro.

---

# Serie.java

A classe `Serie` foi a que apresentou mais problemas.

## Problema no polimorfismo

O método foi criado dessa forma:

```java
calcularPrecoAluguel(double desconto)
```

Porém, na classe `Conteudo`, o método esperado é:

```java
calcularPrecoAluguel()
```

Como os métodos possuem assinaturas diferentes, o método da `Serie` não está sobrescrevendo o método da classe mãe.

Na prática, quando o sistema chamar:

```java
conteudo.calcularPrecoAluguel();
```

o método criado na `Serie` não será utilizado.

Isso pode fazer com que o sistema utilize o preço padrão de R$ 9,90 da classe `Conteudo`, quando deveria calcular o preço de acordo com o número de temporadas.

O ideal seria corrigir a assinatura e utilizar `@Override`:

```java
@Override
public double calcularPrecoAluguel() {
    return numeroTemporadas * 4.90;
}
```

## Problema no construtor

Outro problema está no construtor da classe.

Ele recebe informações como:

* título;
* categoria;
* duração;
* classificação etária;
* número de temporadas.

Porém, apenas o número de temporadas é atribuído.

O construtor deveria utilizar `super(...)` para enviar os dados necessários para a classe `Conteudo`.

Caso isso não seja feito, informações como título e categoria podem acabar ficando `null`.

---

## Clean Code

Também foram encontrados alguns pontos mais relacionados à organização do código.

### Comentários desnecessários

Existem comentários como:

```java
// cria a série com os dados recebidos
```

Esse tipo de comentário não ajuda muito, porque o próprio código já deixa claro o que está acontecendo.

Comentários são mais úteis quando explicam uma regra de negócio ou uma decisão que não seja óbvia apenas olhando o código.

### Magic Numbers

O preço de `4.90` também está diretamente no código.

Seria melhor criar uma constante:

```java
private static final double PRECO_POR_TEMPORADA = 4.90;
```

E depois utilizar:

```java
return numeroTemporadas * PRECO_POR_TEMPORADA;
```

Dessa forma fica mais fácil entender o que o valor representa.

### Promoção

Apesar dos problemas encontrados na classe, o cálculo da promoção está correto:

```java
preco * 0.8
```

Esse cálculo realmente representa um desconto de 20%.

---

# Resumo

| Classe         | Problema encontrado                              |
| -------------- | ------------------------------------------------ |
| `Conteudo`     | Atributo `duracaoMinutos` público                |
| `Conteudo`     | Preço padrão de R$ 9,90                          |
| `Conteudo`     | Uso de `instanceof`                              |
| `Conteudo`     | Falta de validação da duração                    |
| `Documentario` | Não possui preço próprio                         |
| `Documentario` | Pode herdar preço de R$ 9,90                     |
| `Filme`        | Promoção aumenta o preço em vez de dar desconto  |
| `Filme`        | Uso de Magic Numbers                             |
| `Serie`        | Método não sobrescreve corretamente a classe mãe |
| `Serie`        | Construtor não utiliza `super(...)`              |
| `Serie`        | Comentários desnecessários                       |
| `Serie`        | Uso de Magic Numbers                             |

## Conclusão

De forma geral, os principais problemas encontrados estão relacionados ao uso incorreto de herança e polimorfismo, algumas regras de negócio que não foram implementadas corretamente e alguns pontos de Clean Code.

As principais correções seriam:

* corrigir o desconto de `Filme`;
* fazer `Documentario` retornar R$ 0,00;
* corrigir o método de preço da `Serie`;
* corrigir o construtor da `Serie`;
* transformar `calcularPrecoAluguel()` em abstrato na classe `Conteudo`;
* melhorar o encapsulamento dos atributos;
* adicionar a validação da duração;
* substituir valores fixos por constantes;
* remover comentários que não agregam informação.

Com essas alterações, o código fica mais organizado, mais fácil de manter e também mais alinhado com os conceitos de POO e Clean Code.

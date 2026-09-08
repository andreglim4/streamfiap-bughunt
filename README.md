# Checkpoint 4 — Bug Hunt StreamFIAP

> Copie este arquivo para a raiz do seu repositório com o nome **README.md**
> e preencha todas as seções.

## Identificação

**Grupo:** ___

| Integrante | RM | Turma |
|---|---|---|
|Lucas Eiki Tanaka Gushikem | rm561607| 2CCPO|
|André Gouveia de Lima| rm564219 | 2CCPO|



| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 6 / 6 |

---

## Parte 1 — Bugs encontrados

> Uma linha por bug, na ordem em que você os encontrou. Use a numeração dos seus
> commits (`fix: bug01 ...`). Preencha TODAS as colunas — metade da nota está aqui.

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
| :--- | :--- | :--- | :--- | :--- |
| **01** | Aceitava cadastro com duração 0 ou negativa | `ConteudoController`: Falta de validação nos POSTs | Adicionado `if` validando se a duração é <= 0 | Validação de Dados |
| **02** | Documentário era cobrado a R$ 9,90 | `Documentario`: Não sobrescrevia o método da classe mãe | Adicionado `@Override` retornando 0.0 | Herança / Polimorfismo |
| **03** | Filme com promoção ficava mais caro | `Filme`: Multiplicação por 1.2 causava acréscimo | Alterado para `preco * 0.8` | Regra de Negócio |
| **04** | Série cobrava valor fixo de R$ 9,90 | `Serie`: Parâmetro extra quebrava a sobrescrita | Removido parâmetro e ajustado cálculo por temporada | Sobrescrita de Métodos |
| **05** | Série era salva com nome e categoria nulos | `Serie`: Construtor não repassava os dados à superclasse | Adicionada chamada `super(...)` no construtor | Encadeamento de Construtores |
| **06** | Falha ao cadastrar usuário (ID vazio/duplicado) | `Usuario`: Faltava autoincremento no ID | Adicionada anotação `@GeneratedValue` | Mapeamento ORM / JPA |
| **07** | Nome do usuário ficava nulo no banco | `Usuario`: Shadowing de variável no construtor (`nome = nome`) | Alterado para usar `this.nome = nome` | Palavra-chave `this` |
| **08** | Aluguel negado com saldo, e aceito sem saldo | `Usuario`: Validador invertido (`preco >= creditos`) | Alterado para `this.creditos >= preco` | Lógica Booleana |
| **09** | Permitido alugar conteúdo indisponível | `Usuario`: Falta de verificação condicional na flag `disponivel` | Adicionado `if (!c.isDisponivel())` lançando exceção | Validação de Estado |
| **10** | Promoções de 20% não eram aplicadas | `Usuario`: O aluguel chamava o preço base | Alterado para chamar `calcularPrecoPromocional()` | Delegação de Comportamento |
| **11** | Busca por categoria retornava vazio | `ConteudoController`: Uso de `==` para comparar `String` | Alterado para uso do método `.equals()` | Comparação de Objetos |
| **12** | Busca por ID inexistente retornava 200 OK vazio | `ConteudoController`: Bloco `try/catch` vazio engolia o erro | Removido o `try/catch` para a exceção subir | Tratamento de Exceções |

---


## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
| :--- | :--- | :--- | :--- |
| **01** | `Conteudo.java` | **Encapsulamento:** Atributo `duracaoMinutos` estava público. | Alterado para `private` e ajustado o acesso nos controllers via *Getter*. |
| **02** | `Filme.java` | **Magic Numbers:** Valores `9.90` e `5.00` soltos no código. | Extraídos para constantes `PRECO_BASE` e `TAXA_ESTREIA`. |
| **03** | `Usuario.java` | **Responsabilidade Mista:** Impressão de recibo (`System.out`) no domínio. | Removido bloco de I/O do model. |
| **04** | `ConteudoController` | **Performance/Anti-pattern:** Filtro de categoria feito na memória com `for`. | Delegado para o banco via query no `ConteudoRepository`. |
| **05** | `ConteudoController` | **Código Morto:** Métodos antigos e lógicas comentadas no arquivo. | Removido o método `calcularDescontoAntigo` e os comentários inúteis. |
| **06** | `AluguelController` | **Field Injection:** Uso de `@Autowired` direto em atributos (acoplamento). | Injeção alterada para ser feita via construtor da classe. |


---

## Parte 3 — Perguntas de reflexão

> Responda com suas palavras, 5 a 10 linhas cada, **usando o código real do projeto
> como exemplo**. Respostas genéricas de tutorial não pontuam.

### 1. Injeção de dependência (Aula 13)
Os controllers recebem os repositories via `@Autowired` (ex.: `ConteudoController`
usa `ConteudoRepository`). Explique por que o Spring precisa gerenciar esses objetos
em vez de criarmos com `new ConteudoRepository()`. O que exatamente o Spring faz ao
injetar um bean, e por que isso não funcionaria com um `new` comum?

R: O Spring gerencia os beans em um contêiner IoC (Inversion of Control) para promover o desacoplamento e facilitar a manutenção e os testes unitários. Quando o Spring injeta um bean, ele instancia a classe dependente e a atribui automaticamente (via construtor ou setter). Isso não funcionaria com um new manual porque a criação manual engessa o código, acoplando a classe a implementações concretas e fazendo com que o desenvolvedor precise gerenciar manualmente o ciclo de vida e as dependências de cada objeto, perdendo as vantagens do gerenciamento automatizado do framework.


### 2. JDBC vs Spring Data JPA (Aulas 12 e 13)
Na Aula 12 escrevemos um `ProdutoDAO` na mão com `Connection`, `PreparedStatement` e
`ResultSet`. Aqui o `ConteudoRepository` tem 2 linhas e faz CRUD completo. Compare as
duas abordagens: o que o Spring Data JPA automatiza, o que o JDBC/DAO ainda resolve
melhor, e como o `findByCategoria` consegue funcionar sem implementação.

R: O JDBC exige a abertura manual de conexões, escrita de comandos SQL crus (PreparedStatement), mapeamento manual de colunas para objetos e tratamento exaustivo de exceções de banco de dados (SQLException). Em contrapartida, o Spring Data JPA abstrai toda essa complexidade através de interfaces como JpaRepository, permitindo operações de CRUD completas e consultas personalizadas apenas por convenção de nomes ou anotações (@Query), aumentando a produtividade e reduzindo drasticamente a quantidade de código boilerplate.


### 3. Exceções checked vs unchecked (Aula 11)
A `ClassificacaoIndicativaException` estourava como um erro genérico do servidor,
sem mensagem útil para o cliente. Explique a diferença entre `extends Exception` e
`extends RuntimeException` no contexto desse bug, e como você fez a mensagem da
regra (classificação indicativa) chegar de forma clara ao cliente da API.

R: Exceções checked (como IOException) obrigam o compilador a tratá-las explicitamente com blocos try/catch ou a declará-las na assinatura do método (throws), sendo ideais para falhas recuperáveis do ambiente externo. Já as exceções unchecked (que herdam de RuntimeException, como IllegalArgumentException ou nossas exceções de negócio personalizadas) não exigem tratamento obrigatório em tempo de compilação, sendo recomendadas para erros de lógica de programação ou validações de regras de negócio que interrompem o fluxo de forma limpa até serem tratadas por um manipulador global (GlobalExceptionHandler).


### 4. Sobrescrita vs sobrecarga (Aula 7)
Um dos bugs compilava sem nenhum erro: o método da `Serie` parecia sobrescrever
`calcularPrecoAluguel`, mas na verdade sobrecarregava. Explique a diferença entre
override e overload nesse caso e por que a anotação `@Override` teria impedido o bug.

R: A sobrecarga (overloading) ocorre na mesma classe quando criamos métodos com o mesmo nome, mas com assinaturas (parâmetros) diferentes, permitindo flexibilidade na chamada das funções. Já a sobrescrita (overriding) ocorre quando uma classe filha redefine um método herdado de sua classe mãe mantendo a mesma assinatura, sendo o pilar fundamental para implementar o polimorfismo, permitindo que comportamentos específicos sejam executados dinamicamente em tempo de execução dependendo do tipo do objeto.


### 5. Onde blindar o objeto? (Aulas 3, 4 e 13)
Vimos bugs de dados inválidos aceitos (duração negativa, créditos negativos, campos
nulos). Em quais lugares (construtor, setter, método do model) cada tipo de validação
deve ficar? Justifique usando os bugs que você encontrou e explique por que validar só
em um lugar não foi suficiente.

R: O objeto deve ser blindado internamente em suas próprias classes de domínio (Model / Entity) por meio de construtores, modificadores de acesso restritos (private) e validações nos próprios métodos de modificação (setters ou regras de negócio). Isso garante que o objeto nunca aceite um estado inválido ("Fail-fast"), mantendo a consistência dos dados independentemente de qual camada da aplicação (controller ou serviço) esteja tentando manipulá-lo.


### 6. Abstração e interface (Aulas 8 e 9)
`Conteudo` é abstrata e `Promocionavel` é uma interface. Explique a diferença de
propósito entre as duas nesse projeto e o que mudaria no código se o Documentário
passasse a ter promoções — quais classes/linhas seriam tocadas e quais ficariam
intactas? O que isso diz sobre o design do sistema?

R:A abstração e as interfaces permitem definir contratos genéricos de comportamento sem amarrar a implementação a uma classe específica, seguindo o princípio aberto/fechado (OCP) do SOLID. No contexto do projeto, isso diferencia a regra geral da aplicação das particularidades de cada tipo de conteúdo ou fluxo, permitindo adicionar novas funcionalidades ou regras de negócio sem que seja necessário modificar o código já existente e testado nas camadas superiores.


---

## Parte 4 — Espaço livre (opcional)

Alguma dificuldade, dúvida ou comentário sobre o checkpoint?

```

```

---
title: "Streams em JAVA: Tudo que você precisa saber"
date: 2023-11-12T16:30:03+00:00
weight: 2
# aliases: ["/first"]
tags: ["JAVA","STREAMS"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Streams é um dos conceitos mais temidos e fundamentais da linguagem JAVA, mas não é tão difícil quanto parece, e aprender vai deixar seu código bem melhor, confira :)"
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/kauegatto/kauedotcat/content"
    Text: "Sugerir Alterações" # edit text
    appendFilePath: true # to append file path to Edit link
---
## Map, Filter, Reduce

Vamos começar com um exemplo?

```java
* Given a list of people
* We need to compute the average of the age of those people
* For the people older than 20
```

1. Nesse caso, é meio claro, começamos com um objeto pessoa, mas trabalharemos / transformaremos o dado de uma maneira que consigamos a idade (**map**). O map pega um objeto, e mapeia para outro, geralmente de tipo diferente


![Map, um objeto entra, é transformado (seta) e sai outro](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q8t11tcox29jfu7dcw0x.png)

2. Com isso, vamos filtrar o dado age, para que ele só compute a média de maiores de 20 anos. Isso é um **filter.** O filter literalmente filtra os dados, não os transforma (diferente de map) e decide se deve manter ou não o dado.
    2.1. Filters em código recebem uma predicate como parâmetro, ela representa uma função que recebe um argumento e retorna um valor booleano.

![Filter, entra idade, passa pelo filter (seta) e voltam apenas idades >20 ](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ycioms5b7brys8h07bz1.png)

3. Por fim, vamos calcular a média desse dado, já transformado de pessoas e filtado para que a idade seja >20. Essa operação avg é o reduce, nesse caso, vamos pensar nela a princípio como uma agregação (tipo min,max,count, etc).


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6tifjl5kvz87iiogbxzv.png)

| Operação | Comportamento |
| --- | --- |
| Map | Transforma o dado, mudando seu tipo. Não muda o número de elementos. |
| Filter | Não transforma o dado, reduz (ou mantém igual) o número de elementos |
| Reduce | Combina os elementos em um resultado único. É uma operação terminal que produz um único valor com base em uma operação de redução, como soma, média ou máximo. |

## Por que usar Streams?

```java
List<Person> people = ...;

int sum = 0;
int count = 0;
for (Person person: people) {
	if (person.getAge() > 20) {
		count++;
		sum += person.getAge();
	}
}
double average = 0d;

if(count > 0) {
	average = sum / count;
}
```

No exemplo acima, temos um código que **descreve com detalhes o que deve ser feito para chegarmos em um resultado** (o resultado do exemplo apresentado), e se mudarmos o algoritmo, precisamos mudar o código, mesmo que o algoritmo seja o mesmo. Não necessariamente precisa ser assim, em SQL, por exemplo, escreveríamos:

```sql
Select AVG(age) from People
Where People.age>20
```

**Note que no exemplo acima, descrevemos ao código o que queremos que seja feito, as premissas, descrevemos o resultado, e não como o resultado deve ser computado.**

## Collection X Streams

Tudo certo, se o código faz parte de uma lista ou qualquer coleção, podemos tentar implementar assim:

```java
List<Person> people = ...;
double average = 
people.map(person -> person.getAge())
	.filter(age — > age > 20)
	.average();
```

**ERRADO!** A API de collections não provê esse tipo de operações, o código acima não compila 😄!

### Mas por que a API de Collections e Streams são separadas?

Imagine que essas operações são feitas em uma lista de 1.000.000 de pessoas! Portanto, após o primeiro map, você teria uma Lista com 1.000.000 de inteiros de idade (porque o map não reduziria o tamanho em si), acho que já deu para perceber que duplicar a collection vai ser altamente custoso para o processador e para a memória

<aside>
💡 **Um objeto de stream oferece operações de map / filter / reduce sem duplicação e com máxima eficiência!**

</aside>

Mas como isso funciona? Quando usamos `pessoas.stream()`, retornamos uma `Stream<Pessoa>`. Por definição, um objeto de stream não carrega dados, é grátis cria-lo.

> A collection is an in-memory data structure to hold values and before we start using collection, all the values should have been populated. Whereas a java Stream is a data structure that is computed on-demand. Java Stream doesn’t store data, it operates on the source data structure (collection and array) and produce pipelined data that we can use and perform specific operations.  

https://www.digitalocean.com/community/tutorials/java-8-stream
> 

## Operações Intermediárias X Finais

Streams atuam sobre coleções para realizar map / filter / reduce, operações intermediárias são operações sobre streams que retornam streams ( `Stream<T> map()`  → Método de stream, retorna stream). 

Operações finais são operações que retornam a coleção de novo, depois do processamento  / pipelining acabar `toList()` é um exemplo

## E como ficaria o código

Aqui está um exemplo de como ficaria o código.

```java
List<Pessoa> pessoas = new ArrayList<>();

	pessoas.stream()
				 .map(...)
				 .filter(...)
				 .average();
```

<aside>
💡 Importante: Você não pode processar a mesma stream duas vezes:

```java
Stream<String> personNames = personStream.map(Person::getName);
Stream<String> emptyNames = personNames.filter( name -> name.isEmpty())
Stream<String> notEmptyNames = personNames.filter( name -> !name.isEmpty())
```

Esse código quebrará, tendo em vista que estamos processando a mesma stream personNames duas vezes!

```java
Stream<String> personNames = personStream.map(Person::getName);
Stream<Integer> personAges = personStream.map(Person::getAge);
```

Nesse caso, estamos aplicando o map para criar duas streams diferentes, não quebrará o código.

```java
Stream<String> emptyNames = personStream.map(Person::getName).filter(name -> name.isEmpty());
Stream<String> notEmptyNames = personStream.map(Person::getName).filter(name -> !name.isEmpty());
```

Esse código, processando duas streams diferentes também funcionará

Por favor, não crie variáveis para trabalhar com streams, fiz por didática

</aside>

## FlatMapping

Flatmapping funciona para lidar com relações 1:N

Exemplo: Cidades, onde temos várias pessoas por cidade.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/z2j5fpkzolyphphn5t1y.png)

Suponha que queremos pegar todas as pessoas, independente de suas cidades. Nesse caso, nos preocupamos com a entidade relacionada, não com as cidades em si**. O flatmap faz isso, pega uma entidade Cidade e retorna para nós uma `Stream<Pessoas>`.**

```java
List<City> cities = ...;

Function<City, Stream<Person>> flatMapper = 
	city -> city.getPeople().stream();

long count = 
	cities.stream()
	.flatMap(flatMapper) /*poderíamos colocar city -> city.getPeople().stream() direto*/
	.count()
```

<aside>
💡 Uma operação de flatmap recebe um objeto e retorna uma stream de outros objetos

```java
List<String> words "Gomu","Gomu", "No", "Mi");

Stream<String> streamStream =
words.stream()
.map(w -> w.split("")) // Stream<String[]>
.flatMap(Arrays::stream) ; // Stream<String>
```

O flatmap funciona nesse caso, pois ao aplicar **`flatMap(Arrays::stream)`**, você está dizendo ao Java para pegar cada array de caracteres da **`Stream<String[]>`** e transformá-lo em uma stream de strings (**`Stream<String>`**) usando o método estático **`Arrays.stream()`**.

</aside>

## Streams a partir de RegEx

```java
String sentence = "the quick brown fox jumps over the lazy dog";
String[] words = sentence. split(
Stream<String> wordsStream = Arrays.stream(words);
Ion count = wordsStream.count()
```

Ao fazermos dessa forma, criamos um array intermediário, perdemos o propósito de streams.

Podemos fazer:

```java
Pattern pattern = Pattern.compile(" ");
long count = pattern.sp1itAsStream(sentence) . count();
```

## Streams de outro tipo

Em java, existem interfaces de stream feitas para manipular certos primitivos mais adequadamente, um exemplo disso é a `IntStream`

### IntStream

**`IntStream`** tende a ser mais eficiente em termos de espaço e desempenho quando você está trabalhando com valores inteiros primitivos, uma vez que evita a criação de objetos **`Integer`**.

A **`IntStream`** oferece operações especializadas para trabalhar com valores inteiros, como **`sum()`**, **`average()`**, **`range()`**, **`rangeClosed()`**, etc. Uma **`Stream<Integer>`** fornece operações de stream **genéricas** aplicáveis a **objetos**, mas é menos eficiente quando se trata de cálculos com tipos primitivos.

<aside>
💡 Além do IntStream, também existem `LongStream`e `DoubleStream`(Além da `Stream<T>`)

</aside>

## Refactoring → Exemplo com múltiplas tarefas ao mesmo tempo

Streams não são uma boa para realizarem múltiplas tarefas simultaneamente, portanto se temos um for loop que agrega valores à 3 variáveis diferentes, precisamos dividir em 3 for’s (repetidos) e trocar cada um deles por uma stream. Claro, isso diminui muito a performance, mas em um for com poucos elementos a serem iterados, será basicamente imperceptível.

```java
public String statement() {

	double totalAmount = rentals.stream()
		.mapToDoub1e(this::computeRenta1Amount)
		.sum();
	int frequentRenterPoints = rentals.stream()
	.mapToInt (this::getFrequentRenterPoints)
	.sum();

	String statement = composeHeader();

	statement += rentals.stream()
	.map(this::computeStatementLine)
	.collect(Collectors.joining());
	
}
```

## Reduções

Antes, ouvimos que redução era algo similar a uma agregação do SQL, vamos continuar com isso em mente, mas como isso funciona?

Vamos ter como exemplo a soma, a soma é uma uma implementação de `BinaryOperator<Integer>`, que pega dois elementos e os soma ( `sum = (i1,i2) -> i1+i2;`). Isso acontece com dois elementos por vez, associando os valores e depois somando com o outro.

## Reduções de uma stream vazia

**Mas e qual a redução de uma Stream vazia?** A **redução de uma Stream vazia é o seu elemento identidade (identity),** um elemento identidade é um valor pré-definido que atua como ponto de partida em uma operação de redução. Em contextos de programação funcional, quando não há elementos para combinar na Stream, a operação de redução retorna o elemento identidade como resultado, garantindo que a operação seja definida e não cause exceções em casos de Stream vazia. Isso é especialmente útil para lidar com casos em que não há dados disponíveis para a operação específica, permitindo que o código se comporte de maneira previsível e segura.

Por exemplo, o identity de uma soma é 0, tendo em vista que o 0 não impactará no valor final, para múltiplicação, um.

### Max, Min, Avg?

O "identity element" ou valor inicial para as operações de **`max`**, **`min`** e **`average`** em Streams do Java é um pouco diferente porque essas operações retornam um **`Optional`**, que pode ser vazio (caso a Stream esteja vazia) ou conter um valor, portanto, essas operações não tem valores identity.

Portanto, para acessar max, min e avg, você deve fazer algo do tipo:

```java
Optional<Integer> max = numbers.stream().max(Integer::compareTo);
Optional<Integer> min = numbers.stream().min(Integer::compareTo);
OptionalDouble average = numbers.stream().mapToDouble(Integer::doubleValue).average();

	System.out.println("Max: " + max.orElse(null));
	System.out.println("Min: " + min.orElse(null));
	System.out.println("Average: " + average.orElse(Double.NaN));
```

### Como podemos escrever nossas reductions?

Talvez você esteja se perguntando o porquê de entendermos também a fundamentação de identity elements, se ainda não ficou claro, vai ficar agora:

A operação **`reduce`** em Streams do Java tem dois parâmetros principais:

1. O valor inicial (identity element): Já foi explicado
2. Uma função de acumulação (accumulator function): Esta função é usada para combinar os elementos da Stream em um único resultado. A função deve ser uma expressão lambda ou um método de referência que aceite dois argumentos e retorne um resultado. Em soma, seria 
`(a, b) -> a + b`

<aside>
💡 Se você não passar um identity correto, o resultado estará incorreto.

Se sua stream não possuir um identity, você pode usar uma outra sobrecarga do método que não recebe o identity, mas retorna um Optional. Use-o somente se não possuir o identity…

</aside>

## Reduções em um container mutável

Reduções em um container mutável referem-se à aplicação de operações de redução, como soma, multiplicação, média, entre outras, a elementos armazenados em um contêiner que pode ser alterado durante o processo de redução.  Um "container" neste contexto pode ser definido como uma estrutura de dados flexível que permite armazenar e modificar elementos de forma dinâmica (Lists, Maps, etc.). Portanto, podemos simplificar reduções em containers mutáveis como reduções em coleções.

Ao chamar **`.stream()`** em uma lista, você está criando uma Stream dos elementos contidos na lista. A Stream é uma sequência de elementos que pode ser processada de maneira funcional, **mas não modifica a lista original.** Quando você chama **`.max()`**,  por exemplo, está solicitando o elemento máximo com base em algum critério da lista, mas a lista em si permanece a mesma.

### Coletores!

```java
List<Person> pessoasDaBaixada = new ArrayList() ;
pessoasDaBaixada.stream()
.filter(p -> p.getDDD().equals("013"))
.forEach(p -> pessoasDaBaixada.add(p));
```

Mas esse exemplo acima não é muito diferente do que já vimos, por que estamos focando nesse tipo de redução?

Para filtrar e coletar elementos em uma nova lista usando a API de Streams do Java, você deve usar um **coletor** adequado, como **`Collectors.toList()`**

Os coletores permitem que você capture os resultados das operações de redução em coleções ou outros tipos de dados concretos. **Em outras palavras, eles transformam os elementos processados em uma Stream em uma coleção real que pode ser usada e manipulada posteriormente.**

```java
List<Person> pessoasDaBaixada = new ArrayList<>();

// Supondo que o número de DDD seja uma String
List<Person> pessoasComDDD13 = pessoasDaBaixada.stream()
    .filter(p -> "13".equals(p.getDDD()))
    .collect(Collectors.toList());
	//.toList() retornará uma lista imutável!
```

> *”O novo método Stream.toList() não produz nem uma lista não modificável nem é um atalho para `collect(toUnmodifiableList())`, porque `toUnmodifiableList()` **não aceita valores nulos**. A implementação de Stream.toList() não é limitada pela interface `Collector`; portanto, `Stream.toList()` aloca menos memória. Isso a torna ideal para uso quando o tamanho da stream é conhecida antecipadamente.”*  [Link para comentário no stackoverflow](https://stackoverflow.com/questions/65969919/differences-of-java-16s-stream-tolist-and-stream-collectcollectors-tolist#comment121174778_65991907)
> 

### **Exemplos de Outros Coletores**

- **`Collectors.toSet()`**: Cria um conjunto a partir dos elementos da Stream, removendo duplicatas.
- **`Collectors.toMap(keyMapper, valueMapper)`**: Cria um map a partir dos elementos da Stream, usando funções de mapeamento para extrair chaves e valores.
- **`Collectors.groupingBy(classifier)`**: Agrupa elementos da Stream com base em um critério definido pela função de classificação.
- **`Collectors.joining(delimiter)`**: Concatena os elementos da Stream em uma única String usando um delimitador.
- **`Collectors.summingInt()`** ou **`Collectors.summingLong()`**: Calcula a soma dos valores inteiros ou longos de elementos da Stream.

### Exemplos de uso:

```java
Map<Integer, List<Person>> pessoasPorIdade = pessoas.stream()
    .collect(Collectors.groupingBy(Person::getIdade));
```

```java
List<Person> pessoas = Arrays.asList(
    new Person("Jorge", 25),
    new Person("Ben", 30),
    new Person("Jor", 35)
);

Map<String, Integer> mapNomeIdade = pessoas.stream()
    .collect(Collectors.toMap(Person::getNome, Person::getIdade));
```

<aside>
💡 Não necessariamente precisamos de collectors para trabalhar com coleções, mas eles tornam a vida mais fácil, veja os dois exemplos anteriores:

</aside>

```java

// ex1
Map<Integer, List<Person>> pessoasPorIdade = new HashMap<>();
pessoas.forEach(person -> {
    Integer idade = person.getIdade();
    List<Person> pessoasComIdade = pessoasPorIdade.get(idade);
    if (pessoasComIdade == null) {
        pessoasComIdade = new ArrayList<>();
        pessoasPorIdade.put(idade, pessoasComIdade);
    }
    pessoasComIdade.add(person);
});

// ex2
Map<String, Integer> mapNomeIdade = new HashMap<>();
pessoas.stream().forEach(person -> mapNomeIdade.put(person.getNome(), person.getIdade()));
```

## Pincelando: Streams Paralelas
À medida que exploramos as reductions e operações terminais, é crucial considerar o potencial das streams paralelas em Java, mas o que são? 🤔

Streams paralelas oferecem a capacidade de executar operações em paralelo, aproveitando múltiplos núcleos da CPU, o que melhora muito o desempenho da stream.

Apesar disso, vale dizer que operações em paralelo nem sempre podem ser consideradas em uma stream, suponha que você esteja calculando uma média com uma reduction, para calcularmos a média, precisamos primeiro somar os elementos, e então dividirmos, se somarmos e depois dividirmos partes distintas, isso trará um **resultado incorreto, isso significa que a média é uma operação não associativa!**

### Operações Associativas 
São operações em que a ordem em que os elementos são combinados não afeta o resultado. Exemplos comuns de operações associativas incluem soma e multiplicação.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
// Cálculo da soma em uma stream paralela
int sum = numbers.parallelStream().reduce(0, Integer::sum);
Esse parágrafo tem como objetivo apenas introduzir e notificar a existência de streams paralelas, procure saber mais sobre elas!
```

### Operações Não-Associativas
São o oposto!

# Conclusão e Agradecimento

Obrigado pela leitura, espero que tenha sido produtiva e que você tenha todo o conhecimento necessário para conseguir dar seus próprios passos e construir streams funcionais e avançadas, caso necessário.

# Referências

https://app.pluralsight.com/library/courses/692a1310-42db-4f3c-a33b-208a55b7bd84/table-of-contents

https://www.digitalocean.com/community/tutorials/java-8-stream

[Maratona Java Virado no Jiraya](https://www.youtube.com/playlist?list=PL62G310vn6nFIsOCC0H-C2infYgwm8SWW)

https://acervolima.com/coletores-java/
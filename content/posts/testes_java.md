---
title: "[WIP] Testes em Java - JUnit, Mockito, Integração e TestContainers"
date: 2024-01-03T16:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["JAVA","SPRING","TESTES"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Vamos explorar como fazer testes em java!"
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

Aviso - Algumas imagens estarão quebradas aqui, enquanto for um trabalho em progresso, recomendo a leitura [aqui!](https://kauegatto.notion.site/WIP-Testes-c79bc9928c4d44d4979f59d127012e6c?pvs=4)

# Spring + Testing

1. Configuração do Maven:
    - Certifique-se de que o Maven esteja instalado em seu sistema.
    - No arquivo **`pom.xml`** do seu projeto, adicione as dependências necessárias para JUnit e o suporte de testes do Spring Boot. Normalmente, essas dependências (normalmente **spring-boot-starter-test)** já estão incluídas no arquivo de modelo gerado pelo Spring Initializr ao criar um projeto Spring Boot.
2. Estrutura de diretórios:
    - No diretório do seu projeto, crie a estrutura de diretórios padrão para testes: **`src/test/java`** e **`src/test/resources`**.
    - Os testes de unidade devem ser colocados no diretório **`src/test/java`** seguindo a mesma estrutura de pacotes do código-fonte principal.
3. Criação de testes:
    - Crie classes de teste, essas classes de teste são geralmente nomeadas de acordo com a classe que estão testando, seguidas por "Test". Exemplo: **`UserService`**,  →**`UserServiceTest`**.
    
    - Anote a classe de teste com **`@RunWith(SpringRunner.class)`** para permitir a execução do teste no contexto do Spring Boot.
    - Anote a classe de teste com **`@SpringBootTest`** para carregar o contexto do Spring Boot durante a execução do teste.
    - Injete as dependências necessárias
    - Crie métodos de teste usando a anotação **`@Test`** e implemente a lógica de teste dentro desses métodos.
        - Se esse teste lançar uma exceção, defina a propriedade expected na annotation.
        - 

# JUnit - Testes de Unidade

- Testes de unidade são testes feitos para cobrir um comportamento ou unidade específica de execução dentro de um método.
    - Seu método de iniciar conversação pode ter vários e-se, nesse caso, um teste de unidade seria criado para validar cada e-se
- Testes de unidade, de maneira geral são executados com a biblioteca `JUnit`, que será o foco desse capítulo. Para sua utilização, criamos um método que retorna `void` e é anotado por `@Test`, esse método deve conter operações e validações (assertions) para garantir o retorno esperado.

```java
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class CalculadoraTest {

	@Test
  @DisplayName("Testando soma")
	public void testSoma() {
	    // Arrange (preparação)
	    Calculadora calculadora = new Calculadora();
	
	    // Act (ação)
	    int resultado = calculadora.soma(2, 3);
	
	    // Assert (verificação)
	    assertEquals(5, resultado);
	}
	
	@Test
	public void testSubtracao() {
	    // Arrange
	    Calculadora calculadora = new Calculadora();
	
	    // Act
	    int resultado = calculadora.subtracao(5, 2);
	
	    // Assert
	    assertEquals(3, resultado);
	}

	@Disabled("Esperando resolverem o bug que 3 é diferente de 1")
	@Test
	public void testNaoSeiOq(){
		  assertEquals(3,1);
  }
}
```

- Normalmente organizamos nossos testes de unidade em 3 etapas: given-when-then ou o triplo A: Arrange, Act & Assert

## **Assertions**

```java
// Padrão: 
assertEquals(2, calculadora.soma(1, 1));
assertEquals(4, calculadora.multiplica(2, 2), "The optional failure message is now the last parameter");
assertTrue('a' < 'b');
// Exceptions
Exception exception = assertThrows(Aritmetica.class, () -> calculadora.divide(1, 0));
assertEquals("/ por zero", exception.getMessage()); // opcional
```

Existem também bibliotecas focadas em gerar assertions mais legíveis/fluentes, como `AssertJ` e `Hamcrest`, podemos usa-las em conjunto com JUnit:

```java
import static org.assertj.core.api.Assertions.*;
assertThat(frodo.getName()).isEqualTo("Frodo");
assertThat(frodo).isNotEqualTo(sauron);
assertThat(fellowshipOfTheRing).hasSize(9)
                               .contains(frodo, sam)
                               .doesNotContain(sauron);

```

## Testes Parametrizados

Testes parametrizados permitem a execução do “mesmo teste”, mas com inputs diferentes, com bem mais facilidade do que criar testes separados:

```java
import org.junit.Test;
import org.junit.runner.RunWith;

import java.util.Arrays;
import java.util.Collection;
import static org.junit.Assert.assertEquals;

public class CalculadoraTest {

    @ParameterizedTest
	  @CsvSource({"2, 3, 5",
								"5, 2, 3",
								"0, 0, 0",
								"-1, 1, 0"})
    public void testSoma(int inputA, int inputB, int expectedResult) {
       // Arrange
        Calculadora calculadora = new Calculadora();
        // Act
        int resultado = calculadora.soma(inputA, inputB);
        // Assert
        assertEquals(expectedResult, resultado);
    }
}
```

Note que passamos os valores usando `@CsvSource`, mas não é a única maneira

### @ValueSource

Uma das maneiras mais simples de realizar testes parametrizados no JUnit 5 é usando a anotação **`@ValueSource`**. Esta anotação permite especificar um único vetor de valores literais para fornecer argumentos a um método de teste parametrizado.

Os **principais tipos** são suportados**:** int, long, float, String, Class, etc:

```java
javaCopy code
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;
import static org.junit.jupiter.api.Assertions.assertTrue;

public class ExemploTest {

    @ParameterizedTest
    @ValueSource(ints = {1, 2, 3})
    void testComValueSource(int argumento) {
        assertTrue(argumento > 0 && argumento < 4);
    }
}
```

### **@NullSource, @EmptySource**

Para testes de fronteira, ou input ruins pode ser útil passar valores nulos ou empty para nossos testes parametrizados:

1. **@NullSource: F**ornece um argumento nulo para o método de teste parametrizado anotado. **Não pode ser usada para parâmetros que têm um tipo primitivo.**
2. **@EmptySource:** Fornece um argumento vazio para o teste parametrizado. Ela pode ser utilizada para parâmetros de tipos como **`java.lang.String`**, **`java.util.Collection`**, **`java.util.List`**, **`java.util.Set`**, **`java.util.Map`**, e arrays primitivos e de objetos.
3. **@NullAndEmptySource:**
Esta é uma anotação composta que combina as funcionalidades de **`@NullSource`** e **`@EmptySource`**.

Se precisarmos fornecer vários tipos de strings em branco para um teste parametrizado, podemos usar **`@ValueSource`** da seguinte maneira:

```java

@ValueSource(strings = {" ", "   ", "\t", "\n"})
```

Podemos combinar **`@NullSource`**, **`@EmptySource`** e **`@ValueSource`:**

```java
@ParameterizedTest
@NullAndEmptySource
@ValueSource(strings = {" ", "   ", "\t", "\n"})
void testNullEmptyAndBlankStrings(String texto) {
    assertTrue(texto == null || texto.trim().isEmpty());
}
```

### @MethodSource

A anotação **`@MethodSource`** permite referenciar métodos na própria classe de teste ou em classes externas. Esses métodos devem ser estáticos. Cada método deve gerar uma sequência de argumentos. No final das contas, teremos uma `Stream<Arguments>`, mas podemos enviar uma ArrayList<Integer>, Arrays ou qualquer coisa que faça sentido e deixar o JUnit se virar 🙂.

Exemplo:

```java
@ParameterizedTest
@MethodSource("stringProvider")
void testComMethodSource(String argument) {
    assertNotNull(argument);
}

static Stream<String> stringProvider() {
    return Stream.of("apple", "banana");
}
```

Se um método parametrizado tiver diversos parâmtros, você precisará retornar uma collection, stream ou array de instâncias de `Arguments`, ou um arrays de objetos: 

```java
@ParameterizedTest
@MethodSource("stringIntAndListProvider")
void testWithMultiArgMethodSource(String str, int num, List<String> list) {
    assertEquals(5, str.length());
    assertTrue(num >=1 && num <=2);
    assertEquals(2, list.size());
}

static Stream<Arguments> stringIntAndListProvider() {
    return Stream.of(
        arguments("apple", 1, Arrays.asList("a", "b")),
        arguments("lemon", 2, Arrays.asList("x", "y"))
    );
}
```

Podemos usar métodos estáticos de outras classes como `MethodSource` dessa forma:

```java
@MethodSource("example.StringsProviders#tinyStrings")
```

### **@CsvSource**

A anotação **`@CsvSource`** permite expressar listas de argumentos como valores separados por vírgulas. Cada string fornecida representa um registro CSV e resulta em uma invocação do teste parametrizado.

O delimitador padrão é a vírgula, mas pode ser alterado. 

As aspas simples são usadas como aspas reais, servem para indicar que aquilo dentro é um texto, mas isso também pode ser configurado.

| Example Input | Resulting Argument List |
| --- | --- |
| @CsvSource({ "apple, banana" }) | "apple", "banana" |
| @CsvSource({ "apple, 'lemon, lime'" }) | "apple", "lemon, lime" |
| @CsvSource({ "apple, ''" }) | "apple", "" |
| @CsvSource({ "apple, " }) | "apple", null |
| @CsvSource(value = { "apple, banana, NIL" }, nullValues = "NIL") | "apple", "banana", null |
| @CsvSource(value = { " apple , banana" }, ignoreLeadingAndTrailingWhitespace = false) | " apple ", " banana" |

Exemplo:

```java
@ParameterizedTest
@CsvSource({
    "apple,         1",
    "banana,        2",
    "'lemon, lime', 0xF1",
    "strawberry,    700_000"
})
void testComCsvSource(String fruit, int rank) {
    assertNotNull(fruit);
    assertNotEquals(0, rank);
}
```

Talvez a magia do csv source, além da leitura, seja a possibilidade de implementar diversos testes via um arquivo csv localizado no sistema de arquivos local e/ou classpath

`@CsvFileSource(resources = "/custom-csv.csv")`

# Mocking

Tanto em testes de integração quanto de unidade, percebemos que devemos travar alguns valores, ou seja, se estamos testando uma regra de negócio dentro de um service, gostaríamos que nosso repositório sempre estivesse correto, assumimos isso como verdade para que nosso teste de unidade seja realmente de unidade.

Não indo muito longe, é fácil criar mocks usando a biblioteca mockito:

```java
import static org.mockito.Mockito.*;

// (4.10+)
List mockedList = mock();
// em versões mais antigas: List mockedList = mock(List.class);

// Com um objeto mockado, não recebemos exceptions:
mockedList.add("one");
mockedList.clear();

// Com um objeto mockado, sabemos tudo que aconteceu:
verify(mockedList).add("one");
verify(mockedList).clear();
```

```java
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import static org.mockito.Mockito.when;

public class UserServiceTest {

    @Mock
    UserRepository repository;

    @InjectMocks
    UserService service;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    void test() {
        when(repository.findById("123")).thenReturn(new User()); // mockado :)
    }
	}
```

1. **Configurando Comportamento:**
    - Utilize **`when()`** ou **`given()`** para especificar como o mock deve se comportar.
    - Se as respostas padrão não atenderem, implemente sua própria lógica estendendo a interface **`Answer`** e atribua ao mock.
2. **`spy()` e `@Spy`:**
    - Utilize **`spy()`** para criar um spy, é quase que um mock parcial, que chama os métodos reais, mas ainda pode ser verificado e manipulado.
3. **`@InjectMocks`:**
    - Anote a classe de teste com **`@InjectMocks`** para a injeção automática de mocks e spies anotados com **`@Mock`** ou **`@Spy`**.
4. **`verify()`:**
    - Use **`verify()`** para verificar se os métodos foram chamados com os argumentos esperados.
    - Você pode ser mais flexível usando coisas como **`any()`.**

# Testando Repositórios (De verdade!)

- Podemos testar repositórios de alguns jeitos, uma das maneiras é utilizar um banco em memória. Para SQL, h2 é fácil e rápido, podemos fazer as configs no banco usando um application-properties para os testes e validar tudo bonitinho normalmente 🙂. Outra alternativa seria utilizar TestContainers

## TestContainers

> "Testcontainers é um framework de código aberto para fornecer instâncias descartáveis e leves de bancos de dados, message brokers, browsers ou praticamente qualquer coisa que possa ser executada em um container Docker.”
> 
- Adicionando dependências:

```xml
<dependency>
	<groupId>org.testcontainers</groupId>
	<artifactId>mongodb</artifactId>
	<version>1.19.2</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.testcontainers</groupId>
	<artifactId>testcontainers</artifactId>
	<version>1.19.2</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.testcontainers</groupId>
	<artifactId>junit-jupiter</artifactId>
	<version>1.19.2</version>
	<scope>test</scope>
</dependency>
```

A primeira dependência irá variar de acordo com suas necessidades, dependendo do que precisar subir, no meu caso, só o MongoDB, se precisasse de RabbitMQ, também o adicionaria, por exemplo.

No código, é simples, depois de termos as dependências configuradas corretamente, podemos adicionar e instanciar os containers passando a `tag` da imagem docker como parâmetro, para o mongodb, `mongo:latest`.

1. Adicionar o Container:   `private static final MongoDBContainer MONGO_DB_CONTAINER = new MongoDBContainer("mongo:latest");`
2.  Adicionar `@Container` à variável do container
3. Se estiver usando Spring 3.1+, adicionar a anotação `@ServiceConnection` também. Essa configuração pega os dados do container criado e automaticamente sobrescreve com essas informações o container que seria usado originalmente
    1. Caso contrário, precisamos fazer isso na mão: crie um método que recebe como parâmetro `DynamicPropertyRegistry` 
    2. Anote esse método com `@DynamicPropertySource`
    3. Altere as propriedades que estavam configuradas anteriormente usando `registry.add("propriedade", valor)`

```java
@Container
private static final MongoDBContainer MONGO_DB_CONTAINER = new MongoDBContainer("mongo:latest");
@Container
private static final RabbitMQContainer RABBIT_MQ_CONTAINER = new RabbitMQContainer("rabbitmq:management");
@DynamicPropertySource
static void mongoDbProperties(DynamicPropertyRegistry registry) {
  MONGO_DB_CONTAINER.start();
  registry.add("spring.data.mongodb.uri", MONGO_DB_CONTAINER::getReplicaSetUrl);
  registry.add("spring.rabbitmq.addresses",() -> "amqp://guest:guest@localhost:"+RABBIT_MQ_CONTAINER.getAmqpPort());
}
```

A partir daí, siga sua vida com seu Rabbit, Mongo ou qualquer outra instância descartável.

```java
@Testcontainers
@SpringBootTest
public class TicketRepositoryJPATest {
  @Autowired
  private TicketRepositoryJPA ticketRepository;
  @Autowired
  private TicketFactory ticketFactory;

  @Container
  // @ServiceConnection spring 3.1+ - makes DynamicPropertySource unnecessary
  private static final MongoDBContainer MONGO_DB_CONTAINER = new MongoDBContainer("mongo:latest");
  @DynamicPropertySource
  static void mongoDbProperties(DynamicPropertyRegistry registry) {
    MONGO_DB_CONTAINER.start();
    registry.add("spring.data.mongodb.uri", MONGO_DB_CONTAINER::getReplicaSetUrl);
  }

  @Test
  @Order(0)
  void InsertTicket_Success(){
    Ticket t = ticketFactory.createTicket("validemail@gmail.com", "I have a problem", "hellp");
    ticketRepository.insert(t);
  }
  @Test
  @Order(1)
  void FindAll_findsOne(){
    var tickets = ticketRepository.findAll();
    assertEquals(tickets.size(), 1);
  }
}
```

No caso acima faço um teste simples de repositório onde garanto que alguns métodos estão sendo executados corretamente, mas poderia por exemplo, fazer um teste de integração que garante service + repositório, ou até mesmo um teste de integração completo com `TestRestTemplate`.

<aside>
💡 Nesse caso, note que criei a instância do container como um método estático à nível da classe, apesar disso, essa não é a única abordagem:
Se tivermos:
Se tivermos:

</aside>

### Importanto classes de declaração de Testcontainer

Um padrão comum ao usar o Testcontainers é declarar instâncias de **`Container`**como campos estáticos. Frequentemente, esses campos são definidos diretamente na classe de teste. Eles também podem ser declarados em uma classe pai ou em uma interface que o teste implementa:

```java
public interface MyContainers {
	@Container
	MongoDBContainer mongoContainer = new MongoDBContainer("mongo:5.0");
	
	@Container
	Neo4jContainer<?> neo4jContainer = new Neo4jContainer<>("neo4j:5");
}
```

Para mais discussões, não focando só no setup, mas em configurações diferentes, prós e contras, recomendo esse post: https://maciejwalkowiak.com/blog/testcontainers-spring-boot-setup/

Outra ideia interessante é que podemos configurar TestContainers para aplicações rodando em desenvolvimento, vai servir como um docker compose que não precisamos rodar. É legal, mas não gosto muito da abordagem pois usar docker-compose se tornou parte comum dia a dia de muitos devs e possuí fácil leitura e troubleshooting. Se te animar, para explorar esse ponto: https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.testing.testcontainers.at-development-time

## Referências:

https://spring.io/blog/2023/06/23/improved-testcontainers-support-in-spring-boot-3-1

https://howtodoinjava.com/spring-boot/testcontainers-with-junit-and-spring-boot/

https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.testing.testcontainers

# Testes de Integração - TestRestTemplate

### TestRestTemplate

`TestRestTemplate` is a convenience alternative to Spring’s `RestTemplate` that is useful in integration tests. You can get a vanilla template or one that sends Basic HTTP authentication (with a username and password). In either case, the template is fault tolerant. This means that it behaves in a test-friendly way by not throwing exceptions on 4xx and 5xx errors. Instead, such errors can be detected through the returned `ResponseEntity` and its status code.

|  | Spring Framework 5.0 provides a new WebTestClient that works for https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.testing.spring-boot-applications.spring-webflux-tests and both https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.testing.spring-boot-applications.with-running-server. It provides a fluent API for assertions, unlike TestRestTemplate. |
| --- | --- |

It is recommended, but not mandatory, to use the Apache HTTP Client (version 5.1 or better). If you have that on your classpath, the `TestRestTemplate` responds by configuring the client appropriately. If you do use Apache’s HTTP client, some additional test-friendly features are enabled:

- Redirects are not followed (so you can assert the response location).
- Cookies are ignored (so the template is stateless).

`TestRestTemplate` can be instantiated directly in your integration tests, as shown in the following example:

**JavaKotlin**

`class MyTests {

    private final TestRestTemplate template = new TestRestTemplate();

    @Test
    void testRequest() {
        ResponseEntity<String> headers = this.template.getForEntity("https://myhost.example.com/example", String.class);
        assertThat(headers.getHeaders().getLocation()).hasHost("other.example.com");
    }

}`

Alternatively, if you use the `@SpringBootTest` annotation with `WebEnvironment.RANDOM_PORT` or `WebEnvironment.DEFINED_PORT`, you can inject a fully configured `TestRestTemplate` and start using it. If necessary, additional customizations can be applied through the `RestTemplateBuilder` bean. Any URLs that do not specify a host and port automatically connect to the embedded server, as shown in the following example:

```java
import org.junit.jupiter.api.Test;

import java.time.Duration;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.http.HttpHeaders;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class MySpringBootTests {

    @Autowired
    private TestRestTemplate template;

    @Test
    void testRequest() {
        HttpHeaders headers = this.template.getForEntity("/example", String.class).getHeaders();
        assertThat(headers.getLocation()).hasHost("other.example.com");
    }

    @TestConfiguration(proxyBeanMethods = false)
    static class RestTemplateBuilderConfiguration {

        @Bean
        RestTemplateBuilder restTemplateBuilder() {
            return new RestTemplateBuilder().setConnectTimeout(Duration.ofSeconds(1))
                .setReadTimeout(Duration.ofSeconds(1));
        }

    }

}

```

1. Execução de testes:
    - Use a ferramenta de linha de comando do Maven para executar seus testes. Execute o comando **`mvn test`** na raiz do seu projeto.
    - O Maven compilará o código-fonte e as classes de teste, carregará o contexto do Spring Boot e executará os testes de unidade.
    - Os resultados dos testes serão exibidos no console, indicando se os testes passaram ou falharam.

## Application-Test

É normal usarmos variáveis de ambientes para a definição de diversos fatores da nossa aplicação, contudo, por isso é uma boa prática a criação de de uma estrutura de application properties (ou yaml) para testes dentro de **`src/test/resources`**

Exemplo:

```yaml
spring:
  autoconfigure:
    exclude:
      - org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration
      - org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration
      - org.springframework.cloud.openfeign.FeignAutoConfiguration
      - org.springframework.cloud.openfeign.hateoas.FeignHalAutoConfiguration
      - org.springframework.cloud.openfeign.loadbalancer.FeignLoadBalancerAutoConfiguration
      - org.springframework.cloud.openfeign.encoding.FeignAcceptGzipEncodingAutoConfiguration
      - org.springframework.cloud.openfeign.encoding.FeignContentGzipEncodingAutoConfiguration
      - org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration
logging:
  level:
    org:
      springframework:
        web:
          client:
            RestTemplate: debug
Organization:
  zendesk:
    country-to-instance:
      co:
        base-url: https://example.com/co
        user: co_zendesk_user
        api-token: co_zendesk_api_token
      ec:
        base-url: https://example.com/ec
        user: ec_zendesk_user
        password: ec_zendesk_password
  caffeine-cache:
    country-schedule-cache:
      refresh-after-write: PT1M
  countries:
    enabled:
      - co
      - ec
      - sa # invalid country - it will pass the entry point but will get stuck in any other country filtering point
  conversation:
    default-virtual-agent-profile:
      name: Test Agent
```

Com essa classe criada,  podemos passar no nosso SpringBootTest o profile que queremos utilizar.

```java
@Slf4j
@SpringBootTest(
    webEnvironment = WebEnvironment.RANDOM_PORT,
    properties = "spring.profiles.active=test")
```

# Testes de Integração

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9a9c67d7-98cc-495e-bfb7-5994dbeb18da/Untitled.png)

Caso possível, devemos usar o autowired para injetar as dependências. Se necessário, podemos mockar dependências com @MockBean 

```java
@MockBean ChatInfoRepository repository;
@Autowired ScheduleProperties scheduleProperties;

```

# Rest Assured ou cucumber
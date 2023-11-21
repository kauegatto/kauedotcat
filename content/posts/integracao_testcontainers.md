---
title: "TestContainers em Java: Testes de integração, repositórios e outras coisas!"
date: 2023-11-20T05:55:03+00:00
weight: 3
# aliases: ["/first"]
tags: ["TEST","JAVA","Teste de Integracao", "Teste de Unidade"]
series: ["Testes em JAVA"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Testcontainers é um framework de código aberto para fornecer instâncias descartáveis e leves de bancos de dados, message brokers, browsers ou praticamente qualquer coisa que possa ser executada em um container Docker."
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
Aviso: Esse post ainda passará por uma revisão!

# Testando Repositórios (De verdade!)

- Podemos testar repositórios de alguns jeitos, uma das maneiras é utilizar um banco em memória. Para SQL, h2 é fácil e rápido, podemos fazer as configs no banco usando um application-properties para os testes e validar tudo bonitinho normalmente 🙂. Outra alternativa seria utilizar TestContainers
- Conseguindo testar e subir repositórios e message brokers reais, conseguimos fazer testes de integração!
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
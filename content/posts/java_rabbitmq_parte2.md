---
title: "RabbitMQ com Java e Spring : Entendendo de verdade e com um toque de elegância (parte 2)"
date: 2023-11-12T19:30:03+00:00
weight: 2
# aliases: ["/first"]
tags: ["SPRING","JAVA","RABBITMQ"]
series: ["RabbitMQ com SPRING"]
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: true
description: "RabbitMQ é uma das principais soluções para envio de mensagens assíncronas na indústria, vamos entender o que é o protocolo AMQP e começar a implementar essa ferramenta com muitos exemplos em JAVA"
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
# Aprofundando

### Nota:

Nesse momento, entraremos um pouco mais em detalhes sobre como o protocolo AMQP funciona, escrevi um “guia” bem básico sobre propriedades do protocolo, se quiser conferir, pode ver aqui :)

[mensageria.pdf](https://prod-files-secure.s3.us-west-2.amazonaws.com/f9f3aa45-f67b-42fb-a9fd-ab2167038212/299d79c0-36e3-442a-a6bf-3cdf3d4e5a1a/mensageria.pdf)

Ou [nesse link](https://drive.google.com/drive/folders/1PcESYtUyQ6O9QZun67YhjYXFjJSoCLZq?usp=sharing)

## Contexto

O `Spring AMQP` consiste em dois módulos principais: `spring-amqp` e `spring-rabbit`. O ‘spring-amqp’ contém o pacote `org.springframework.amqp.core`, que trata das principais abstrações definidas no protocolo AMQP (RabbitMQ é um broker, que implementa esse protocolo), esse pacote não se baseia em nenhuma biblioteca de clientes nem implementação de broker.

Essas abstrações então são implementadas pelos módulos específicos dos brokers (`spring-rabbit`). [Teoricamente, como o AMQP é opera em nível de protocolo, você poderia utilizar o cliente do rabbit com outro broker, mas isso não é oficialmente suportado](https://docs.spring.io/spring-amqp/reference/html/#amqp-abstractions).

## A mensagem

A mensagem ****definida no protocolo amqp é um conjunto de bytes e propriedades, passados separadamente. Para tornar o uso mais fácil, dentro do java juntamos isso em uma abstração chamada `Message`

```java
public class Message {

    private final MessageProperties messageProperties;
    private final byte[] body;

    public Message(byte[] body, MessageProperties messageProperties) {
        this.body = body;
        this.messageProperties = messageProperties;
    }

    public byte[] getBody() {
        return this.body;
    }

    public MessageProperties getMessageProperties() {
        return this.messageProperties;
    }
}
```

## A exchange

A exchange é uma outra abstração simples, é basicamente o centro de distribuição de mensagens, que envia as mensagens de acordo com suas diretrizes:

```java
public interface Exchange {
    String getName();
    String getExchangeType();
    boolean isDurable();
    boolean isAutoDelete();
    Map<String, Object> getArguments();
}
```

Os tipos básicos de exchange são: `direct`, `topic`, `fanout` e `headers`. Você pode encontrar implementações para cada um dos tipos no pacote core.

> A `Topic` exchange supports bindings with routing patterns that may include the '*' and '#' wildcards for 'exactly-one' and 'zero-or-more', respectively. The `Fanout` exchange publishes to all queues that are bound to it without taking any routing key into consideration.
> 

<aside>
💡 A especificação AMQP define uma exchange padrão não nomeada, todas as queues sem exchange vinculadas são automaticamente vinculadas à ela, com seus nomes como routing keys

</aside>

## Queues

A classe `Queue` também representa uma abstração desse tipo no protocolo:

```java
public class Queue  {

    private final String name;
    private volatile boolean durable;
    private volatile boolean exclusive;
    private volatile boolean autoDelete;
    private volatile Map<String, Object> arguments;

    /**
     * The queue is durable, non-exclusive and non auto-delete.
     *
     * @param name the name of the queue.
     */
    public Queue(String name) {
        this(name, true, false, false);
    }

    // Getters e Setters omitidos
}
```

## Bindings

Bindings são a relação entre filas e exchanges!

```java
new Binding(someQueue, someDirectExchange, "foo.bar"); // direct exchange, routing keys fixas
new Binding(someQueue, someTopicExchange, "foo.*"); // topic exchange, usando wildcard
new Binding(someQueue, someFanoutExchange); // fanout
Binding b = BindingBuilder.bind(someQueue).to(someTopicExchange).with("foo.*"); 
// BindingBuilder é a maneira bonitinha, eu gosto, mas importa estático!
```

<aside>
💡 Uma instância de uma `Binding` não trará alterações reais por ela mesma, para isso, deveremos usar a classe `AmqpAdmin` ou definir as bindings usando a anotação `@Bean`, é o que veremos a seguir

</aside>

# Definindo Exchanges e Bindings customizadas

Já vimos nos exemplos anteriores como as exchanges, bindings e queues são criadas, a partir de agora, só criar!

Nossas configurações, ao invés de só possuir bean de `Queue`, agora incluirão `Exchanges`e `Bindings`

```java
@Configuration
@Slf4j
@RequiredArgsConstructor
public class RabbitMqConfiguration {
  private final TicketQueueProperties ticketQueueProperties;

  @Bean
  public Queue queue(){
    log.info("Looking for queue: {}", ticketQueueProperties.getName());
    return new Queue(ticketQueueProperties.getName(), true);
  }
  @Bean Exchange ticketDirectExchange(){
    final String EXCHANGE_NAME = "ticket";
    log.info("Creating exchange: ticket-exchange");
    return new DirectExchange(EXCHANGE_NAME);
  }
  @Bean Binding ticketBinding(){
    log.info("Create ticket binding");
    return BindingBuilder.bind(queue()).to(ticketDirectExchange()).with(ticketQueueProperties.getName()).noargs();
  }
}
```

<aside>
💡 Para fazer direito, provavelmente também faríamos o refactor da nossa `TicketQueueProperties`, provavelmente teríamos um `RabbitMqProperties`, onde deixaríamos configurações de filas, exchanges e bindings de maneira mais organizada!

</aside>

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/f9f3aa45-f67b-42fb-a9fd-ab2167038212/64aa053c-0181-4311-b4df-0b3790254061/Untitled.png)

Perfeito, já vimos nossa exchange funcionando bonito! Tudo pronto!

Só que não! Lembre-se que o nosso publisher está enviando mensagens com a routing key correta, mas para a exchange errada, vamos mudar o código para o seguinte:

```java
rabbitTemplate.convertAndSend("direct.ticket",ticketQueueProperties.getName(),event.name());
```

- Aqui deveríamos puxar esse nome da nossa ideal `TicketQueueProperties` :) para evitar esse spaghetti

# Passando Objetos - Message Converters

O **`AmqpTemplate`** também define vários métodos para enviar e receber mensagens que, no fim das contas, delegam tarefas para um **`MessageConverter`**. O **`MessageConverter`** fornece um método único para cada direção: um para converter para um **`Message`** e outro para converter a partir de um **`Message`**. 

Definição da interface **`MessageConverter`**:

```java
public interface MessageConverter {
    Message toMessage(Object object, MessageProperties messageProperties)
            throws MessageConversionException;
    Object fromMessage(Message message) throws MessageConversionException;
}
```

## SimpleMessageConverter

A implementação padrão do strategy **`MessageConverter`** é chamada de **`SimpleMessageConverter`**. Este é o conversor usado por uma instância de **`RabbitTemplate`** se você não configurar explicitamente uma alternativa. 

> Converts a String to a `[TextMessage](https://jakarta.ee/specifications/platform/9/apidocs/jakarta/jms/TextMessage.html)`, a byte array to a `[BytesMessage](https://jakarta.ee/specifications/platform/9/apidocs/jakarta/jms/BytesMessage.html)`, a Map to a `[MapMessage](https://jakarta.ee/specifications/platform/9/apidocs/jakarta/jms/MapMessage.html)`, and a Serializable object to a `[ObjectMessage](https://jakarta.ee/specifications/platform/9/apidocs/jakarta/jms/ObjectMessage.html)` (or vice versa).
> 

## Trocando o Conversor

Para trabalhar com objetos serializados e desserializados para JSON, vamos usar o `Jackson2JsonMessageConverter`.

```java
@Bean
public MessageConverter messageConverter() {
  return new Jackson2JsonMessageConverter();
}
```

Colocaremos isso tanto no consumer quanto no producer :)

---

# Declarables, Definição Dinâmica e Declarativa de Filas

Falamos anteriormente do nosso`TicketQueueProperties` , que poderíamos melhorá-lo, é o que vamos fazer, na realidade, vamos substituí-lo.

Primeiro de tudo, vamos definir um formato declarativo para filas, exchanges e bindings que nos agrade, para mim:

```yaml
broker:
    queues:
        ticket:
            name: default.ticket
    exchanges:
        ticket:
            name: direct.ticket
            type: direct
    bindings:
        ticket:
            exchange: direct.ticket
            queue: default.ticket
            routingKey: default.ticket
```

## Criando um ConfigurationProperties adequado

A partir disso, vamos mapear essas propriedades em classes de uma maneira adequada. Chamarei a classe de `BrokerConfigurationProperties`:

```java
package com.kaue.ticketservice.infrastructure.properties;

import jakarta.validation.constraints.NotEmpty;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

import lombok.Data;

@Configuration
@ConfigurationProperties(prefix = "broker")
@Data
public class BrokerConfigurationProperties {
  private Map<String, QueueProperties> queues;
  private Map<String, ExchangeProperties> exchanges;
  private Map<String, BindingProperties> bindings;

  @Data
  public static class QueueProperties {
    @NotEmpty
    private String name;
  }

  @Data
  public static class ExchangeProperties {
    @NotEmpty
    private String name;
    private String type;
  }

  @Data
  public static class BindingProperties {
    @NotEmpty
    private String exchange;
    @NotEmpty
    private String queue;
    @NotEmpty
    private String routingKey;
  }
}
```

- Possuímos 3 maps, estruturas que linkam uma chave à sua correspondente configuração, `Queue`, `Exchange` ou `Binding` Properties.
- Fazemos o mapeamento padrão, usando : `@ConfigurationProperties(prefix = "broker")`, até aqui, sem segredos 🙂

## Transformando as propriedades em objetos!

A partir de agora, o terceiro passo pode parecer simples, devemos criar beans a partir das propriedades, isso não é um problema, pelo menos não se quisermos definir os Beans da maneira que fizemos antes, apesar disso, se quisermos definir uma lista de Queues, Exchanges e Bindings, devemos usar a classe `Declarables`, e prover um bean para ela.

> [Declarables: *“…Used to declare multiple objects on the broker using a single bean declaration for the collection.”*](https://docs.spring.io/spring-amqp/api/org/springframework/amqp/core/Declarables.html)
> 

```java
@Bean
public Declarables es() {
  return new Declarables(
    new DirectExchange("e2", false, true),
    new DirectExchange("e3", false, true));
}

@Bean
public Declarables qs() {
	return new Declarables(
    new Queue("q2", false, false, true),
    new Queue("q3", false, false, true));
}

@Bean
public Declarables bs() {
  return new Declarables(
    new Binding("q2", DestinationType.QUEUE, "e2", "k2", null),
    new Binding("q3", DestinationType.QUEUE, "e3", "k3", null));
}
```

O exemplo acima, [da documentação de referência do spring](https://docs.spring.io/spring-amqp/reference/html/#collection-declaration), é uma boa forma de exemplificar o uso mais simples de Declarables, vamos ver minha implementação em particular, que adiciona declarables de acordo com a `BrokerConfigurationProperties`

```java
package com.kaue.ticketservice.infrastructure.configuration;

// ... ommitted

/**
 * This classes creates all queues, exchanges and bindings based on application.yaml when they're needed (called by a consumer or posted a message into).
 */
@Configuration
@Slf4j
@RequiredArgsConstructor
public class RabbitMqConfiguration {
  private final BrokerConfigurationProperties brokerConfig;
  private final List<Queue> definedQueues = new ArrayList<>();
  private final List<Exchange> definedExchanges = new ArrayList<>();

  @Bean
  public Declarables queues() {
    if (brokerConfig == null || brokerConfig.getQueues() == null) {
      return new Declarables(); // Return an empty list if no queues are configured
    }

    var queueList = brokerConfig.getQueues().values().stream()
      .filter(Objects::nonNull)
      .map(queueProperties -> new Queue(queueProperties.getName(), true))
      .toList();

    definedQueues.addAll(queueList);
    log.info("Declared queues");
    return new Declarables(queueList);
  }
  @Bean
  public Declarables exchanges() {
    if (brokerConfig == null || brokerConfig.getExchanges() == null) {
      return new Declarables(); // Return an empty list if no exchanges are configured
    }

    var exchangesList = brokerConfig.getExchanges().values().stream()
      .filter(Objects::nonNull)
      .map(exchangeProperties -> new DirectExchange(exchangeProperties.getName())) // todo use correct exchange type
      .toList();

    definedExchanges.addAll(exchangesList);
    log.info("Declared exchanges");
    return new Declarables(exchangesList);
  }
  @Bean
  public Declarables bindings() {
    if (brokerConfig == null || brokerConfig.getBindings() == null) {
      return new Declarables();
    }

    var bindingsList = brokerConfig.getBindings().values().stream()
        .map(bindingProperties -> {
          log.info("Creating binding between exchange {} and queue {} with routing key {}",
                  bindingProperties.getExchange(), bindingProperties.getQueue(), bindingProperties.getRoutingKey());
          Queue queue = findQueueByName(bindingProperties.getQueue());
          Exchange exchange = findExchangeByName(bindingProperties.getExchange());

          return BindingBuilder.bind(queue)
                  .to(exchange)
                  .with(bindingProperties.getRoutingKey())
                  .noargs();
        })
        .toList();
    return new Declarables(bindingsList);
  }

  private Queue findQueueByName (String queueName){
      return definedQueues.stream()
              .filter(queue -> queueName.equals(queue.getName()))
              .findFirst()
              .orElse(null);
    }

    private Exchange findExchangeByName (String exchangeName){
      return definedExchanges.stream()
              .filter(exchange -> exchangeName.equals(exchange.getName()))
              .findFirst()
              .orElse(null);
    }
  }
```

Embora grande, a implementação é relativamente simples, usamos streams para transformar as propriedades em classes reais e retornamos o Declarable como um `Bean`, um objeto gerenciado pelo spring.

# Poderes de RabbitListener

No Spring, quando um método anotado como listener joga uma exception, as mensagens podem ser inseridas novamente na fila e reprocessadas, descartadas ou colocadas em uma Dead Letter Queue. Nada é devolvido ao emissor da mensagem.

## Error Handling

Na versão 2.0 do Spring AMQP em diante, @RabbitLisetener tem 2 atributos: `errorHandler` e`returnExceptions`, mas eles não são configurados por padrão.

Você pode usar o `errorHandler` para prover um Bean de `RabbitListenerErrorHandler`. Essa interface funcional tem um método:

```java
@FunctionalInterface
public interface RabbitListenerErrorHandler {
    Object handleError(Message amqpMessage, org.springframework.messaging.Message<?> message,
              ListenerExecutionFailedException exception) throws Exception;
}
```

Aqui, por exemplo, poderíamos dizer que exceções de serviço ou fatais jogam exceções **`AmqpRejectAndDontRequeueException`,** para evitar requeue.

> As you can see, you have access to the raw message received from the container, the spring-messaging `Message<?>` object produced by the message converter, and the exception that was thrown by the listener (wrapped in a `ListenerExecutionFailedException`). The error handler can either return some result (which is sent as the reply) or throw the original or a new exception (which is thrown to the container or returned to the sender, depending on the `returnExceptions` setting).
> 

A citação acima comenta uma maneira de enviar exceptions de volta ao sender,  se te interessar, pode [ver aqui](https://docs.spring.io/spring-amqp/docs/current/reference/html/#annotation-error-handling)

## Retries!

Podemos customizar e modificar configurações de retry indicadas dentro do nosso projeto, para isso usaremos o projeto `spring-retry`, vamos ver uma configuração simples no `Bean` do `RabbitTemplate`:

```java
@Bean
public RabbitTemplate rabbitTemplate() {
    RabbitTemplate template = new RabbitTemplate(connectionFactory());

		RetryTemplate retryTemplate = RetryTemplate.builder()
				.maxAttempts(3)
				.fixedBackoff(1000)
				.retryOn(RemoteAccessException.class)
				.build();

		retryTemplate.execute(ctx -> {
		    // ... do something
		});

    template.setRetryTemplate(retryTemplate);
    return template;
}
```

Para mais informações, veja o `[spring-retry](https://github.com/spring-projects/spring-retry#using-retrytemplate)`

# Dead Letters

> When a listener throws an exception, it is wrapped in a `ListenerExecutionFailedException`. Normally the message is rejected and requeued by the broker. Setting `defaultRequeueRejected` to `false` causes messages to be discarded (or routed to a dead letter exchange).
> 

Vamos tentar seguir o que o comentário acima da documentação do spring diz:

```yaml
spring:
	rabbitmq :
		... adresses e outras configs
		listener:
			simple:
				default-requeue-rejected: false
```

Depois dessa configuração, as mensagens quando possuem um erro são DELETADAS, desabilitando os retries. Isso provavelmente não é o que queremos, por isso, vamos estudar as DLQ’s.

Dead Letter Queues (DLQ) são filas que possuem mensagens que tiveram sua execução falhada em algum momento, o comportamento das DLQ’s pode ser configurado no próprio broker.

Dead Letter Queues são úteis em sistemas mais críticos, onde necessitamos que um job rode de qualquer forma, onde podemos jogar mensagens de DLQ’s na exchange padrão novamente, ou pelo menos entendermos o porquê daquilo não ter sido executado, essas filas possuem diversas funções.

A maneira de definir dead letters é algo explicado dentro do protocolo AMQP, podemos apenas seguir essa configuração:

```java
@Bean
Queue messagesQueue() {
    return QueueBuilder.durable("queue-name")
      .withArgument("x-dead-letter-exchange", "nome-exchange.dlx")
      .withArgument("x-dead-letter-routing-key", "queue-name.dlq") // nao precisa ser o nome da queue, mas é comum para direct
      .build();
}
 
@Bean
Queue deadLetterQueue() {
    return QueueBuilder.durable("queue-name.dlq").build();
}
```

No fim das contas, uma dead letter queue é uma queue normal, e uma dead letter exchange também, portanto, se uma mensagem chegar na DLX (Dead Letter Exchange) e não tiver uma routing key correta, ela não chegará na fila, tudo normal por aqui.

<aside>
💡 Se tivermos uma exchange como string vazia, ela usará a exchange padrão!

</aside>

# Observações

Existem diversas maneiras de trabalhar com rabbitMQ, e uma infinidade de propriedades e configurações não mostradas aqui, como por exemplo: Feedback síncrono de exchanges e filas, Consumers Assíncronos, Containers Diferentes, propriedades de requeue, monitoramento de consumers, etc. Se algo fizer sentido para seu contexto, pode buscar no material de referência do Spring 🙂

Mais sobre DLQ: https://www.youtube.com/watch?v=GgIJWxk_-jM

Mais sobre exception handling: https://www.baeldung.com/spring-amqp-error-handling

# Referências

https://docs.spring.io/spring-amqp/reference/html/#template-retry

https://github.com/spring-projects/spring-retry

https://www.baeldung.com/spring-amqp-error-handling
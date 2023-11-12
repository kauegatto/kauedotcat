---
title: "RabbitMQ com Java e Spring : Começando (pt. 1)"
date: 2023-11-12T16:30:03+00:00
weight: 2
# aliases: ["/first"]
tags: ["SPRING","JAVA","RABBITMQ"]
showToc: true
TocOpen: false
draft: false
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
Bem vindo(a)! ao meu post de RabbitMQ com JAVA. **Esse post não tem como objetivo te ensinar RabbitMQ em detalhes ou até o protocolo AMQP. Na realidade, possuo um outro artigo onde comento sobre algumas peculiaridades do protocolo AMQP** [nesse link](https://drive.google.com/drive/folders/1PcESYtUyQ6O9QZun67YhjYXFjJSoCLZq?usp=sharing). De qualquer forma, na parte dois vou explicar por cima o que são filas, exchanges, bindings e seus tipos.

A ideia hoje é fazermos algo realmente simples e **mão na massa**: 
1. Configurarmos nosso rabbitMQ no docker com docker-compose
2. Configurarmos um publisher, que envia mensagens de texto para as filas do RabbitMQ
3. Configurarmos um consumer, que recebe essas mensagens e as trata.

# Configuração Básica

Antes de tudo, precisamos subir um servidor do Rabbit em nossa máquina local, a melhor maneira, na minha visão, de fazer isso, é usando docker, curto mais usar compose pra essas tarefas, então:

```bash
version: '3.1'

services:
  rabbitmq:
      image: rabbitmq:management
      container_name: 'rabbitmq'
      ports:
        - "5672:5672"
        - "15672:15672"
```
Depois disso, você pode rodar `docker-compose up` e ser feliz. (Você vai precisar ter docker & docker compose instalados).]
**Verifique sua instalação em `localhost:15672`**

## Definindo propriedades para a fila

Gosto de organizar propriedades em um `ConfigurationProperty` (principalmente quando elas podem crescer, o que é o caso), que busca informações do `application.properties` ou `application.yaml` 

Nesse caso em específico, vou fazer algo bem simples. 

```java
package com.kaue.ticketservice.infrastructure.properties;

import lombok.Getter;
import lombok.Setter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@ConfigurationProperties("broker.queue.ticket")
@Component
@Getter
@Setter
public class TicketQueueProperties {
  private String name;
}
```

```yaml
broker:
    queue:
        ticket:
            name: default.ticket
```

Poderíamos ter múltiplas entries, e cada queue poderia ter outras propriedades além de name (faremos isso na parte 2 😈):

```yaml
broker:
  queues:
    ticket:
      name: default.ticket
      durable: true
      autoDelete: false
      exclusive: false
    otherQueue:
      name: other.queue
      durable: false
      autoDelete: true
      exclusive: true
```

Se quiser adicionar mais filas, definiria uma classe com as configurações para cada fila e definiria o `ConfigurationProperties` mais ou menos assim:

```java
@Component
@ConfigurationProperties(prefix = "broker.queues")
public class QueueProperties {

    private Map<String, QueueConfig> queue;
...
}
```

Mas a princípio, vamos atuar só com ticket e name, do jeito que passei anteriormente.

## Definindo a conexão com o Rabbit
No seu application yaml ou properties, adicione:
```yaml
spring:
  rabbitmq:
    addresses: ${amqpURL:amqp://guest:guest@localhost}
```

Nesse caso, se a variável de ambiente amqpURL existir, ela será utilizada, caso contrário, será utilizado o padrão guest:guest, que funcionará perfeitamente com o docker compose apresentado anteriormente, então não precisa mexer se não for usar rabbit cloud ou tiver configurados as credenciais :)

## Adicionando a dependência Spring-Amqp
O RabbitMQ é uma ferramenta que implementa regras do protocolo AMQP, portanto, usaremos o Spring AMQP como dependência para configurar o nosso Rabbit!

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-amqp</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.amqp</groupId>
			<artifactId>spring-rabbit-test</artifactId>
			<scope>test</scope>
		</dependency>
```

## Criando uma Configuração Básica para Beans do Rabbit
O Spring boot trabalha com beans, que são basicamente objetos os quais ele instancia e gera, nesse caso, vamos prover configurações de beans do `rabbit` para o SPRING tomar conta, ou seja, criando um bean do tipo `Queue`, uma fila será criada automaticamente 

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
}
```
⚠️ Para evitar confusão: 

Estou usando `RequiredArgsConstructor` com um campo final: `TicketQueueProperties`, `RequiredArgsConstructor` faz com que exista um construtor que contenha todos os campos `final` nele, portanto, como é o único construtor, o Spring Boot o usará e automaticamente irá inserir a dependência `TicketQueueProperties` correta, o resultado é o mesmo que o `@Autowired`, mas a injeção via construtor é mais recomendada que o uso de Autowired ☝️🤓!
Aqui, podemos definir diversos beans, configurações de outras filas e exchanges, et cetera, um método para cada Bean;

# Definindo o primeiro Publisher
Aqui vamos usar a composição e injetar na nossa classe uma instância de `RabbitTemplate`, depois, usar o método publish. Nesse caso, vamos utilizar a exchange padrão, e o nome da fila será o primeiro parâmetro, sendo o segundo a mensagem em si.
```java
// ommitted 
import lombok.AllArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.core.RabbitTemplate;

@Slf4j
@RequiredArgsConstructor
public class RabbitTicketPublisher implements MessagePublisher {
  private final TicketQueueProperties ticketQueueProperties;
  private final RabbitTemplate rabbitTemplate;

  @Override
  public void publish(Text text) {
    log.info("Notifying queue: {} of text{}", ticketQueueProperties.getName(), text);
    rabbitTemplate.convertAndSend(ticketQueueProperties.getName(),text);
  }
}
```

`MessagePublisher`é uma interface própria que defini em meu domínio, para desacoplar a camada de infraestrutura, deixei apenas um método publish, onde enviamos os eventos e/ ou mensagens para algum lugar, as implentações sabem que lugar é esse.

## Definindo a injeção de dependências.

De maneira similar ao que já vi em C#, optei por cuidar da DI mais manualmente:

```java
// ommitted

@Configuration
@AllArgsConstructor
public class DIConfiguration {
  private TicketRepository ticketRepository;
  private TicketQueueProperties ticketQueueProperties;
  private RabbitTemplate rabbitTemplate;
  @Bean
  public TicketService ticketService() {
    return new TicketService(ticketRepository, ticketsMessagePublisher());
  }
  @Bean
  public MessagePublisher ticketsMessagePublisher(){
    return new RabbitTicketPublisher(ticketQueueProperties, rabbitTemplate);
  }
}
```

Poderíamos também criar uma interface para cada publisher, mas não sei o quanto gostaria dessa abordagem, talvez haja algo melhor, mas para mim, cuidar da desambiguação de Beans dessa forma não está sendo um problema (por hora)

# O Primeiro Consumer:
Aqui, vamos definir que estamos ouvindo a fila de nome X pela annotation `@RabbitListener`.
```java
@Slf4j
@Component
public class TicketConsumer {
  @RabbitListener(queues = "${broker.queue.ticket.name}")
  public void listenEmailQueue(@Payload String text){
    log.info("Received: {}", text);
  }
}
```

Aqui estou usando o @Value ao invés do configuration properties para exemplificar, sei que diversas pessoas preferem essa abordagem!

# Resultado

![Logs on console, rabbit running](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/leceodvqowfaxhpljisn.png)

![Rabbit gui showing queue consumption and message handling](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/k2rmjsg8dkxbznwa6u5n.png)

# Parte 2 : O que veremos
- O que são filas, exchanges e bindings 
- Definição automática elegante de filas, exchanges e bindings via application yaml usando declarables
- Enviando objetos!
- Outros super poderes do protocolo (introdução) : Retries, DLQ, DLXZ

# Referências

https://docs.spring.io/spring-amqp/reference/html/#template-retry

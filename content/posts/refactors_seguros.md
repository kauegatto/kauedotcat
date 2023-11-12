---
title: "Features e Refactors Seguros com Java e SPRING: 2 dicas simples!"
date: 2023-11-12T16:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["JAVA", "SPRING", "BACKEND"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Refactors nem sempre são confortáveis de se fazer, principalmente em pontos ou sistemas críticos, ao mexer nesses pontos, é sempre bom ter um plano B"
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
# Contexto
Quando trabalhando em sistemas reais, temos que nos preocupar com a segurança de nosso código, pontos específicos de nosso código podem ser mais suscetíveis a falhas, mudanças de uma camada de baixo nível (infraestrutura) podem acarretar em problemas caso haja grandes alterações ou até mesmo uma mudança de vendor. Nesse breve artigo irei discutir duas atividades bem frequentes na minha rotina no Bees/AmBev 
## 1. Features Toggle
Em diversos momentos, faz sentido que uma feature seja facilmente desligada ou não usando um toggle, fazendo com que essa alteração não precise de um deploy, dando mais agilidade e segurança à sua alteração.
Para isso temos algumas opções, podemos usar variáveis de ambientes ou até mesmo guardar o valor no banco de dados.
Uma maneira fácil de implementar em Spring é simplesmente puxar os dados de seu` application.properties` ou `application.yaml`

### No Spring
#### Opção 1: @Value
Essa opção é particularmente útil para configurações mais simples, que possuem apenas um campo (ex: Enabled) e não são usadas em diversos lugares.
Yaml:
```yaml
features:
    algumaFeature:
        enabled: true
    outraFeature:
        enabled: true
```
Spring:
```java
@Value("${features.algumaFeature.enabled}")
private Boolean valueFromFile;
```
#### Opção 2: @ConfigurationProperties
Melhor para cenários mais complexos, ou quando a propriedade é usada em diversos lugares.
Yaml:
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
            routingKey: default.ticke
```
Spring:
```java
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
Aqui basta você fazer o mapeamento das classes de acordo com suas propriedades. Se houver dificuldade, é uma tarefa em que alguma AI provavelmente vai lidar com facilidade.

## 2. Diferentes Beans de Infraestrutura! Permita um rollback fácil
Em um código ideal, a sua camada de domínio provavelmente estará desacoplada de sua camada de infraestrutura por meio de interfaces. Se isso não faz sentido para você, recomendo meu post de SOLID :)
Continuando, com um código com módulos de alto e baixo nível desacoplados, conseguimos mudar a infraestrutura sem mexer no domínio. Isso nos dá a liberdade de gerenciar os beans que sua classe de domínio irá usar com mais facilidade, pois todas as implementações de infraestrutura irão respeitar a interface.
```java
@AllArgsConstructor
public class TicketService {
  private final TicketRepository repository;
  private final Notifier messagePublisher;
  public List<Ticket> findAll(){
    return repository.findAll();
  }
  public Ticket findById(String Id){
    return repository.findById(Id).orElseThrow(
            () -> new TicketNotFoundException("Ticket not found")
    );
  }
  public Ticket save(Ticket ticket){
    messagePublisher.Notify(TicketEventsEnum.TICKET_CREATED);
    return repository.save(ticket);
  }
}
```
Temos aqui um Simples Service, tendo suas dependências injetadas via construtor pelo SPRING. Note que nosso serviço conhece regras de negócio, dentre elas, quando um ticket é criado, ele tem que notificar algo, e tem que guardar em algum lugar (repository pode cuidar de mongoDB, MySQL, et cetera.)
Note que se tivermos um repositório com código específico do MySQL e quisermos migrar para um código com Spring Data JPA, teremos que mudar a camada de infraestrutura (resumidamente, vamos escrever mais código de Repositório), mas nossa regra de negócio é a mesma.
Imagine agora que essa camada é um ponto crítico da sua aplicação, caso a implementação que você fez com tanta boa vontade dê errado, você terá que fazer um rollback.
### Definindo os Beans Manualmente
Ao invés de definir os repositórios com `@Component`, como teremos dois repositórios, o Spring não saberá com qual Bean lidar. 
Temos diversas maneiras de fazer a desambiguação de Beans, dentre elas, prioridades, primary, qualifier, aqui vou para uma abordagem simples, apenas para exemplificar a feature.
No meu yaml, irei definir uma propriedade:
```yaml
repository:
    type: ${DEFINED_REPOSITORY:mysql}
```
Caso exista uma variável de ambiente chamada DEFINED_REPOSITORY, ela será o padrão para o tipo do meu repositório, caso contrário, será mysql, como um fallback. (No caso, escolhi deixar o MySQL como fallback pois teoricamente ele é o repositório testado e já em produção).
Perfeito, agora vamos definir qual bean utilizar:
```java
@Configuration
public class RepositoryConfiguration{

  @Value("${repository.type}")
  private String repositoryType;
 
  @Bean
  public TicketRepository ticketRepository (){
    if("mysql".equalsIgnoreCase(repositoryType)){
       return new MySQLTicketRepository();
    }
    if("spring".equalsIgnoreCase(repositoryType)){
      return new JpaTicketRepository();
    }
    return new MySQLTicketRepository(); // fallback
  }
}
```
Outra maneira de fazer esse tipo de configuração é usar profiles do Spring.

# Obrigado!
Obrigado, espero que o breve artigo tenha sido útil. Escrevi em 20 minutos e não o revisei, então se houver algum problema, pode me avisar :)
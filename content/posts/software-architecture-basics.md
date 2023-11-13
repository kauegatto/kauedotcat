---
title: "Arquitetura de Software para devs: MVC, Hexagonal, DDD"
date: 2023-11-13T12:55:03+00:00
weight: 1
aliases: ["/arquitetura"]
tags: ["ARCHITECTURE","OOP","Microservices", "DDD"]
series: ["Arquitetura"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "O básico que você como dev precisa conhecer de arquitetura de software! Maior e mais denso post, mas também o mais rico."
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
    image: "https://dev-to-uploads.s3.amazonaws.com/uploads/articles/k9l7tsfne0qynrhvk1x7.png" # image path/url
    alt: "Camadas comuns em uma aplicação que segue Domain Driven Design" # alt text
    caption: "Camadas comuns em uma aplicação que segue Domain Driven Design" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/kauegatto/kauedotcat/content"
    Text: "Sugerir Alterações" # edit text
    appendFilePath: true # to append file path to Edit link
---
# Arquitetura à nivel de Software:
Refere-se à organização e definição de regras a serem seguidas no seu projeto em si, seja ele um microserviço, monolito ou qualquer outra parte de uma solução maior, nossa ênfase está no nível do seu serviço, um serviço seu pode seguir à risca SOLID, arquitetura hexagonal e uma PoC pode seguir o famoso: faz rápido e funcionando.

De outro lado, cuidando e decidindo se temos SOA, Microserviços, Monolitos ou qual protocolo de comunicação usamos, temos a arquitetura de soluções, o que não é o foco do artigo
## Modelo Baseado em Camadas

É bem comum dividirmos nosso software em camadas, é o que fazemos na maior parte das arquiteturas de software modernas, essa divisão tem como objetivo separar partes do código que não devem interagir muito entre si exceto por alguns pontos de contato (que podem ser outras camadas), e também garantir que exista um “meio de campo” entre certas camadas, ou seja, a interface não vai falar diretamente com o banco de dados, existe um caminho para isso. 

Um dos pontos negativos desses modelos é que eles não costumam definir a obrigação ou sugestão de interfaces para comunicação com serviços externos, normalmente services são totalmente acoplados à infraestrutura, em alguns casos, até mesmo temos DAO’s que implementam regras de persistência na camada de modelo. O problema disso é claro, nossas regras de negócio muitas vezes acabam acopladas à meras ferramentas, trocar o banco de dados exige que você mexa em um pedaço que deveria representar sua regra de negócio, o que não acontece em outros modelos como arquitetura hexagonal (a menos que você adapte seu padrão em camadas para ter abstrações significativas, o que é totalmente válido 🙂). 

### MVC

O MVC (Model-View-Controller) é um pattern arquitetural usado como um molde pra distribuição de responsabilidades em trechos de código que tratam de interfaces com o usuário (UI). Há três responsabilidades pre-estabelecidas:

- View: contém a lógica que monta a UI (telas ou equivalente) e que trata a entrada de dados, recebendo eventos do usuário (clique, digitação etc.). As do usuário são repassadas para o Controller. A View pode buscar dados diretamente do Model para exibição na UL
- Controller. o "meio de campo", recebe interações do usuário da View e colabora com o Model para enviar e obter dados, que são repassados para a View.
- Model: o resto do código, que não tem a ver com UI. Realiza cálculos, regras de negócio, persistência, integrações com outros sistemas etc. Há diversas variações como MVP, MVVM, entre outros.

> Outros exemplos → **MVVM**
The main thrust of the Model/View/ViewModel architecture seems to be that on top of the data (”the Model”), there’s another layer of non-visual components (”the ViewModel”) that map the concepts of the data more closely to the concepts of the view of the data (”the View”). It’s the ViewModel that the View binds to, not the Model directly.
> 

## Arquitetura Hexagonal - Ports And Adapters

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/givfldzlj01691cl5dn8.png)

A arquitetura hexagonal é uma proposta de arquitetura de software que segue lógicas de desenvolvimento de software que pensam em acoplamento e coesão, basicamente, módulos de alto nível (que possuem regras de negócio) não devem depender de implementações de módulos de baixo nível (frameworks, bibliotecas de terceiros, et cetera.). Tudo que acessa o coração / domínio / regra de negócio da sua aplicação deve passar por portas, que são basicamente interfaces que representam o que aquela biblioteca fará para você, chamamos a implementação dessas interfaces de adaptadores. 

Ou seja, em um sistema de login, podemos disparar um evento em uma fila de mensagens RabbitMQ que será consumido por um outro serviço de notificação, pensando em um nível um pouco mais abstrato, esquecendo bibliotecas ou ferramentas, podemos criar uma interface de publicador de eventos e dizer que vamos usar ela, ou seja, nossa lógica de negócio precisa enviar uma notificação de cadastro e um payload, independente se rabbitMQ, kafka ou outra porta está sendo usada:

```java
private final Notifier messagePublisher;
public class TicketService {
  public Ticket create(Ticket ticket){
    messagePublisher.Notify(TicketEventsEnum.TICKET_CREATED);
    return repository.save(ticket);
  }
}
```

No **service**, a regra de negócio de criação de um ticket é exatamente essa, note que não estou usando uma regra de rabbitMQ necessariamente, nem mesmo o seu vocabulário (normalmente usaríamos publish) o importante para a regra de criação, é enviar uma notificação!

A dependência Notifier é na realidade uma interface própria, que representa o que necessitamos, o envio de uma notificação, é uma porta.

```java
public interface Notifier {
  void Notify(Object message);
}
```

O Adaptador, que é basicamente uma das opções de notificadores que vc tem é a implementação real, você poderia trocar os adaptadores e ainda assim não ter problemas no seu domínio, tendo em vista que todo adaptadores respeita a mesma interface (porta).

Imagine que você está indo viajar, o núcleo da aplicação é o conteúdo essencial da sua mala - os itens vitais que você não pode deixar para trás. Os adaptadores são os diversos compartimentos e bolsos especializados na mala, cada um projetado para acomodar diferentes necessidades, você tem um plugue para tomadas da europa, outros para os estados unidos e outra que suporta o padrão adotado na ásia (não sei nem se é diferente). Da mesma forma, os adaptadores na arquitetura hexagonal conectam o núcleo da aplicação a interfaces externas variadas, como bancos de dados, interfaces de usuário e serviços externos, esses adaptadores permitem que a aplicação funcione em ambientes diversos. 



## Clean Architecture

Não entrarei em detalhes pela sua complexidade e individualidades, mas saiba que tanto a clean architecture quanto a onion se baseiam no mesmo fundamento, de proteger a camada de domínio, com os mesmos princípios de abstração por interfaces, e adaptadores implementando-as

# DDD - Isso não é sobre DDD

O Deisgn orientado à Domínio (Domain Driven Design / DDD) é um conceito extenso e vai além de um sugestões sobre como dividir seu código em camadas (esse nem é o foco), comentando a maneira com que o software é escrito, a linguagem utilizada no processo de fabricação, o que são as fronteiras entre suas entidades e regras de negócio e como elas devem ser implementadas, realmente fazendo com que a preocupação de domínio seja a central na construção de software. 

Encare o DDD como uma *prescrição de metodologia e **processo*** para o desenvolvimento de sistemas complexos cujo foco é mapear atividades, tarefas, eventos e dados dentro de um  domínio de problema nos artefatos de tecnologia de um domínio de solução.

Apesar disso, **Evans em seu livro deu diversas sugestões arquiteturais, como por exemplo, os services,** muitas vezes, mal utilizados ou interpretados. Veremos agora algumas sugestões e pontos do autor.

## DDD - Sugestões Arquiteturais → e de design de código

Antes de tudo, acho importante definir o que é o domínio de uma aplicação:

> **Domínio:**
No contexto de Engenharia de Software é o “conhecimento” utilizado em uma determinada área de aplicação, um campo específico para qual o sistema foi desenvolvido, ou seja, os problemas, regras e soluções que envolvem uma parte da aplicação, apesar disso, muitas vezes nos referimos ao domínio de negócio (núcleo de regras e conhecimentos que envolvem o negócio) somente como domínio, leve isso em consideração, porém tenha em mente que uma aplicação tem diversos domínios.
> 

Quando o código relacionado **ao domínio** é distribuído por uma porção tão grande de outros códigos (espalhado), torna-se extremamente difícil distingui-los e raciocinar. Alterações superficiais na interface do usuário podem realmente alterar a lógica de negócios (alterações vazam para onde não devem). 

**Assim sendo:**

**Isole o modelo do domínio e a lógica de negócios e elimine qualquer dependência que eles possam ter na infraestrutura, na interface do usuário ou mesmo na lógica do aplicativo que não seja lógica de negócios. 
Particione um programa complexo em camadas. Desenvolva um design dentro de cada camada que seja coeso e que dependa apenas das camadas abaixo.** Concentre todo o código relacionado ao modelo do domínio em uma camada e isole-o do código da interface do usuário, do aplicativo e da infraestrutura. Os objetos de domínio, livres da responsabilidade de se exibir, de se armazenar, de gerenciar tarefas do aplicativo, e assim por diante, podem se concentrar em expressar o modelo do domínio. Isso permite que um modelo evolua para se tornar rico e limpo o suficiente para capturar o conhecimento essencial do negócio e colocá-lo para funcionar, sempre que uma regra de negócio surgir, o modelo de domínio deve ser o necessário por implementá-la, quem deve se adaptar às regras de negócio é a implementação, e nunca o contrário.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ezp1csg5vz7dgso6mqaj.png)

**Dito isso, colocar as responsabilidades certas no domínio não significa que o modelo deve ser anêmico, o modelo pode (e deve) manter regras e formas para que seu escopo seja válido. Ou seja, fazemos o possível para que uma entidade de domínio nasça e continue sempre de acordo com suas regras de negócio.**

## Modelos Anêmicos - Um problema

Conceito muito difundido no artigo **Anemic Domain Model, de Martin Fowler.** 

Quando falamos de modelos de domínio anêmicos dizemos de modelos onde as regras de negócio associadas à uma entidade é externa à própria entidade. Temos uma classe pedido mas o método para verificar se o pedido contém itens ou não está em um “service”, que acaba sendo uma classe que possui regras que poderiam existir dentro de uma própria entidade (se contiver somente o seu comportamento). 

Classes que possuem somente atributos são classes de domínio anêmicas, idealmente, uma classe deve conter comportamento e atributos.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bc8uq0g0tkliwqa8jdqx.png)

Podemos chamar classes JAVA ou C# que são totalmente desacoplada de outras bibliotecas ou framewrks de POCO (no C#) ou POJO (no JAVA). Por serem códigos puros escritos em java ou  c#, que não deviram de uma classe base e nem retornam ou utilizam de tipos especiais, ou seja, são classes simples que sabem apenas de seu domínio, **devemos sempre seguir os princípios da [ignorância da infraestrutura](https://ayende.com/blog/3137/infrastructure-ignorance) e [ignorância da persistência](https://deviq.com/principles/persistence-ignorance) para essas classes.**

> Portanto, as entidades não devem ser associadas aos modos de exibição do cliente pois, no nível da interface do usuário, alguns dados podem ainda não ter sido validados. É por esse motivo que o ViewModel existe. O ViewModel é um modelo de dados exclusivamente para necessidades de camada de apresentação. As entidades de domínio não pertencem diretamente ao ViewModel. Em vez disso, você precisa converter entre entidades de domínio e ViewModels e vice-versa. - ****Projetar um microsserviço orientado a DDD, Microsoft****
> 

## Refatorando um Domínio anêmico

### Atributos distantes do comportamento

Para começar, recomendo ler o caso 1 de [3. Refatoração → Casos Usuais](https://www.notion.so/3-Refatora-o-Casos-Usuais-03076ae1673a41a39e0c8182df7b230c?pvs=21), depois volte aqui.  De maneira geral, classes devem guardar dentro de si atributos e comportamentos, se você possui comportamentos que agem sobre os atributos de uma classe específica, costuma fazer sentido encapsulá-los dentro da classe.

 Exemplo:

```java
Class ComprarIngressoService{
	void comprar(Pessoa pessoa, Evento evento){
		...
		if(pessoa.idade<18){
			// menor de idade
		}
		if(pessoa.getCadastro()=="ativo"){
			//
		}
	}
}
```

Esse tipo de validação é aparentemente inofensiva, contudo, frágil,  pode causar diversas repetições no código e aumentar pontos de contato para uma possível alteração, em alguns casos, esse tipo de erro piora muito a leitura. Faz sentido que a classe Pessoa cuide de propriedades das pessoas, logo, a refatoração a seguir é possível:

```java
Class ComprarIngressoService{
	void comprar(Pessoa pessoa, Evento evento){
		...
		if(pessoa.maiorDeIdade()){
			// menor de idade
		}
		if(pessoa.estaAtiva()){
			//
		}
	}
}
```

### Construtores, Builders e falta de amor aos erros de compilação

Um grande motivo para escrevermos códigos que são fortemente tipados é a possibilidade de perceber erros em tempo de compilação, erros que impedem que façamos coisas que não fazem sentido dado o contexto do que estamos tentando fazer, a semântica de string, por exemplo, entendida como cadeia de caracteres, não permite a soma de números a ela (**Some** um à kaue).

Dito isto, grande parte das classes de domínio não validam seu estado, muitas vezes nem em sua criação. É comum ver por ai classes com construtores vazios e códigos setters públicos (pois getters e setters **teoricamente** protegem o encapsulamento) isso por si só não garante que uma classe irá ser usada como esperada, veja o exemplo a seguir.

 

```java
class Pessoa{
	Pessoa(){
		// construtor vazio, no java é opcional	
	}
	@Getter
	@Setter // simulando o lombok, mas pode imaginar que são métodos getter e setters públicos
	private Long  id;
	@Getter
	@Setter
	private String nome;
	@Getter
	@Setter
	private Int peso;
}

// em algum outro lugar:
Pessoa kaue = new Pessoa();
kaue.setId(1);
kaue.setNome("kaue");
kaue.setPeso(65);
// teoricamente kaue está tranquilo levando em conta que todos os campos foram preenchidos, mas e o seguinte?
Pessoa douglas = new Pessoa();
douglas.setPeso(70);
```

Não existe erro de compilação e nem de execução (POR ENQUANTO) aqui. 

É óbvio que os setters deveriam validar se os campos foram preenchidos de seguindo um certo padrão e que faltam métodos para lidar com o objeto pessoa como indicado no ponto anterior, o modelo está anêmico, mas esse não é o foco, criamos um objeto de uma Pessoa chamado douglas, que possui apenas seu peso definido, o que provavelmente não faz sentido quando pensamos na criação de uma pessoa em um sistema, deveríamos (dependendo do negócio) ao menos forçar o preenchimento de id e nome. 

---

```java
class Pessoa{
	Pessoa(String id, String nome){
		// único construtor recebendo os campos opcionais.
		this.id=id;
		this.nome=nome;
	}
	@Getter
	// setter não existe mais
	private Long id;
	@Getter
	// setter pode até existir, mas nesse caso não vou criar.
	private String nome;
	@Getter
	@Setter
	private Int peso;
}

// em algum outro lugar:
Pessoa kaue = new Pessoa(1,"kaue");
kaue.setPeso(65);

Pessoa douglas = new Pessoa(); // erro
douglas.setPeso(70);
```

Mas e classes builders? Também não é incomum ver builders que esquecem de implementar os campos obrigatórios, para nossa felicidade, é algo simples de ser resolvido.

```java
PessoaBuilder pessoaBuilder = new PessoaBuilder(1,"kaue"); // construtor do BUILDER tem em si os parâmetros necessários para criar a classe que constrói
// se o método para pegar o builder for um método estático, só passar em seu parâmetro
Pessoa kaue = pessoaBuilder
											 .withPeso(70)
                       .build();
```

Usando o lombok  @Builder, podemos fazer:

```java
import lombok.Builder;

@Builder(builderMethodName = "hiddenBuilder")
public class Person {
		@NotNull
    private String name;
    private String surname;

    public static PersonBuilder builder(String name) {
        return hiddenBuilder().name(name);
				// o nome desse builder interno é arbitrário
    }
}
// 
Person p = Person.builder("Kaue").surname("Surname").build();
```

Essa tática, usando o lombok ou não, tem alguns problemas e normalmente não faz sentido em classes que tem muitos parâmetros obrigatórios e até mesmo em algumas classes simples, pois depreca, mesmo que um pouco, uma das grandes vantagens que a classe builder tem, a visibilidade, imagine isso:

```java
Endereco e = Endereco.builder("Osvaldo Albherto", "Parque Bitaru", "42", "Abilio")
.complemento("Ap 1")
.maisInformacoes("Pode entregar pro vizinho")
.build();
```

Somente lendo esse código, você só consegue ter certeza do complemento e maisInformacoes, os outros campos não são tão visíveis, ainda assim, como opinião pessoal, prefiro por ter esse código, que se torna um pouco menos visível mas garante o uso correto da classe, mostrando erros de compilação na própria IDE caso os atributos obrigatórios não estejam preenchidos. 

## Modelos Ricos: como lidar com dependências excessivas

Se sua classe POJO de domínio necessitar de bibliotecas ou outras dependências (faça-as serem interfaces 🙏), instanciá-la ficará extremamente inconveniente, para isso existe o **Design Pattern: Factory**

### Design Pattern: Factory

F**actories são métodos (ou classes) que possuem como retorno a criação de um outro objeto**, em casos mais simples, podem ser métodos estáticos dentro da própria classe, em casos mais complexos, onde teremos diferentes dependências a serem injetadas nas classes de domínio atráves de  um framework ou container de injeção de dependência, como o Spring faz, podemos usar classes.

Imagine a existência de uma classe usuário, que necessita que seu próprio email seja validado, e para isso, você quer usar uma biblioteca x ou y, você, respeitando princípios básicos, criará uma interface a qual Usuário dependerá, e fará com que a injeção de dependência passe a você uma instância do validador em algum momento, isso irá se tornar **extremamente** inconveniente muito rápido, portanto, podemos fazer:

```java
public class Usuario {
    private String email;
    private EmailValidator emailValidator;

    public Usuario(String email, EmailValidator emailValidator) {
        this.email = email;
        this.emailValidator = emailValidator;
    }

    public boolean isEmailValid() {
        return emailValidator.isValid(email);
    }

    // Outros métodos da classe Usuario
}
```

```java
public class UsuarioFactory {
    private final EmailValidator emailValidator;

    // Construtor com injeção de dependência!
    public UsuarioFactory(EmailValidator emailValidator) {
        this.emailValidator = emailValidator;
    }

    // Método para criar instância de Usuario usando o validador de e-mail fornecido pelo Spring (ou pelo seu framework de DI)
    public Usuario createUsuario(String email) {
        return new Usuario(email, emailValidator);
    }
}
```

## E os Services?

Evans Descreve em seu livro três tipos de services:

**Application Service**:

- Fornece para o usuário operações que o seu software pode executar, e controla a execução dessas operações através de chamadas a métodos de objetos das outras camadas (domínio, infraestrutura, etc.). **É importante dizer que a Application Service não contém regras de negócios ou conhecimento do domínio**, sendo assim, ela apenas coordena as chamadas a métodos de outras camadas e mantém o estado que reflete o progresso de uma operação para o usuário.

> Application Layer: Defines the jobs the software is supposed to do and directs the expressive domain objects to work out problems. The tasks this layer is responsible for are meaningful to the business or necessary for interaction with the application layers of other systems. This layer is kept thin. It does not contain business rules or knowledge, but only coordinates tasks and delegates work to collaborations of domain objects in the next layer down. It does not have state reflecting the business situation, but it can have state that reflects the progress of a task for the user or the program.  - Evans DDD
> 

**Domain Services**:

- Fornece para a **Application Service** métodos que permitam a execução de operações sobre os objetos de Domínio (camada mais interna). **Embora seja comum representar grande parte dos conceitos e regras principais do negócio aqui, o ideal é que esses detalhes sejam representados diretamente nos Domain Models.** Sendo assim, o Domain Service deve chamar e controlar a execução de métodos dos objetos do Domain Model **quando não é trivial ou lógico declarar um método diretamente no modelo de domínio**

> As vezes, a situação simplesmente não se trata de uma coisa.
> 
> 
> Alguns conceitos do domínio não são naturais para serem modelados na forma de objetos.
> 
> Forçar a funcionalidade do domínio necessária para que ela seja a responsabilidade de uma Entidade ou Objeto de Valor distorce a definição de um objeto baseado em modelos ou adiciona objetos artificiais sem sentido.
> 
> Assim sendo:
> 
> Quando um processo ou transformação significativa no domínio não é uma responsabilidade natural de uma Entidade ou Objeto de Valor, adicione uma operação no modelo como uma interface autônoma declarada como Serviço. Defina um contrato de serviço, um conjunto de asserções sobre interações com o Serviço. (Veja “asserções”) Torne essas asserções participantes da Linguagem Onipresente de um Contexto Delimitado específico. Dê um nome ao Serviço, que também se torne parte da Linguagem Onipresente.
> Evans - DDD
> 

**Infrastructure Services**:

- Fornece métodos que permitem a execução de operações sobre a infraestrutura na qual o software está sendo executado. Isso significa que esses serviços tem conhecimento sobre detalhes das implementações concretas da infraestrutura tais como: acesso a bancos de dados, acesso a rede, controle de operações de IO, acesso a hardware etc. Geralmente esse service é utilizado pelos Application Services para complementar e auxiliar suas operações, por exemplo, fornecer um método que permita a criação e controle de um buffer para realizar download de arquivos.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p94x93xzwh0ggch4xryl.png)

## Contrapontos:

<aside>
💡 ***Regardless, if your microservice or Bounded Context is very simple (a CRUD service), the anemic domain model in the form of entity objects with just data properties might be good enough, and it might not be worth implementing more complex DDD patterns. In that case, it will be simply a persistence model, because you have intentionally created an entity with only data for CRUD purposes.***

**[Design a microservice domain model](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/microservice-domain-model)** - Microsoft

</aside>

> Some people say that the anemic domain model is an anti-pattern. It really depends on what you are implementing. If the microservice you are creating is simple enough (for example, a CRUD service), following the anemic domain model it is not an anti-pattern. However, if you need to tackle the complexity of a microservice’s domain that has a lot of ever-changing business rules, the anemic domain model might be an anti-pattern for that microservice or Bounded Context. In that case, designing it as a rich model with entities containing data plus behavior as well as implementing additional DDD patterns (aggregates, value objects, etc.) might have huge benefits for the long-term success of such a microservice.
 - Microsoft resource
> 

Aqui entendemos uma coisa que deve ser clara, não existe bala de prata na computação, faz sentido abstraírmos o SPRING,  Controllers, Services e outras funcionalidades ou entedemos que nossa aplicação nasce acoplada ao SPRING e morre com ele? 

Aqui, tudo cabe à você entender pontos, contrapontos e o seu contexto, no seu caso. Se sua aplicação só existe junto à infraestrutura de uma biblioteca, talvez não haja motivo para desacoplá-la, se você não vê perspectivas para deixar de usar lombok, não necessariamente precisa fazer seu modelo de domínio POJOS realmente puras use seu lombok, e seja feliz. Um projeto simples ou que necessita ser entregue muito rapidamente não usar de conceitos como Arquitetura Hexagonal, DDD, CQRS ou qualquer outro pattern não se traduz emprojeto simples ou que significa código ruim.

# Referências:

[The Software Architecture Chronicles](https://herbertograca.com/2017/07/03/the-software-architecture-chronicles/)

Esse blog, essa crônica em específico - é maravilhosa!! Se estiver off, procure no wayback machine.

Domain Driven Design, Eric Evans

[Anemic Domain Model, Martin Fowler (Cosigned by Evans)](https://martinfowler.com/bliki/AnemicDomainModel.html)

Sumário de Padrões e Definições do DDD - Traduzido por Ricardo Pereira Dias


[Projetando um microsserviço orientado a DDD](https://learn.microsoft.com/pt-br/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice)

https://www.youtube.com/watch?v=1Lcr2c3MVF4

[Persistence Ignorance | DevIQ](https://deviq.com/principles/persistence-ignorance)

[Infrastructure Ignorance](https://ayende.com/blog/3137/infrastructure-ignorance)

[Hexagonal Architecture, DDD, and Spring | Baeldung](https://www.baeldung.com/hexagonal-architecture-ddd-spring)
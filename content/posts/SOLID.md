---
title: "SOLID! Um Post Aprofundado"
date: 2023-11-12T16:30:03+00:00
weight: 1
# aliases: ["/first"]
tags: ["SOLID","POO","ARCHITECTURE"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Um post aprofundado sobre SOLID, com inúmeros exemplos de código e com um aprofundamento maior que o normal."
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
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
[Leitura no Notion: SOLID](https://www.notion.so/kauegatto/2-SOLID-2803807def464f3e884a1311ead6955c?pvs=4)
******
Sei que existem 1 milhão de posts sobre SOLID, considero esse "guia" um pouco fora do comum pelo seu aprofundamento, tentei esclarecer todas as dúvidas que tinha ou poderia ter e sempre trazer exemplos, além de usar boas referências. Espero que esse post seja o seu guia definitivo de SOLID, assim como é para mim!

# O que é SOLID?

SOLID é um Acrônimo para 5 boas práticas e/ou princípios que envolvem o desenvolvimento de um bom código orientado à objetos, não quero me estender na origem, vamos para os princípios!

# Single Responsability Principle

O nome, embora auto-explicativo e que muitas vezes levava esse princípio a ser explicado como *“Uma classe deve ter uma, e apenas uma responsabilidade”*  não é necessariamente o que você deve pensar na hora de implementá-lo. Entenda esse padrão como : ***“Uma classe deve ter um, e apenas um motivo para mudar”***. Ou seja, não crie uma classe com a função de Emitir Nota Fiscal, crie uma classe NotaFiscal, e garanta que apenas coisas que envolvem o domínio de Nota Fiscal irão altera-lá, a mudança da emissão não muda o seu domínio de Nota Fiscal. Se uma alteração na impressora fizer a classe Nota Fiscal ser alterada, algo está incorreto. 

> A questão principal do SRP é o motivo para uma classe ser modificada. E esse motivo para mudança, em geral, está relacionado a um grupo de usuários ou stakeholders, que Uncle
Bob chama de atores. - Desbravando Solid

Levando em conta o que dizemos acima, também podemos dizer que o SRP pode ser definido por: *“Um módulo deve ser responsável por um , e apenas um ator”.*  O ator que se importa com a função da emissão nota fiscal é o setor de vendas, o ator que se importa com suas horas extras, o rh, o ator que se importa com salvar no banco por id é a implementação de persistência. 

Cada "interessado", de negócio ou técnico, faz com que a classe tenha uma responsabilidade diferente.

Identificar classes que mudam por diversos motivos (ou atores) é simples, a coesão pode ser uma das métricas utilizadas:

> Classes coesas têm uma característica semelhante: os conceitos que essas classes representam estariam relacionados e separá-los seria pouco natural. O SRP, no fim das contas, é uma outra maneira de falar sobre a necessidade de código coeso. - Desbravando Solid
> 

Aniche (OOP E SOLID para ninjas) sugere que, para encontrar classes pouco coesas, devemos procurar classes que: 

- Possuem muitos métodos diferentes.
- São modificadas com frequência.
- Não param nunca de crescer.

Outro fator importante é perceber a duplicação (ou pior, repetição) de código. Se seu código possui partes repetidas diversas vezes, tenha isso como um forte indicativo que essa responsabilidade provavelmente deveria estar encapsulada em algum lugar (e muito provavelmente em uma classe).

Portanto, se seu sistema pega os dados, busca coisas no banco, salva como ePUB ou PDF, tudo em uma classe só, ele provavelmente não é coeso e muito menos segue o SRP. Um exemplo disso é que toda vez que a maneira que um pdf for gerado houver de mudar, você terá que mexer nessa classe principal, e se você tiver que repetir essa alteração em diversos pontos onde o código está repetido (o que já não é um bom indicador), você provavelmente terá problemas em algum momento (e mesmo que não tenha, sua manutenção definitivamente não está facilitada).

## Caso real

Recentemente ajudei um amigo em um projeto pessoal, onde ele enviava emails para a confirmação de cadastro de usuários, todas essas responsabilidades ficavam dentro da mesma classe Usuário (Gerar token, enviar email, registrar usuário).
A partir daí, trabalhamos para termos um código mais coeso, no momento em que um usuário é registrado, ele envia um **evento** de registro de usuário (o que é só um aviso falando: cadastrei um usuário).
Com isso, uma classe chamada enviarEmailListener se prontificava a ouvir eventos de registro e alteração de senha de usuário e ela era a responsável por enviar emails.
Portanto, com o refactor, a classe de Usuário não se preocupa com o que acontece após o registro do usuário, ela apenas notifica que isso aconteceu.
Outro ponto de melhoria nisso foi a possibilidade de tornar o envio do email assíncrono, então para cadastrar um usuário, não precisavamos esperar o serviço de email fazer sua ação, ela é independente (nesse caso, não fazia sentido ser uma transação, o cadastro de um usuário não depende do email, se um erro ocorrer nessa etapa, ele pode só pedir outro email)

Note que se tivermos mais ocasiões onde devemos enviar emails de usuário (além de cadastro e alteração de emails) podemos apenas adicionar um novo evento a ser ouvido pelo listener e se precisarmos mudar a forma de enviar email, mudamos apenas em um único método (o que aumenta MUITO a capacidade de manter o código), essa é a função do SRP.

# Open-Closed Principle

> *Software entities ... should be open for extension, but closed for modification.*

> A module will be said to be open if it is still available for extension. For example, it should be possible to add fields to the data structures it contains, or new elements to the set of functions it performs.

Essas citações trazem uma noção base do que o OCP quer dizer, nosso código deve estar sempre pronto para evoluir. e de maneira natural, não devemos sentir a necessidade de modificar muitos arquivos diferentes, ou mesmo procurar (usando o CTRL+F, por exemplo) os lugares que devem ser alterados.

A segunda parte diz sobre ser fechada para modificação. nesse aspecto, podemos entender que elas não devem ter seu comportamento alterado a todo momento.

## Um exemplo

```java
public class CalculadoraDePrecos {
    public double calcula(Compra produto) {
        Frete correios = new Frete();
        double desconto = 0.0; // Inicialize desconto com um valor padrão

        if (REGRA 1) {
					// faz algo            
        }

        if (REGRA 2) {
					// faz algo            
        }

        return produto.getValor() * (1 - desconto) + frete;
    }
}
```

Note que sempre que adicionarmos regras de frete, nosso código precisará alterar a classe calculadora de preços, incluindo um novo IF, ou seja, a classe que implementar a regra de frete está acoplada à Calculadora de Preços

Uma segunda implementação comum é colocar os ifs dentro das classes
específicas. Por exemplo, a classe Frete passaria a ter as diferentes regras de negócio (os mesmos if-else). Apesar de ficar melhor, continua claro que não é a melhor abordagem.

### Melhorando

Se temos diferentes regras de desconto e de frete, basta criarmos interfaces que as representam:

```java
public interface TabelaDePreco {
	double descontoPara(double valor);
}
public class TabelaDePreco1 implements TabelaDePreco { }
public class TabelaDePreco2 implements TabelaDePreco { }
public class TabelaDePreco3 implements TabelaDePreco { }
```

```java

public interface ServicoDeEntrega {
	double para(String cidade);
}
public class Frete1 implements ServicoDeEntrega {}
public class Frete2 implements ServicoDeEntrega {}
public class Frete3 implements ServicoDeEntrega {}
```

> **"Veja que essa simples mudança altera toda a maneira de se lidar com a classe. Com ela 'aberta', ou seja, recebendo as dependências pelo construtor, podemos passar a implementação concreta que quisermos para ela. Se passarmos a implementação TabelaDePreco1, e invocarmos o método `calcula()`, o resultado será um; se passarmos a implementação TabelaDePreco2 e invocarmos o mesmo método, o resultado será outro." (ANICHE, 2015).**
> 

```java
public class CalculadoraDePrecos {
    private TabelaDePreco tabela;
    private ServicoDeEntrega entrega;

    public CalculadoraDePrecos(TabelaDePreco tabela, ServicoDeEntrega entrega) {
        this.tabela = tabela;
        this.entrega = entrega;
    }

    public double calcula(Compra produto) {
        double desconto = tabela.descontoPara(produto.getValor());
        double frete = entrega.para(produto.getCidade());
        return produto.getValor() * (1 - desconto) + frete;
    }
}
```

<aside>
💡 Exemplo do Livro OOP e SOLID para Ninjas

</aside>

Note que aqui trabalhamos também com a inversão de dependências (DIP), o “D”, do SOLID.

> "Se o OCP declara	o objetivo de	uma	arquitetura OO, o DIP declara o seu mecanismo	fundamental."
> 

Se você ainda não leu sobre o DIP, continue lendo o post, mais tarde revisite esse ponto, vai fazer sentido 😃

## Abstrações e Capacidade de Extensão

> **"O que discutimos aqui, de certa forma, mistura-se com a discussão do capítulo anterior sobre estabilidade e inversão de dependências. As interfaces (abstrações) `TabelaDePreco` e `ServicoDeEntrega` tendem a ser estáveis. A `CalculadoraDePrecos` é uma implementação mais instável e que só depende de abstrações estáveis. Pensar em abstrações nos ajuda a resolver o problema do acoplamento e, de quebra, ainda nos ajuda a ter códigos facilmente extensíveis. Isso é programar orientado a objetos. É lidar com acoplamento, coesão, pensando em abstrações para nossos problemas. Quando se tem uma boa abstração, é fácil evoluir o sistema. Seu sistema deve evoluir por meio de novas implementações dessas abstrações, previamente pensadas, e não por meio de diversos ifs espalhados por todo o código." (ANICHE, 2015).**
> 

## Design Pattern: Command

Não vou me aprofundar em patterns nesse post, mas acho legal repassar alguns exemplos que vi no material base

```java
public class EmissorNotaFiscal	{
		private	RegrasDeTributacao	tributacao;
		private	LegislacaoFiscal	legislacao;
		private	List<AcaoPosEmissao>	acoes;
		//...
		public	NotaFiscal	gera(Fatura	fatura) {
				List<Imposto>	impostos	=	tributacao.verifica(fatura);
				List<Isencao>	isencoes	=	legislacao.analisa(fatura);
				//	método	auxiliar
				NotaFiscal	nota	=	aplica(impostos,	isencoes);
				//	modificado
				for	(AcaoPosEmissao	acao:	acoes)	{
						acao.faz(nota);
				}
				return	nota;
		}
```

Exemplo do livro Desbravando Solid (AQUILES, 2022)

Note que é muito fácil adicionar novos comportamentos depois de uma nota fiscal ser emitida, podemos simplesmente adicionar um objeto na lista de ações, isso caracteriza o padrão `Command`.

Recomendo a leitura mais aprofundada em: https://refactoring.guru/design-patterns/command

## Design Pattern: Strategy

Exemplo de: https://en.wikipedia.org/wiki/Strategy_pattern

> **"Strategy permite que o algoritmo varie independentemente dos clientes que o utilizam. Strategy é um dos padrões incluídos no influente livro "Design Patterns" de Gamma et al., que popularizou o conceito de usar padrões de design para descrever como projetar software orientado a objetos flexível e reutilizável. Adiar a decisão sobre qual algoritmo usar até o tempo de execução permite que o código chamador seja mais flexível e reutilizável."**
https://en.wikipedia.org/wiki/Strategy_pattern
> 

```java
public interface IComportamentoDeFreio {
    public void frear();
}

public class FreioComABS implements IComportamentoDeFreio {
    public void frear() {
        System.out.println("Freio com ABS aplicado");
    }
}

public class FreioSimples implements IComportamentoDeFreio {
    public void frear() {
        System.out.println("Freio simples aplicado");
    }
}

/* Cliente que pode usar os algoritmos acima de forma intercambiável */
public abstract class Carro {
    private IComportamentoDeFreio comportamentoDeFreio;

    public Carro(IComportamentoDeFreio comportamentoDeFreio) {
      this.comportamentoDeFreio = comportamentoDeFreio;
    }

    public void aplicarFreio() {
        comportamentoDeFreio.frear(); // note que usamos a interface!
    }

    public void setComportamentoDeFreio(IComportamentoDeFreio tipoDeFreio) {
        this.comportamentoDeFreio = tipoDeFreio; // podemos mudar livremente!
    }
}
```

```java
public class SUV extends Carro {
    public SUV() {
        super(new FreioComABS());
    }
}
```

```java
public class CarExample {
    public static void main(final String[] arguments) {
        Carro suvCar = new SUV();
        suvCar.frear(); // freio com abs

        suvCar.setBrakeBehavior( new FreioSimples() );
        suvCar.frear();    // freia, mas sem abs.
    }
}
```

Nesse exemplo, note que definimos uma interface comum, a qual um carro se acopla, podemos trocar a implementação dessa interface livremente, também ajuda no OCP.

# Liskov Substitution Principle

> ***Funções que utilizam ponteiros ou referências para classes base devem ser capazes de usar objetos de classes derivadas sem saber disso***
> 

Nesse caso, vamos começar com citações:

> ***"Se um gato possui raça e patas, e um cachorro possui raça, patas e tipoDoPelo, logo Cachorro extends Gato? Pode parecer engraçado, mas é (...) herança por preguiça, por comodismo, porque vai dar uma ajudinha. A relação “é um” não se encaixa aqui, e vai nos gerar problemas."
Paulo Silveira, no artigo Como não aprender orientação a objetos: Herança (SILVEIRA, 2006)***
> 

O Princípio da Substituição de Liskov (LSP) estabelece uma diretriz fundamental : Ele afirma que objetos de classes derivadas ou subclasses devem poder ser usados no lugar de objetos da classe base ou superclasse sem quebrar o comportamento esperado do programa. Em outras palavras, uma classe derivada deve estender ou especializar o comportamento da classe base sem modificar seu contrato.

> ***"A ideia intuitiva de um subtipo é aquela cujos objetos fornecem todo o comportamento de objetos de outro tipo (o supertipo) mais algo extra."
Data Abstraction and Hierarchy (LISKOV, 1988)***
> 

Imagine um exemplo em que temos uma classe base chamada **`Animal`** e classes derivadas, como **`Cachorro`** e **`Gato`**. O LSP nos orienta a garantir que qualquer código que funcione com objetos **`Animal`** também funcione corretamente com objetos **`Cachorro`** e **`Gato`**. Isso significa que as subclasses não devem introduzir comportamentos que contradigam as expectativas estabelecidas pela classe base. Ou seja, se todo animal implementa o método comer, classes derivadas desse Animal devem também incluir o método comer, que funciona como esperado. (Se seu animal não come, reveja suas abstrações, o próximo princípio, ISP, pode ser útil aqui!)

## Favoreça composição sobre Herança

Essa é uma frase amplamente falada no meio da computação, algo que você já ouviu milhares de vezes, seja em artigos, livros como o Design Patterns, do GoF ou até mesmo vídeos no youtube, mas por quê?

Em vez de herdar comportamentos de uma classe base, prefira compor objetos que fornecem os comportamentos necessários. O princípio de design normalmente começa pela definição de interfaces que representam o comportamento que o sistema deve exibir, as classes recebem objetos dessas interfaces e fazem os comportamentos a partir daí

### Benefícios

Favorecer a composição proporciona maior flexibilidade no design. É mais natural construir classes de domínio de negócios a partir de vários componentes do que tentar encontrar pontos em comum entre eles e criar uma árvore de herança. Por exemplo, um pedal de acelerador e um volante têm poucos traços em comum, mas são componentes vitais em um carro.

Além disso, a composição oferece um domínio de negócios mais estável, pois está menos sujeito às peculiaridades dos membros da família. Em outras palavras, é melhor compor o que um objeto pode fazer (***[tem-um](https://en.wikipedia.org/wiki/Has-a)***) do que estender o que ele é (***[é-um](https://en.wikipedia.org/wiki/Is-a)***).

<aside>
💡 Linguagens como Go e Rust não possuem mecanismos de herança, e focam exclusivamente na composição de tipos a fim de evitar problemas com herança.

</aside>

### Drawbacks

“Composition is good until it is not” - O uso excessivo de composição pode levar a classes superlotadas e complexas, mas de maneira geral, se você mantiver : A divisão de responsabilidades, O acoplamento controlado, Uso de Design Patterns adequados, isso provavelmente não será um problema.

# Interface Segregation Principle

> [“*Make fine grained interfaces that are client specific.”](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)
[”Clients should not be forced to depend upon interfaces that they do not use”](https://web.archive.org/web/20150906155800/http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf)

Uncle Bob*
> 

O Interface Segregation Principle (ISP) basicamente nos orienta a escrever interfaces coesas, em que seus métodos conversem entre si e sempre sejam implementados.

Vamos a um exemplo:

```java
interface Imposto {
	NotaFiscal geraNota();
	double imposto(double valorCheio);
}
```

Imagine que em algum momento da existência da sua aplicação, um imposto não gere nota fiscal, nesse caso, ele irá jogar uma exception? retornar nulo? É exatamente esse tipo de situação que o ISP quer que você evite passar, pois **ambas as opções anteriores são ruins e configuram quebra de contrato.**

Nesse caso anterior, temos uma interface não muito coesa, com dois comportamentos distintos (isso se dá pois um imposto nem sempre gera nota), nesse caso, o mais adequado seria dividi-la em duas interfaces distintas, como por exemplo:

```java
interface Tributavel{
	double imposto(double valorCheio);
}

interface GeradorNF{
	NotaFiscal geraNota();
}
```

Com isso, podemos fazer com que nossas classes sejam construídas por composição de interfaces que às sirvam perfeitamente, como:

```java
class ImpostoGeraNota implements Tributavel, GeradorNF{
	// os dois métodos aqui
}
class Imposto implements Tributavel{
	// 	implementa imposto(double valorCheio)
}
```

> [Many client specific interfaces are better than one general purpose interface](https://web.archive.org/web/20150906155800/http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf)

*Uncle Bob*
> 

Além disso, muitas vezes queremos apenas parte dos atributos ou comportamentos de uma classe, e

> *Se você tiver uma classe que tenha vários clientes, em vez de carregar a classe com todos os métodos de que os clientes precisam, crie interfaces específicas para cada cliente e implemente-as na classe.*
> 
> 
> [Design Principles and Design Patterns](http://www.cvc.uab.es/shared/teach/a21291/temes/object_oriented_design/materials_adicionals/principles_and_patterns.pdf) (MARTIN, 2000) 
> 
> Citação retirada da Referência da Caelum
> 

## Por que?

Mas Kauê, por quê é uma boa prática criarmos interfaces magras? Como disse anteriormente, a palavra chave é **coesão**, a coesão é o elemento chave que garante a estabilidade de nossas interfaces, e se essa interface realmente precisar ser mudada, apenas os membros que implementam ela terão sua implementação alterada. No final das contas, o ISP é sobre avaliarmos a coesão das nossas interfaces,

Lembre-se também que devemos sempre nos acoplar a membros os mais estáveis possíveis! Nesse caso, tanto `Tributavel` quanto `GeradorNF` são mais interfaces coesas e estáveis.

> “Classes que dependem de interfaces leves sofrem menos com mudanças em outros pontos do sistema. Novamente, elas são pequenas, portanto, têm poucas razões para mudar.” - Aniche, M. OOP E SOLID para Ninjas.
> 

# Dependency Inversion Principle

> Depend upon Abstractions. Do not depend upon concretions. - Design Principles and Design Patterns, Bob Martin.
> 

Aqui, vamos começar com conceitos bem fundamentados dentro da computação

> “Depender de abstrações e não de implementações” - Bob Martin
”Programe voltado à interface, não à implementação” - Design Patterns, GoF
> 

Mas o que isso significa? A ideia aqui é que abstrações e interfaces definem contratos estáveis para nossos sistemas, classes e comportamentos base do sistema, ou seja, não-voláteis, não devem ser impactados por implementação de módulos as quais elas dependem.

Sua classe que cuida do registro do usuário não deve depender da implementação que o envio de email possui, você não deve ter imports de uma biblioteca específica dentro de uma classe que cuida de regras de negócio. Nesse caso, você deverá usar abstração, por exemplo, `EnviadorEmail` e usará algum método dela que encapsule detalhes de implementação

 
Vamos ver um exemplo bom:

```java
public interface EnviadorEmail {
    void enviarEmail(String destinatario, String mensagem);
}

// em seu próprio arquivo:
public class RegistrarUsuario {
    private final EnviadorEmail enviadorEmail;

    public RegistrarUsuario (EnviadorEmail enviadorEmail) {
        this.enviadorEmail = enviadorEmail;
    }

    public void registrarUsuario(String nome, String email) {
        // Lógica para registrar o usuário

        // Envio de e-mail de boas-vindas
        String mensagem = "Bem-vindo, " + nome + "!";
        enviadorEmail.enviarEmail(email, mensagem);
    }
}
```

No código acima, podemos mudar a implementação do provedor de email e não precisaremos mexer na classe RegistrarUsuário, perfeito!

<aside>
💡 Nesse caso em específico, gosto da abordagem de eventos, onde o enviador de email que diz estar ouvindo eventos de cadastro, mudança de senha e/ou emissão de nota fiscal. Mas é apenas uma ressalva :)

</aside>

## Um pouco mais concreto:

Vamos dar algumas definições que podem ajudar a você seguir como guideline para garantir que seu código está desacoplado

> Módulos de Alto Nível: São módulos que implementam regras de negócio, devem ser reutilizáveis, estáveis e inafetados por mudanças em módulos de baixo nível.

Módulos de Baixo Nível: São módulos que dizem respeito à detalhes de implementação, muitas vezes, agem como utilitários para módulos de alto nível, mas devem ser utilizados através de uma barreira de abstração sempre que fizer sentido
> 

<aside>
💡 Nem toda abstração é necessária, nem todo acoplamento é ruim. Não faz sentido abstrairmos classes estáveis da linguagem que estamos utilizando, como por exemplo, a classe `String` em Java.

</aside>

Portanto, nossa “”regra”” é:  

******************************************************************

Módulos de Alto nível não devem depender explicitamente de módulos de baixo nível, e sim de abstrações.

****************************************************************** 

### E de onde vem o nome de “Inversão de Dependência”?



![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ocx5a6bauxldjpj1amvq.png)

No exemplo acima é claro, `RegistrarUsuário`, uma classe de alto nível, depende do Enviador de Email, que utiliza a dependência do AWS SES, no final das contas, `RegistrarUsuário` também está acoplado ao SES.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j2oeb1i5vbr9crtjjvkd.png)

Nesse caso, note que as dependências foram invertidas.

## Nem toda interface é de alto nível

A linguagem Java está cheia de interfaces e abstrações, mas muitas vezes essas abstrações atuam diretamente sobre detalhes de implementação, como `java.sql.Connection` do JDBC, elas são interfaces para mecanismos de entrega, detalhes técnicos e uma classe que depende delas, não necessariamente segue o dip, mas isso não é um problema, pois essas interfaces são **estáveis**

## Dicas gerais

1. Não precisamos implementar e criar uma dependência diretamente com a classe EnviadorEmailX ou EnviadorEmailY, nesse caso, podemos usar o design pattern `Factory` e abstrair essa dependência, podemos fazer isso dentro da própria interface com java.

```java
public interface EnviadorEmail {
    void enviarEmail(String destinatario, String mensagem);

    static EnviadorEmail criarEnviadorEmailPadrao() {
        return new EnviadorEmailPadrao();
    }

    static EnviadorEmail criarEnviadorEmailAlternativo() {
        return new EnviadorEmailAlternativo();
    }
}
// Em outra classe
public class Main {
    public static void main(String[] args) {
        EnviadorEmail enviadorPadrao = EnviadorEmail.criarEnviadorEmailPadrao();
        EnviadorEmail enviadorAlternativo = EnviadorEmail.criarEnviadorEmailAlternativo();

        // Use as instâncias de EnviadorEmail conforme necessário
        enviadorPadrao.enviarEmail("destinatario@example.com", "Mensagem de teste");
        enviadorAlternativo.enviarEmail("outrodestinatario@example.com", "Outra mensagem de teste");
    }
}
```

1. Em java, se definirmos interfaces comuns para diversos provedores de notificação como `enviadorMensagem`, que ser para tanto enviadorEmail quanto enviadorSMS, podemos fazer com que o spring nos entregue uma lista com todos os Beans anotados com `@Component` que satisfaça essa interface (Podemos usar o applicationContext para isso), simplificando o código iterando pela lista e usando o seu método de enviar. 

```java
@Component
public class EnviadorEmail implements EnviadorMensagem {
    @Override
    public void enviarMensagem(String destinatario, String mensagem) {
			// abstraido :p
    }
}

@Component
public class EnviadorSMS implements EnviadorMensagem {
    // você já sabe
}
```

# Importante!!

Obrigado pela leitura, espero que tenha sido útil, de verdade, tentei trazer exemplos de código e bastante citações, mas a maioria desse tipo de conteúdo é fonte dos livros da casa do código e caelum, mas de maneira mais enxuta e com minhas palavras, recomendo fortemente a compra dos dois livros abaixo.

Se encontrar erros ortográficos ou algo estranho, me envie uma mensagem ou comente aqui, arrumarei assim que possível. Não farei uma rigorosa verificação antes de postar aqui.


# Referências

https://www.amazon.com.br/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8

[https://en.wikipedia.org/wiki/Composition_over_inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance#:~:text=To%20favor%20composition%20over%20inheritance,and%20creating%20a%20family%20tree)

https://www.casadocodigo.com.br/products/livro-oo-solid

https://www.casadocodigo.com.br/products/livro-desbravando-solid

https://github.com/caelum/apostila-oo-avancado-em-java/blob/master/09-interface-segregation-principle.md

https://en.wikipedia.org/wiki/Strategy_pattern

https://refactoring.guru/design-patterns/command
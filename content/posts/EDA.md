---
title: "Arquiteturas Orientadas à Eventos, Microserviços e Monolitos Modulares"
date: 2025-04-25T17:00:03+00:00
weight: 1
# aliases: ["/first"]
tags: ["EDA","ARCHITECTURE"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Discussões gerais sobre arquitetura de software e meios de comunicação assíncronos."
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
# Arquitetura Orientada à Eventos, Microserviços e Monolitos

Se você é desenvolvedor, é provável que já tenha ouvido falar sobre alguns conceitos comuns: **SOLID**, **Acoplamento**, **Coesão**, etc. Ao trabalhar com sistemas, conhecemos vantagens e desafios de diferentes tipos de arquiteturas à nível de software e solução, e entendemos como esses conceitos impactam a experiência, tempo e qualidade de um software.


Desde a última década, **Microserviços** se tornaram um desses conceitos fundamentais, e suas vantagens e desvantagens começaram a ser mais palpáveis conforme a adoção desse tipo de arquitetura em sistemas reais. Sistemas distribuídos orientados à serviços são frequentemente vendidos como a solução ideal para todos os sistemas, enquanto isso, outros autores argumentam exatamente o contrário.

Sam Newmann, em seu livro "Construindo Microserviços":
> "Infelizmente as pessoas passaram a ver os sistemas monolíticos como algo a ser evitado, isto é, algo que é inerentemente problemático. Uma arquitetura monolítica é uma opção, e uma opção válida. Eu poderia ir além e dizer que, em minha opinião, é a opção padrão sensata como estilo de arquitetura.
>Em outras palavras, estou procurando um motivo para ser convencido a utilizar microserviços, em vez de procurar um motivo para não usar"

No mesmo livro, outras arquiteturas são apresentadas, dentre elas o "Sistema Monolítico Modular":

> "Para muitas empresas, o sistema monolítico modular pode ser uma excelente opção. Se as fronteiras dos módulos forem bem definidas, é possível ter um grau elevado de paralelismo nos trabalhos, ao mesmo tempo que os problemas da arquitetura de microserviços, mais distribuída, são evitados, pois há uma topologia muito mais simples para implantação. A Shopify é um ótimo exemplo de uma empresa que empregou essa técnica como uma alternativa à decomposição em microserviços e parece fnucionar muito bem para essa empresa"

E então:
> "Um sistema distribuído é um sistema no qual a falha de um computador que você nem sabia que existia pode deixar seu prório computador inutilizável" - Leslie Lampert

Um ponto que favorece bastante monolitos modularizados é a fácil capacidade de transformar módulos em serviços independentes (Microserviços), conforme a necessidade passe a existir. Aproveitando o baixo acoplamento entre os módulos, essa alteração se torna mais simples pois evita mexer em outras partes do sistema (No máximo nos _adaptadores_, responsáveis por se comunicar com esse novo serviço), visto que módulos agem como uma camada de isolamento.

De maneira geral, tendo a pensar que para organizações menores, prova de conceitos ou outros casos de uso, microserviços podem não ser o ideal para você, apesar disso, o foco do artigo é menos nesses dois tipos de arquitetura, e sim como ambos podem se beneficiar (ou não) de um sistema de comunicação assí baseado em eventos.

Independente se você está produzindo seu software em uma arquitetura realmente distribuída ou só modularizada, um ponto importante é garantir o baixo acomplamento e alta coesão desses módulos ou serviços.

# Protegendo seu código de acoplamento ruim

Acoplamento sempre existirá, livros como _**"OOP e Solid para Ninjas"**_ - de Mauricio Aniche e _**"Desbravando SOLID"**_ - de Alexandre Aquiles enfatizam isso, mas principalmente, a diferença entre um acoplamento bom e ruim.

De maneira geral, acoplamento está intrinsecamente relacionado à coesão. Módulos com pouco acoplamento ruim são coesos, se elementos de código mudam em conjunto, eles devem se manter em conjunto, assim, quando algum desses elementos mudar, isso não exige alteração em múltiplas partes de seu sistema de uma vez, o que seria um forte indicativo que a coesão do seu sistema não é das melhores.

Um exemplo: Regras de negócio de compras devem se manter no módulo e **contexto** de compras, se esse módulo começar a fazer suposições e uso de partes que não pertencem a esse módulo, isso com certeza não é um acoplamento tão bom. Agora suponha que você está fazendo um código que **com certeza** não tem perspectivas de mudar de ORM, criar camadas de abstração talvez seja um trabalho desnecessário, que pode inclusive poluir sua base de código.

Ao desenhar e modelar um Sistema Modular, queremos sempre que os módulos possuam **baixo acoplamento** entre si. Como isso fica no código? Uma forma interessante é, sempre que possível, expor **contratos** entre diferentes módulos, evitando uma comunicação direta. Outra maneira de evitar acoplamento é evitar que um módulo acesse recursos como banco de dados de outro sob responsabilidade de outro módulo - isso é frequentemente chamado de _database-per-service_- isso provavelmente traria para o módulo consumidor preocupações sobre tratamento de dados que deveriam estar no módulo que é o responsável pelo banco.

## Modelos de Comunicação Síncronos e Assíncronos

Aqui, entraremos em um debate sobre **padrões de comunicação** síncronos e assíncronos.
Recomendo a leitura de dois excelentes posts do Matheus Fidelis que se aprofundam bem mais no tema:
2. https://fidelissauro.dev/padroes-de-comunicacao-sincronos/
3. https://fidelissauro.dev/mensageria-eventos-streaming/

Se tratando de Sistemas Modulares (sejam eles Microserviços ou não), um fator principal a ser levado em consideração é o acoplamento entre sistemas. A maneira com que eles se comunicam é um alto contribuínte nessa "métrica":

De maneira geral, podemos dividir a comunicação em duas grandes categorias:

1. Padrão síncrono - Request/response
A chamada é feita por um cliente à um servidor, que eventualmente response a chamada pela mesma conexão, que se mantém aberta até a resposta acontecer.
Note que aqui, o protocolo de comunicação é síncrono, mesmo que você crie alguma thread para processar esse tipo de mensagem de forma assíncrona, de maneira não bloqueante, **isso não torna seu protocolo assíncrono, apenas seu processamento.**

2. Padrão de Comunicação Assíncrono
A conexão entre quem pede a mensagem e o servidor não fica aberta esperando pela resposta em um tempo específico. Em algum momento, o servidor notifica o processamento da informação, normalmente o servidor faz a chamada proativa de chamar seus clientes via webhook ou simplesmente publica mensagens ou eventos sem se preocupar com quem e quando essa notificação será processada.
	1. Baseado em Mensagens - Aqui, sabemos qual nosso destinatário, normalmente enviamos **comandos** para outros serviços, como por exemplo: Atualize o Pedido x. (Figura 1). Existe uma variação assíncrona do modelo Request/Response comummente chamada de request-reply, onde a comunicação entre cliente/servidor se dá por meio de filas.
	2. Baseado em Eventos - Aqui, sua aplicação notifica eventos, coisas que aconteceram sob seu domínio. Se um pedido é pago, o serviço de pagamentos é responsável por enviar um evento de "Pagamento Concluído", para que **quem necessitar** dessa informação, use-a. Aqui, a aplicação que propaga eventos não sabe quem irá consumi-los. (Figura 2)
		1. Costumamos chamar esses eventos de *"Eventos de Integração"*, pois se comunica com um *"Bounded Context"* diferente.
		2. Eventos normalmente são publicados em um sistema apartado, conhecido como Event Bus. Kafka é capaz de realizar a tarefa que um event bus realiza, RabbitMQ é um event bus, NATS.io também.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tleok3v9g8d5j30u5r8g.png)
*Figura 1. Extraído de [.NET Microservices: Architecture for Containerized .NET Applications](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/)


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/go6ujat3mpsmewd35kq4.png)
Figura 2. Extraído de [.NET Microservices: Architecture for Containerized .NET Applications](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/)
## Sincronismo Vs Assíncronismo
> [❗] NOTA
> Ao usar event bus externo com um monolito, fazer chamadas de rede é algo que deve ser evitado.

### O Caminho Síncrono

> Uma abordagem popular é implementar microsserviços baseados em HTTP (REST), devido à sua simplicidade. Uma abordagem baseada em HTTP é perfeitamente aceitável; a questão aqui está relacionada a como você a utiliza. Se você usa requisições e respostas HTTP apenas para interagir com seus microsserviços a partir de aplicações cliente ou de API Gateways, tudo bem. Mas se você criar longas cadeias de chamadas HTTP síncronas entre microsserviços, comunicando-se através de suas fronteiras como se os microsserviços fossem objetos em uma aplicação monolítica, sua aplicação eventualmente terá problemas.
>  [.NET Microservices: Architecture for Containerized .NET Applications](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/) pg.35

 O problema aqui se dá principalmente pela cadeia de serviços, se você se comunica de maneira síncrona com um serviço, ele provavelmente pode se comunicar de maneira síncrona com outro, em uma cadeia que pode ser infinita. Caso um desses sistemas falhe (e eventualmente vão falhar), você terá um problema.

> Na verdade, se seus microsserviços internos estão se comunicando criando cadeias de requisições HTTP como descrito, pode-se argumentar que você tem uma aplicação monolítica, mas baseada em HTTP entre processos em vez de mecanismos de comunicação intra-processo.
 > [.NET Microservices: Architecture for Containerized .NET Applications](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/) pg.35

 Apesar disso, note que programar um serviço síncrono provê código muito mais simples de ser seguido pelo fluxo de execução natural, mais simples de ser desenvolvido, mas provavelmente menos resiliente. Caso queira forçar resiliência em padrões síncronos, com retries com backoff exponencial, circuit breakers, você acaba perdendo o que _na minha opinião_ é a maior vantagem desse tipo de comunicação, a simplicidade.

### O Caminho Assíncrono
Um código orientado à eventos possui uma boa quantidade de desafios associados - Consistência Eventual, Duplicação de mensagens, configuração de um serviço externo, **mas oferece mais resiliência de forma que processos assíncronos podem ser feitos em algum momento no futuro, podendo deixar seu sistema *parcialmente* operante em casos de falha de alguns módulos.** Note que isso aqui nos remete fortemente ao teorema CAP, onde temos que escolher entre *Consistência* e *Disponibilidade*. Não temos como manter nossa aplicação consistente se algum de nossos serviços caiu e precisamos nos comunicar com ele naquele momento.

**Em um sistema de e-commerce, você não quer que sua aplicação deixe de receber pedidos porquê o adquirente que está integrado caiu, ou porquê seu módulo de processamento está bugado, ao deixar esses eventos / pedidos guardados para reprocessamento no futuro, você evita qualquer perda financeira.**

Esse texto, como uma explicação para decisões e desafios acerca de um projeto pessoal, reflete minha decisão: Nesse projeto, fui com a arquitetura orientada à eventos principalmente por questões de aprendizado.

# Conceitos Importantes de uma comunicação assíncrona
## Consistência Eventual
Revisitando o exemplo anterior do E-commerce, onde você não quer deixar de registrar pedidos:
Supondo que você registre os pedidos, mas por algum tipo de falha, não consiga enviar esse pedido para o *Event Bus*, isso significa que outras aplicações não receberão essa notificação que o pedido foi feito, o mesmo pode acontecer para qualquer domínio. **Esse tipo de atraso na atualização de alguns dados é chamado de consistência eventual, o seu sistema, eventualmente, se tornará consistente.** É importante avisar ao usuário como a consistência eventual pode o impactar, resultando que pedidos pagos não mostrem na hora, que dados salvos em seu perfil não apareçam imediatamente, etc.

> [❗] NOTA
> Em alguns lugares, é capaz que o termo *["Consistência Posterior"*](https://pt.wikipedia.org/wiki/Consist%C3%AAncia_posterior) seja empregado, pela semântica da palavra "Eventual" no português, que está mais próximo do "Ocasionalmente". Quando falamos de consistência "*eventual*", o sistema **irá** ficar consistente em algum momento no futuro.

## Entregas de Mensagens
Sistemas que tratam envio de mensagens trabalham com diferentes tipos de garantia (que podem, ou não, ser configurados)
### At Least Once
Em sistemas que garantem entrega "At Least Once", toda mensagem será entregue pelo menos uma vez ao consumidor. Isso significa que, em casos de falha ou timeout, o sistema pode reenviar a mesma mensagem, resultando em possíveis duplicatas. Esse modelo é especialmente útil quando perder uma mensagem é inaceitável - por exemplo, em sistemas de pagamento onde perder uma transação seria catastrófico.
Em um sistema que tenta automaticamente se recuperar de falhas, a mesma mensagem poderia ser enviada múltiplas vezes. Graças à falhas de hardware e rede, o receptor deve ser capaz de implementar uma operação de processamento dessas mensagens que seja idempotente.
### At Most Once
Na garantia "At Most Once", o sistema garante que uma mensagem será entregue no máximo uma vez. Se houver falha na entrega, a mensagem será perdida ao invés de ser reenviada. Esse modelo é útil em cenários onde duplicatas são mais problemáticas que perdas - como em sistemas de métricas ou logs, onde perder algumas mensagens é aceitável, mas duplicatas poderiam distorcer análises.
### DEDUP (Deduplicação)
A deduplicação é uma estratégia fundamental, especialmente quando se usa "At Least Once". Deduplicação é exatamente o que seu nome implica, removendo mensagens duplicadas de uma lista.
Existem várias formas de implementar deduplicação, como identificadores únicos para uma mesma mensagem. Note que dedup garante Idempotência, onde diferentes requisições ou eventos com o mesmo conteúdo têm o mesmo resultado. Frequentemente sistemas garantem à você at-least-once, e ter idempotência é uma obrigatoriedade!
A maior parte dos *Event Buses* possuem jeitos de lidar com a deduplicação de mensagens, mas também podemos fazer isso no código de destino do seu Microserviço (como verificar um messageId, ou se for um evento de pedido pago, o OrderId, visto que um pedido não pode ser pago mais de uma vez), talvez ter as validações em ambas as pontas seja a melhor opção.

## Outros Problemas
Outros problemas que podem surgir ao lidar com sistemas assícronos são:
1. Ordenação de mensagens, happens-before. [Vídeo sobre o assunto](https://www.youtube.com/watch?v=OKHIdpOAxto).
2. Transações distribuidas. [Artigo sobre o assunto](https://newsletter.simpleaws.dev/p/distributed-transactions-event-driven-architectures)
# Garantindo a Publicação de Eventos e Consistência
* Ao publicar em um Event Bus, diversos problemas podem acontecer: Partições de rede, indisponibilidade do bus, queda do seu módulo. Todos esses cenários podem resultar na perda de mensagens.
* Outro problema é a alteração no estado interno de um objeto de domínio dentro do seu bounded context ser notificada de maneira errônea

## E se nossa mensagem nem for enviada?
Outro ponto importante e frequente, que decidi dar um pouco mais de prioridade é: E se nossa mensagem nem for enviada ao event bus? e se ele cair?

Existem alguns padrões e técnicas que nos permitem lidar com esse tipo de situação, uma delas é o **Outbox**, onde temos uma **tabela em algum armazenamento persistente (normalmente um banco de dados)** responsável por informar que um evento está pendente de ser enviado:
Fazemos a alteração da nossa entidade de domínio em uma transação junto com a inserção do evento como "pendente". Caso dê errado, nem nosso objeto nem o evento ficam inconsistentes

Caso essa transação seja concluída, temos essa tabela, avisando que o evento deve ocorrer, e um outro serviço (normalmente chamado de _worker_) processará ele em caso de falhas, reenviando o evento.

Note que com essa abordagem, você persiste apenas os eventos de **integração** de origem de cada microserviço. Outras abordagens (Como Event Sourcing - Que não irei abordar aqui), podem necessitar que você armazene mais eventos.

### Exemplo real
```c#
if (catalogItem == null) return NotFound();
bool raiseProductPriceChangedEvent = false;
IntegrationEvent priceChangedEvent = null;

if (catalogItem.Price != productToUpdate.Price)
	raiseProductPriceChangedEvent = true;
if (raiseProductPriceChangedEvent) // Create event if price has changed
{
	var oldPrice = catalogItem.Price;
	priceChangedEvent = new ProductPriceChangedIntegrationEvent(catalogItem.Id,
	productToUpdate.Price,
	oldPrice);
}
// Update current product
catalogItem = productToUpdate;
// Just save the updated product if the Product's Price hasn't changed.
if (!raiseProductPriceChangedEvent)
{
	await _catalogContext.SaveChangesAsync();
}
else // Publish to event bus only if product price changed
{
	// Achieving atomicity between original DB and the IntegrationEventLog
	// with a local transaction
	using (var transaction = _catalogContext.Database.BeginTransaction())
	{
		_catalogContext.CatalogItems.Update(catalogItem);
		await _catalogContext.SaveChangesAsync();
		await _integrationEventLogService.SaveEventAsync(priceChangedEvent);
		transaction.Commit();
	}
	// Publish the integration event through the event bus
	_eventBus.Publish(priceChangedEvent);
	_integrationEventLogService.MarkEventAsPublishedAsync(
	priceChangedEvent);
	}
return Ok();
}
```

Ou seja:

1. Se o Banco cair?
	1. Se essa operação for uma reação à um evento, a mensagem não será processada, para que possa ser processada em um momento futuro. Mantemos consistência e faremos o que for necessário, assim que possível.
	2. Se for fruto de uma chamada síncrona, podemos retornar um erro. De qualquer forma, o sistema não fica inconsistente.
2. Se não conseguirmos publicar no event bus?
	1. Será publicado depois, teremos um log de que o evento está pronto para a publicação, nosso domínio interno estará atualizado, mas os outros não. Como trabalhamos com consistência eventual, é um cenário completamente ok!
	2. Se o banco cair na hora de salvarmos como enviado, enviaremos o evento novamente, por isso a importância da idempotência

Esse tipo de cenário enfatiza como a alta resiliência de uma aplicação pode ser alcançada com mais facilidade usando um padrão de comunicação assíncrono por meio de eventos e mensagens 

# Conclusão

A função deste texto é dar introdução à arquiteturas síncronas, assíncronas e orientadas à eventos, mostrando vantagens, desafios e técnicas comuns. Os posts posteriores darão mais ênfase na arquitetura à nivel dos módulos - Como desenhar os bounded contexts, quanto de informação colocar nos eventos de integração, etc.

Espero que tenham gostado!

# Referências
1. Construindo Microserviços - Sam newman
2. OOP e SOLID para ninjas
3. Desbravando SOLID
4. [EVENT STORMING - DOMAIN DRIVEN DESIGN, EVENT SOURCING E CQRS! - YouTube](https://www.youtube.com/watch?v=s8cvn2TUXoM&t=136s)
5. [Message Driven Architecture to DECOUPLE a Monolith - YouTube](https://www.youtube.com/watch?v=bxGkavGaEiM&t=626s)
6. [Long live the Monolith! Monolithic Architecture != Big Ball of Mud - YouTube](https://www.youtube.com/watch?v=VGShtGU3hOc&t=9s)
7. [# .NET Microservices: Architecture for Containerized .NET Applications](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/)
8. https://newsletter.simpleaws.dev/p/distributed-transactions-event-driven-architectures
9. [Mensageria, Eventos, Streaming e Arquitetura Assincrona](https://fidelissauro.dev/mensageria-eventos-streaming/)
10. [Padrões de Comunicação Síncronos](https://fidelissauro.dev/padroes-de-comunicacao-sincronos/)
---
title: "6. Virtual Threads em Java! Fazendo a sua aplicação voar!"
date: 2024-04-06T16:43:03+00:00
# weight: 1
aliases: ["/virtual-threads", "loom"]
tags: "Concorrencia", "JAVA"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Vamos finalmente entender o Project Loom! Que adiciona as virutal threads no LTS da JDK 21"
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
Seja bem vindo, esse daqui é o quinto de 6 posts sobre concorrência em Java. A série é focada em Java, mas esse post em especial apresenta conceitos relevantes para literalmente todas as linguagens e também não é uma leitura muito extensiva :).

Nosso roteiro é:

1. Threads! Processando em Paralelo e Ganhando Throughput
2. Sincronização de Threads - DeadLocks, Zonas Críticas e Condições de Corrida
3. Concorrência, agora melhor - Classes Thread Safe
4. Executors, Thread Pools e Futures
5. CompletableFuture
**6. Virtual Threads**
   
# O Artigo
		
Primeiro, vamos estabelecer objetivos desse artigo:

> ✅ Objetivos 
>
> Dar uma breve Introdução à Programação Concorrente e Paralela
> Explicar o histórico da programação concorrente no JAVA
> Mostrar brevemente como o problema de throughput era resolvido no JAVA
> Explicar Virtual Threads
> Mostrar exemplos práticos do uso de Virtual Threads
> Mostrar concorrência estruturada

E...

> ❌ Não Objetivos
>
> Não é objetivo explicar em detalhes mecanismos de programação concorrente & paralela, para isso, recomendo fortemente [esse artigo](https://fidelissauro.dev/concorrencia-paralelismo/) do Matheus Fidelis
> Não é objetivo explicar em detalhes como a programação reativa e multithreaded é feita em JAVA  (sem ser com Virtual Threads)
> Entrar em detalhe sobre assuntos específicos do pacote `java.util.concurrent` - `Futures`, `Executors`, `Synchronizers`,  Coleções *Thread-Safe*, etc. Para isso, recomendo o[curso gratuito de Java](https://www.youtube.com/watch?v=VKjFuX91G5Q&list=PL62G310vn6nFIsOCC0H-C2infYgwm8SWW) do DevDojo

# Virtual Threads
Virtual Threads é uma feature que está disponível para uso em um LTS desde o Java 21, também chamado de project loom, é o projeto de integrar maneiras mais fáceis de escrever programas concorrentes e reativos em JAVA,  a fim de misturar performance e usabilidade

> 🔑 **[Pontos Chave](https://cr.openjdk.org/~rpressler/loom/loom/sol1_part1.html)**
>
> - A virtual thread is a `Thread` — in code, at runtime, in the debugger and in the profiler.
> - A virtual thread is not a wrapper around an OS thread, but a Java entity.
> - Creating a virtual thread is cheap — have millions, and don’t pool them!
> - Blocking a virtual thread is cheap — be synchronous!
> - No language changes are needed.
> - Pluggable schedulers offer the flexibility of asynchronous programming.

---

> ✅ **[Objetivos](https://openjdk.org/jeps/444)**
>
>* Enable server applications written in the simple thread-per-request style to scale with near-optimal hardware utilization.
>* Enable existing code that uses the java.lang.Thread API to adopt virtual threads with minimal change.
>* Enable easy troubleshooting, debugging, and profiling of virtual threads with existing JDK tools.

---

> ❌ **[Não Objetivos](https://openjdk.org/jeps/444)**
>
> * It is not a goal to remove the traditional implementation of threads, or to silently migrate existing applications to use virtual threads.
> * It is not a goal to change the basic concurrency model of Java.
> * It is not a goal to offer a new data parallelism construct in either the Java language or the Java libraries. The [Stream API](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html) remains the preferred way to process large data sets in parallel.

# Conceitos

De maneira geral, usamos as virtual threads por um motivo: Aumentar o <mark style="background: #D2B3FFA6!important">throughput</mark> (vazão) da nossa aplicação (não velocidade, que está relacionado à latência. Por enquanto, a maneira mais comum é uma: [Programação Reativa](https://en.wikipedia.org/wiki/Reactive_programming), vamos entender como a programação reativa  era no passado, e como vamos implementá-la com as virtual threads.

> 💡 **Throughput?**
>
>  Throughput diz  respeito à quantidade de elementos que você processa por uma medida de tempo (exemplo: Requests/Segundo em uma aplicação HTTP, Mensagens Processadas por Segundo em um message broker).

## Programação Assíncrona e Concorrente

A maior parte do código que escrevemos é <mark style="background: #D2B3FFA6;">síncrono</mark>, isso significa que o código vai ser executado imediatamente quando chegar naquela instrução, o código <mark style="background: #D2B3FFA6;">assíncrono</mark> é um código que vai ser executado, <mark style="background: #D2B3FFA6;">em algum momento no futuro</mark>, como uma promessa de execução.
Códigos assíncronos não significam a mesma coisa que concorrentes, um forEach assínrono, por exemplo, roda na thread principal de um programa, um código concorrente significa que ele vai ser executado em outra thread!

Podemos criar tarefas (**Tasks**) para rodarem em Threads, e para criarmos uma Thread, temos duas maneiras:
1. Criar uma Thread a partir de seu construtor, e passar à ela sua Task
2. Usar uma pool (piscina) de Threads e deixar com que o executor entregue a tarefa à uma thread disponível (se ela existir)
	1. Essa Abordagem é muito comum pois Threads são recursos limitados que não são leves de criar e destruir
	2. Conexões de Banco de dados também passam ficam em um "Pool" quando usamos frameworks como o Spring, a biblioteca que cuida da criação de um Pool de Conexões é o [Hikari](https://github.com/brettwooldridge/HikariCP)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sf7z1q17wimc33mk40oz.png)

Nesse cenário, vamos criar uma task, submetê-la ao Executor, e receberemos uma promessa de execução, por esse ponto de vista, essa tarefa é assíncrona pois será executada no futuro, mas também é concorrente, pois será executada em uma thread diferente. Aqui os conceitos se encontram, [mas não são a mesma coisa código concorrente é assíncrono, mas nem todo código assíncrono é concorrente!](https://sci-hub.se/10.1145/2846680.2846687)

## Execução Bloqueante

Uma execução bloqueante significa que uma instrução está sendo executada pela sua CPU (ou  por um core dela) e que **nenhuma outra instrução irá ocorrer enquanto a anterior ainda estiver acontecendo, mesmo que sua CPU não esteja sendo utilizada**, normalmente em uma espera de I/O ou para entrar em um bloco de código sincronizado.
Nesse cenário irá ocorrer uma troca de contexto, um processo relativamente "caro" para sua CPU que basicamente desaloca o processo até que ele exija algo novamente da CPU, por sua vez, o código não bloqueante garante que sua CPU evite trocas de contextos e esteja sempre sendo utilizada.

# O Cenário Atual - Por quê usar Virtual Threads?

Primeiro, analise o código JAVA que faz uma chamada HTTP padrão para um servidor:
```java
URI url = URI.create("https://mydata.com/data");
HttpClient client = HttpClient.newBuilder().build();
HttpRequest request = HttpRequest.newBuilder(url).GET().build();
var response = client.send(request, HttpResponse.BodyHandlers.ofString());
```


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/samjnv68urucu2x14r0p.png)


Aqui, podemos ver que nossa CPU só é realmente utilizada de maneira eficiente durante  200 nano segundos, e fica ociosa a maior parte do tempo, esperando a resposta da chamada.

Como podemos arrumar isso?
## 0. O Patinho lento - One Request Per Thread
A primeira ideia é irmos no aspecto concorrente, quando sua thread estiver esperando, o Task Scheduler vai remover ela do núcleo que está rodando (Context Switching) e coloca outra thread no lugar, executar uma request em cada thread (One-Request-Per-Thread) é a maneira convencional, que vêm sido utilizada há bons anos.

Nesse cenário, precisaríamos de 500 mil de threads - advindo da proporção entre tempo ocioso e trabalhado. 100ms/200ns - (requests) nesse núcleo para alcançarmos o uso de 100% de CPU, garantindo que sua CPU não fique ociosa. Isso definitivamente não boa bom, né?
As threads no JAVA encapsula uma thread do Sistema Operacional, também chamada de Platform Thread ou Kernel Thread, o problema é que o custo de criação de criação de uma Thread em JAVA, é o mesmo de criar uma Thread no SO, que é relativamente caro


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hlf4nnyggvqgc1pe8lak.png)
[Fonte](https://app.pluralsight.com/library/courses/java-concurrent-programming-virtual-threads/table-of-contents)

Recursos caros como threads são colocados em "Pools" para lidar com eles de maneiras mais eficientes:
> ❝ ❞ 
>
> Developers sometimes use thread pools to limit concurrent access to limited resources. For example, if a service cannot handle more than 20 concurrent requests then making all requests to the service via tasks submitted to a thread pool of size 20 will ensure that. This idiom has become ubiquitous because the high cost of platform threads has made thread pools ubiquitous
> Fonte: https://openjdk.org/jeps/444

Perfeito, mas voltando ao exemplo anterior, precisamos de 500.000 threads, quanto isso vai nos custar?
```text
Memória: 500.000Mb
Tempo de início: 500 Segundos
```
Com isso, entendemos que o modelo One Request Per Thread não é mais viável:
O artigo [Transformation patterns for a reactive application, de Bruno Miguel Mendonça Maia](https://repositorio-aberto.up.pt/bitstream/10216/98156/2/31853.pdf) pontua como característica desses sistemas :
### Pontos Negativos

**( – ) Concurrency**. Synchronous programming is not the best suited model for dealing with concurrency as the execution will start and block the current thread while waiting for the result.
**( – ) Throughput**. While a thread waits for the expensive execution to return its result, the OS can exchange active threads to promote concurrency, **but this has overhead costs and hinders throughput due to thread context switching and cache invalidation.**
**( – ) Latency**. Thread blocking on execution and the lower throughput due to the OS exchanging active threads and consequently cache invalidation leads to poorer latency.

### Pontos Positivos

**(+) Ease of use**. The synchronous sequential model and its typical imperative programming style provides a familiar thinking model that results in ease of use.
**(+) Maintainability**. Synchronous programming and its sequential execution model provides an easy to reason with concept that in turn increase maintainability. Furthermore, error handling in sequential execution is easier to tackle.

### 1. O patinho feio - Processar múltiplos requests em uma Thread
**Essa abordagem é a abordagem Reativa**. A abordagem reativa tem um princípio simples de dividir uma request em pequenas porções e nenhuma porção pode conter código bloqueantes:


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/h6e1tzwgce02w18uu2hu.png)


Aqui dividimos as etapas como foi acordado anteriormente (exceto pelo fato de que a step2 pode bloquear a CPU)
Com isso, precisamos usar um framework reativo que permita que usemos essas lambdas (aqui o exemplo é completableFuture, que faz a mesma coisa, mas usando a pool de threads, mas serve bem para explicar.):


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ilug3azfhsxzf8kbju2f.png)


Seu framework de execução terá a responsabilidade de conectar as lambdas para que o resultado delas seja passado para a próxima função corretamente, é seu trabalho não escrever código bloqueante nesse caso. **Como seu framework vai ter pouquíssimas threads (talvez só uma por núcleo), e muitas requests vão ser processadas em uma mesma thread, escrever código bloqueante vai impactar MUITO sua performance.**

Nesse caso em específico, a thread não será bloqueada pois CompletableFuture conhece o HttpRequest.send() e registra um callback, que será executado quando a função terminar de rodar.
## Pontos Negativos
* Código difícil de ler
* Código difícil de dar manutenção
* É fácil de arruinar a performance com um pedaço de código bloqueante.
* Difícil de testar

## 2. O Patinho que Existe - Futures e Callback Hell
Aqui, usamos ainda do One Request Per Thread, mas com estratégias um pouco diferentes, usamos `Futures` para escrever código paralelizável e concorrente, ganhando performance.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9u2kplp87s53uxo6t209.png)
Fonte https://blog.soaresdev.com/funcoes-callback-em-javascript/

O post [Virtual threads: Are futures a thing of the past?](https://blogs.oracle.com/javamagazine/post/virtual-threads-futures), na Java Magazine, retrata a história do código concorrente em Java. Lá, é retratado o uso de Futures para colocar suas Threads para  rodar porções bloqueantes em paralelo:

```java
void handleRequest(Socket socket) {
  var request = new Request(socket);
  var futureWeather = CompletableFuture.supplyAsync(() -> Weather.fetch(request), exec2);
  var futureRestaurants = CompletableFuture.supplyAsync(() -> Restaurants.fetch(request), exec2);
  var futureTheaters = CompletableFuture.supplyAsync(() -> Theaters.fetch(request), exec2);

  new Page(request)
      .setWeather(futureWeather.join())
      .setRestaurants(futureRestaurants.join())
      .setTheaters(futureTheaters.join())
      .send();
}
```

A ordem em que as três tasks esperam pelo resultado não importa, a thread coloca os 3 Jobs para rodar, e depois bloqueia (espera) até que elas tenham terminado. Mas pera, "e depois bloqueia (espera) até que elas tenham terminado"... Exato, ainda podemos melhorar isso bastante, bloquear Threads tem um custo:
1. [Esse blocking traz a possibilidade de deadlocks acontecerem](https://blogs.oracle.com/javamagazine/post/virtual-threads-futures). Aqui, teremos uma pool para computar recursos e para lidar com requests.
2. Bloquear e desbloquear threads traz perda de performance. Claramente sua CPU não vai ficar  2 Segundos esperando a sua resposta de I/O e vai colocar outra thread para trabalhar nesse meio termo, apesar disso, existe um custo não só para fazer a troca de contexto, mas isso também irá causar perda de dados em cache no processador, resultando em *cache misses* quando a thread estiver de volta.

Podemos "resolver" isso usando callbacks!

```java
public class Server {
  private final ServerSocket server = new ServerSocket(port);
  private final ExecutorService exec = Executors.newFixedThreadPool(16);

  public void run() {
    while (!server.isClosed()) {
      var socket = server.accept();
      exec.execute(() -> handleRequest(socket));
    }
    exec.close();
  }

  void handleRequest(Socket socket) {
    var request = new Request(socket);

    var futureWeather = CompletableFuture.supplyAsync(() -> Weather.fetch(request), exec);
    var futureRestaurants = CompletableFuture.supplyAsync(() -> Restaurants.fetch(request), exec);
    var futureTheaters = CompletableFuture.supplyAsync(() -> Theaters.fetch(request), exec);

    var page = new Page(request);

    futureWeather.thenAccept(weather ->
        futureRestaurants.thenAccept(restaurants ->
            futureTheaters.thenAccept(theaters ->
                page.setWeather(weather)
                    .setRestaurants(restaurants)
                    .setTheaters(theaters)
                    .send())));
  }
}
```

Future.thenAccept recebe como argumento um consumer, que irá consumir o resultado dessa future, a invocação de thenAccept só registra o código para uma execução futura, ele não espera o código ser completado, registrando um callback.
Nesse cenário, as threads nunca são bloqueadas e uma única pool não muito vasta pode ser  usada. O código também está livre de deadlocks.

Callbacks são difíceis de escrever e de debugar, você pode ter percebido que nesse simples evento, já temos 3 níveis de aninhamento de código, podemos melhorar isso usando outras features para lidar com futures de maneira não bloqueantes,  como usando `thenCombine`:

```java
void handleRequest(Socket socket) {
  var request = new Request(socket);

  var futureWeather = CompletableFuture.supplyAsync(() -> Weather.fetch(request), exec);
  var futureRestaurants = CompletableFuture.supplyAsync(() -> Restaurants.fetch(request), exec);
  var futureTheaters = CompletableFuture.supplyAsync(() -> Theaters.fetch(request), exec);

  CompletableFuture.completedFuture(new Page(request))
      .thenCombine(futureWeather, Page::setWeather)
      .thenCombine(futureRestaurants, Page::setRestaurants)
      .thenCombine(futureTheaters, Page::setTheaters)
      .thenAccept(Page::send);
}
```
Nesse cenário, o único processamento que a thread que lida com as conexões performa é a criação da página base, mas isso também poderia ser assíncrono:

```java
public class Server {
  private final ServerSocket server = new ServerSocket(port);
  private final ExecutorService exec = Executors.newFixedThreadPool(16);

  public void run() {
    while (!server.isClosed()) {
      var socket = server.accept();
      handleRequest(socket);
    }
    exec.close();
  }

  void handleRequest(Socket socket) {
    var futureRequest = CompletableFuture.supplyAsync(() -> new Request(socket), exec);

    var futureWeather = futureRequest.thenApplyAsync(Weather::fetch, exec);
    var futureRestaurants = futureRequest.thenApplyAsync(Restaurants::fetch, exec);
    var futureTheaters = futureRequest.thenApplyAsync(Theaters::fetch, exec);

    futureRequest
        .thenApplyAsync(Page::new, exec)
        .thenCombine(futureWeather, Page::setWeather)
        .thenCombine(futureRestaurants, Page::setRestaurants)
        .thenCombine(futureTheaters, Page::setTheaters)
        .thenAccept(Page::send);
  }
}
```

### 3. O Patinho bonito - Construir uma thread virtual, mais leve que Platform Threads.
As virtual threads são exatamente isso, conseguimos performance, simplicidade e boa capacidade de manutenção evitando códigos reativos e sem ter medo de códigos bloqueantes.

> ❝ ❞ Implicações
>
> Virtual threads are cheap and plentiful, and thus **should never be pooled**: A new virtual thread should be created for every application task. **Most virtual threads will thus be short-lived and have shallow call stacks, performing as little as a single HTTP client call or a single JDBC query.** Platform threads, by contrast, are heavyweight and expensive, and thus often must be pooled. They tend to be long-lived, have deep call stacks, and be shared among many tasks.
> **In summary, virtual threads preserve the reliable thread-per-request style that is harmonious with the design of the Java Platform while utilizing the available hardware optimally.** Using virtual threads does not require learning new concepts, though it may require unlearning habits developed to cope with today's high cost of threads.
> Virtual threads will not only help application developers — they will also help framework designers provide easy-to-use APIs that are compatible with the platform's design without compromising on scalability.

> ❗ Reforçando!
>
> Não crie  pools de threads virtuais!
# Escrevendo Virtual Threads
As maneiras que podemos criar virtual são simples:


1. Via factory
```java
Thread t3 = Thread.ofVirtual()  
        .name("Thread virtual!")  
        .start(task);  
t3.join();
```

2. Thread.startVirtualThread(task)
```java
Thread t4 = Thread.startVirtualThread(task);  
t4.join();
```

3. Usando um executorService (com o método `newVirtualThreadPerTaskExecutor`):
```java
public static void execute(){  
    var set = ConcurrentHashMap.<String>newKeySet();  
  
    Runnable task = () -> set.add(Thread.currentThread().toString());  
  
    int N_TASKS = 500;  
  
    try (var executorService = Executors.newVirtualThreadPerTaskExecutor()) {  
        for (int index = 0; index < N_TASKS; index++) {  
            executorService.submit(task);  
        }  
    }  
  
    System.out.println("# threads used = " + set.size());  
}
```

## Como Virtual Threads funcionam

**Uma virtual thread é executada em cima de uma platform thread, que chamamos de Carrier Thread.** Essas carrier threads são organizadas em uma única `ForkJoinPool`, onde cada Platform (também carrier) terá uma waitlist de virtual threads associadas à ela.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/amnt7rx42xq8p0t8ajlp.png)

Para evitar **[Starvation](<https://en.wikipedia.org/wiki/Starvation_(computer_science)>)** das threads do Sistema Operacional, se uma waitlist de uma Platform Thread zerar, ela vai "roubar" tarefas de outras threads.
Com isso, percebemos que executar um "runnable" em uma virtual thread, na realidade roda ele em uma thread real, portanto, se formos executar uma operação completamente não bloqueante, virtual threads são mais caras, é um overhead, se for o caso, rode-as diretamente na thread comum. **Virtual Threads são feitas para executar códigos bloqueantes!**

> ⚠️ Aviso
>
> **Virtual Threads não são feitas para rodar operações em memória!**

**Quando operações bloqueantes rodam em virtual threads, elas se separam de sua Carrier Thread**, usando "yield" para basicamente suspender a execução desse código, **liberando a thread principal para trabalhar com outras coisas**, então essa virtual thread é guardada na memória principal (heap) e quando estiver pronta, é colocada de novo na waitlist das threads principais, através de um callback, mas isso tudo é feito de maneira transparente!

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xr4ag965np9jk2o156nk.png)

Note que `Continuation.yield()` responsável por garantir esse processo para que a thread principal não seja bloqueada, precisa ser implementado em operações bloqueantes (isso já está feito).

Por exemplo, nessa linha de código:

```java
response.send(future1.get() + future2.get());
```

Essas operações vão fazer com que a thread virutal monte e desmonte de sua *carrier thread* diversas vezes, provavelmente uma para cada call para `get()`e possívelmente muitas outras vezes ao longo da execução de .send() graças às operações de I/O.

## Exemplos
1. Exemplo usando um executor que cria uma virtual thread para cada task bloqueante.

```java
void handle(Request request, Response response) {
    var url1 = ...
    var url2 = ...
 
    try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
        var future1 = executor.submit(() -> fetchURL(url1));
        var future2 = executor.submit(() -> fetchURL(url2));
        response.send(future1.get() + future2.get());
    } catch (ExecutionException | InterruptedException e) {
        response.fail(e);
    }
}
 
String fetchURL(URL url) throws IOException {
    try (var in = url.openStream()) {
        return new String(in.readAllBytes(), StandardCharsets.UTF_8);
    }
}
```
2. Mesmo procedimento usado no capítulo [[#2. O Patinho que Existe - Futures e Callback Hell]], mas usando virtual threads, que não bloqueiam sua CPU!

```java
void handleRequest(Socket socket) {
  var request = new Request(socket);

  var futureWeather = new CompletableFuture<Weather>();
  var futureRestaurants = new CompletableFuture<Restaurants>();
  var futureTheaters = new CompletableFuture<Theaters>();

  Thread.startVirtualThread(() -> futureWeather.complete(Weather.fetch(request)));
  Thread.startVirtualThread(() -> futureRestaurants.complete(Restaurants.fetch(request)));
  Thread.startVirtualThread(() -> futureTheaters.complete(Theaters.fetch(request)));

  new Page(request)
      .setWeather(futureWeather.join())
      .setRestaurants(futureRestaurants.join())
      .setTheaters(futureTheaters.join())
      .send();
}
```

3. Parecido com o exemplo 2, mas com um aroma mais funcional (Considerando que o executorService provê virtual threads.)

```java
void handleRequest(Socket socket) {
  var futureRequest = CompletableFuture.supplyAsync(() -> new Request(socket), exec);

  var futureWeather = futureRequest.thenApplyAsync(Weather::fetch, exec);
  var futureRestaurants = futureRequest.thenApplyAsync(Restaurants::fetch, exec);
  var futureTheaters = futureRequest.thenApplyAsync(Theaters::fetch, exec);

  futureRequest
      .thenApplyAsync(Page::new, exec)
      .thenCombine(futureWeather, Page::setWeather)
      .thenCombine(futureRestaurants, Page::setRestaurants)
      .thenCombine(futureTheaters, Page::setTheaters)
      .thenAccept(Page::send);
  }
}
```

4. Uma maneira imperativa, sem o uso de futures

```java
void handleRequest(Socket socket) {
    var request = new Request(socket);
    var page = new Page(request);
    
    Thread t1 = Thread.startVirtualThread(() -> page.setWeather(Weather.fetch(request)));
    Thread t2 = Thread.startVirtualThread(() -> page.setRestaurants(Restaurants.fetch(request)));
    Thread t3 = Thread.startVirtualThread(() -> page.setTheaters(Theaters.fetch(request)));
    
    t1.join(); t2.join(); t3.join();
    page.send();
  }
```

Um ponto negativo aqui é que a Pagina  deve ser thread safe e deve ser construída antes do uso dos fetches pois todos esses métodos podem alterar e acessar esse recurso ao mesmo tempo.
Nesse caso, algum tipo de sincronização ou lock terá de ocorrer (*pinning* nesse caso não soa tão ruim pelo contexto que a criação/set de um objeto Page é rápido)

Mais uma imagem de exemplo:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qde67241rpbr6anzh21w.png)

> ⚠️ Aviso
>
> Como movemos coisas para a memória e trazemos-as de volta, teremos problemas se estivermos usando ponteiros diretamente, mas você provavelmente não vai fazer isso em JAVA.
> Apesar disso, o bloco "synchronized", faz isso, e quando isso acontece, a task executa de maneira bloqueante na platform thread, sem ir para a heap memory como as virtual threads fazem, ou seja, o `yield` não acontece.
> Chamamos isso de **pinning**, se você precisar usar "synchronized" em um bloco que leva uma quantidade considerável de tempo (milisegundos), refatore-o para usar **ReentrantLock**, senão você vai bloquear sua pequena quantidade de platform threads por bastante tempo :(
>
> Adicionalmente, você pode observar com facilidade os *pinnings* de platform threads: New diagnostics assist in migrating code to virtual threads and in assessing whether you should replace a particular use of `synchronized` with a `java.util.concurrent` lock:
> * A JDK Flight Recorder (JFR) event is emitted when a thread blocks while pinned (see [JDK Flight Recorder](https://openjdk.org/jeps/444#JDK-Flight-Recorder-JFR)).

##  Habilitando no Spring
Para habilitar virtual threads no SPRING, use:
```yaml
spring:
	threads:
		virtual:
			enabled: true
## ou
spring.threads.virtual.enabled=true
```
Essa alteração já fará com que seu servidor deixe de trabalhar com o antigo cenário de uma thread por request, possívelmente melhorando sua performance, mesmo sem muitas alterações (No caso, isso provavelmente só ocorrerá se você já estiver recebendo uma quantidade de chamadas o suficiente para esgotar sua pool de platform threads, que é 200).
# Concorrência Estruturada  -  Feature Preview
[A feature de Concorrência Estruturada](https://openjdk.org/jeps/453), tem como foco a escrita simples de códigos
concorentes, usando o paradigma imperativo.

Em um passado distante, os códigos que eram escritos eram recheados de "go-tos", o que dificultava muito o custo de manutenção pela dificuldade de entender o fluxo de execução do  programa, estar dentro de um else não significava necessariamente, que seu *if* falhou.

O problema disso é que código concorrente, em sua forma atual é como usar um go-to, você não consegue saber quem invocou a instrução que está rodando na thread.

> ✅ [Objetivos](https://bugs.openjdk.org/browse/JDK-8306641)
>
> * Promover um estilo de programação concorrente que pode evitar riscos comuns associados  ao uso de códigos concorrentes e paralelizado
> * Melhorar a observabilidade desse tipo de código

> ❌ Não-Objetivos
>
> * Substituir maneiras de  trabalhar com código concorrente, como :  `ExecutorService` e `Future`.
> * Definir a API definitiva de Concorrência Estruturada para a plataforma java, permitindo que outras formas surjam em novas  bibliotecas ou releases da JDK
> * Definir maneiras de compartilhar *streams* de dados entre diferentes threads (exemplo: canais)
> * Substituir o mecanismo de interrupção de thread já existente, mas pode ser no futuro.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9ojnardqehu1oj9glscf.png)


> ⚡☠️ Structured Concurrency é uma Feature Preview no JAVA 21
>
> Ou seja, essa API pode sofrer alterações ao longo do tempo, e para utilizá-la, precisamos explicitamente liberar seu uso.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/inddyc6d27nraeaihpt7.png)


Com structured concurrency, criamos um escopo onde tarefas irão rodar de maneira  assíncrona com fork e join, e depois retornamos o resultado dessas operações

# Papos Técnicos para nerds

## O Fork-Join-Pool das Threads que as Virtuals usam

* Comentamos anteriormente como as Virtual Threads funcionavam em cima de threads reais: O scheduler de virtual threads é um 'work-stealing' [`ForkJoinPool`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ForkJoinPool.html) que opera usando FIFO mode. O _paralelismo_ padrão do scheduler é a quantidade padrão de [processadores  (ou quantidade de threads dos seus processadores :)](<https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Runtime.html#availableProcessors()>) que sua máquina  tem. Dá para alterar isso na prop: `jdk.virtualThreadScheduler.parallelism`. Esse `ForkJoinPool` especificamente é tunado de uma maneira diferente de um [pool normal](<[common pool](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ForkJoinPool.html#commonPool())>), que opera usando LIFO.

## Mais

# Referências


[JEP 444: Virtual Threads](https://openjdk.org/jeps/444)

[State of Loom](https://cr.openjdk.org/~rpressler/loom/loom/sol1_part1.html)

[Performance and scalability analysis of Java IO and NIO based server models, their implementation and comparison, Karabyn Petro. 2019](https://er.ucu.edu.ua/bitstream/handle/1/4470/Petro%20Karabyn.pdf?sequence=1)

[Virtual threads: Are futures a thing of the past? - Java Magazine](https://blogs.oracle.com/javamagazine/post/virtual-threads-futures)

[The Ultimate Guide to Java Virtual Threads - Rock the JVM Blog](https://blog.rockthejvm.com/ultimate-guide-to-java-virtual-threads)/

[CompletableFuture with Virtual threads](https://davidvlijmincx.com/posts/virtual-threads-with-completablefuture)/

[Java Virtual Threads](https://medium.com/codex/java-virtual-threads-9fad6c362890) - Esse não foi usado diretamente no texto, mas foi uma boa fonte de conhecimentos.

https://www.youtube.com/watch?v=YQ6EpIk7KgY - Pelo engenheiro chefe responsável pela concepção da JEP das virtual threads.

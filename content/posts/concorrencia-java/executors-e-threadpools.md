---
title: "Executors, Thread Pools e Futures em Java"
date: 2024-04-4T19:42:37+00:00
# weight: 1
aliases: ["/threadsafe", "/atomic-integer"]
tags: ["Concorrencia", "JAVA"]
series: ["Concorrencia em Java!"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Sabemos que threads são recursos caros, e isso é onde o *object pooling brilha*. E respondendo outra pergunta: cade o processamento assíncrono?"
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
Seja bem vindo, esse daqui é o quarto de 6 posts sobre concorrência em Java. Nosso roteiro é:

1. Threads! Processando em Paralelo e Ganhando Throughput
2. Sincronização de Threads - DeadLocks, Zonas Críticas e Condições de Corrida
3. Concorrência, agora melhor - Classes Thread Safe
**4. Executors, Thread Pools e Futures**
1. CompletableFuture
2. Virtual Threads
---
# Introdução
Sabemos que Threads do JAVA são Wrappers em torno de threads do SO, agora o importante de sabermos com essa informação é termos ciência que threads do SO são **pesadas**, portanto, criá-las a todo momento é inviável, mas é isso que aprendemos até então no [[1. Threads! Processando em Paralelo e Ganhando Throughput]].

Na realidade, existem duas maneiras de lidar com threads:
- Controlar diretamente a criação e gerenciamento das Threads, instanciando uma Thread nova toda vez que a aplicação precisar executar uma tarefa assíncrona.
- Abstrair o gerenciamento de Threads do resto da sua aplicação, passando as tasks para um _executor_.
A segunda abordagem, que vamos estudar agora, já soa melhor pelo poder de encapsular código e lidar com threads de uma maneira mais abstrata, mas não é só isso.

# Executors

O pacote `java.util.concurrent` define 3 interfaces de executors:
- `Executor`, que permite começar novas tasks.
- `ExecutorService`, uma subinterface de um `Executor`, que adiciona features que ajudam a gerenciar o ciclo de vida das tasks e do executor. Quase sempre vamos usar isso aqui!!
- `ScheduledExecutorService`, Subinterface de `ExecutorService`, que permite execução futura ou periódica de tasks.
## Executor
Suponha que *`r`* é um Runnable e *`e`* é um executor

Nesse caso, você pode simplesmente usar:
```java
e.execute(r);
```

As implementações de Executor São menos específicas e normalmente são usados `ExecutorService` e  `ScheduledExecutorService`.
## ExecutorService
A interface [`ExecutorService`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html) interface aceita `execute`, mas também adiciona `submit` como um método um pouco mais versátil, que aceita objetos do tipo `Runnable`,mas também [`Callable`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Callable.html) , **permitindo que  task retorne um valor**

O `submit`retorna uma [`Future`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html) usado para conseguir o retorno de um `Callable`
## ScheduledExecutorService
Assim como executorService, mas com algumas peculiaridades para rodar as tasks no futuro e/ou de maneira recorrente, saiba que exista e estude por fora caso seja necessário!
# Thread Pools

A maioria das implementações no `java.util.concurrent` usam  _pools de Threads_, que são  _worker threads_. Esse tipo de Threads basicamente são desacopladas do código que vão rodar, ou seja, elas rodam algo, mas continuam vivas, elas não nascem e morrem como costumamos fazer quando as instanciamos manualmente, essas threads podem ser usadas inúmeras vezes.
Usar esse tipo de Threads diminui bastante o trabalho da criação de objetos de Thread, que usam muita memória, e criam bastante trabalho no processo de alocação e desalocação.
Quando trabalharmos no Spring Boot, sua aplicação já nasce com um ExecutorService com 200 Worker Threads (na configuração padrão) que serão usadas para atender os chamados ao seu serviço, seguindo o padrão de uma thread processando um request, portanto, na realidade, só conseguimos processar 200 requests em paralelo.

> [🤔] **Pols**
> Um _pool_ de algum recurso computacional é uma "piscina" cheia desse recurso específico, a disposição para uso. Usamos Object Pooling quando estamos lidando com objetos caros de se instanciar e/ou destruir (ou objetos que devem ser frequentemente construídos).
> Um exemplo claro para desenvolvedores JAVA é o *Hikari*, biblioteca que cria um pool de conexões de banco de dados (jdbc) e gerencia essas conexões, fornecendo uma conexão (objeto) disponível quando pedido.

Para criarmos um executorService que será responsável por atender as Tasks que passamos a ele com uma ThreadPool fixa, podemos usar:

```java
  private final ExecutorService exec = Executors.newFixedThreadPool(16);
```

Uma vantagem de usar uma pool de threads fixas, é que garantimos que nossa aplicação sempre processará uma quantidade de requests que consegue simultâneamente, se estivéssemos criando e matando threads a todo momento, seria possível que nossa aplicação parasse totalmente graças ao estouro do limite de recursos disponíveis.

[`java.util.concurrent.Executors`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html) Também oferece outras factories sem ser `newFixedThreadPool(int)`:

- O método [`newCachedThreadPool`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html#newCachedThreadPool-int-).
- O método [`newSingleThreadExecutor`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html#newSingleThreadExecutor-int-) .
- Diferentes outras factories que retornam versões `ScheduledExecutorService` versions dos executores que falamos antes.
# Exemplo: Future e Callable
```java
package CallableFuture;  
  
import java.util.concurrent.Callable;  
import java.util.concurrent.ExecutionException;  
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  
import java.util.concurrent.Future;  
import java.util.concurrent.ThreadLocalRandom;  
  
class Spawner implements Callable<String> {  
    @Override  
    public String call() throws Exception {  
        int random = ThreadLocalRandom.current().nextInt(100,400);  
        Thread.sleep(random);  
        return String.format("Spawned %d creatures", random);  
    }  
}  
  
public class Main {  
  
    public static void main(String[] args) throws ExecutionException, InterruptedException {  
        var spawner = new Spawner();  
        System.out.println("Spawnando:");  
        ExecutorService executorService = Executors.newFixedThreadPool(10);  
        Future<String> futureResponse1 = executorService.submit(spawner);  
        Future<String> futureResponse2 = executorService.submit(spawner);  
        // A resposta literalmente virá no futuro!  
        String response1 = futureResponse1.get();  // espera até que o retorno venha (block the i/o)
        String response2 = futureResponse2.get();  
    }  
}
```

# Futures - O porquê ser assíncrono:

Já comentamos anteriormente sobre os `Futures` e os vimos em ação nesse último exemplo, vamos falar um pouco sobre esse contexto.

De maneira geral, escrevemos código síncrono, onde o programa chega na linha x e executa as instruções naquela linha, o código assíncrono é um pouco diferente, lá, pedimos para algo (nesse caso, uma Thread) que execute essa instrução, e sabemos que isso eventualmente será executado, é uma *promessa* de execução.

Imagine que em um sistema financeiro que usa uma thread para a renderização da tela e a mesma para ouras operações, nesse caso, quando você clicar em "Gerenciar cotação do dolar", sua thread (ou seja, sua aplicação) inteira ficará bloqueada esperando a cotação do dolar chegar (provavelmente por alguma API).

Se essa chamada ocorrer de maneira assíncrona, ela irá travar por alguns nanosegundos, apenas registrando a chamada para a API, mas não necessariamente esperando seu retorno, provavelmente registrando um _callback_ para quando esse retorno realmente acontecer.

Nesse cenário, notamos que para utilizarmos o poder do processamento em paralelo, faz sentido usarmos do assincronismo, caso contrário, estaremos apenas delegando uma tarefa para a outra thread e esperando pelo seu retorno, sem muitos benefícios

Exemplo bobo de como um código pode ser feito mais rápido com multithreading  + assincronismo, lendo dois arquivos paralelamente:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2vn30vzao803sxg2l3x1.png)

Nesse cenário, poderíamos ainda usar mais threads para a realização das operações matemáticas, desde que elas possam ser feitas por múltiplas threads de verdade.


## Timeout
Podemos (e devemos quase sempre) definir timeouts para tarefas assíncronas, evitando que nossas threads fiquem muito tempo (ou indefinidamente) esperando algo acontecer:
`future.get(2, TimeUnit.SECONDS);`

# Referência
https://docs.oracle.com/javase/tutorial/essential/concurrency/exinter.html
https://www.youtube.com/watch?v=y7PUfmtWIXs&list=PL62G310vn6nFIsOCC0H-C2infYgwm8SWW&index=237&ab_channel=DevDojo
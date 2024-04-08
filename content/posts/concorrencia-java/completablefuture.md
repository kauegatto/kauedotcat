---
title: "5. CompletableFuture - Dominando o Assíncrono em Java!"
date: 2024-04-05T20:18:37+00:00
aliases: ["/async", "/completable-future"]
tags: ["Concorrencia", "JAVA"]
series: ["Concorrencia em Java!"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Sabemos como lidar com multiplas threads, evitar race conditions, entendemos pooling e sabemos o porquê devemos usar mecanismos assíncronos, agora, vamos entender a API moderna do JAVA para lidar com comportamentos assíncronos e paralelos."
disableHLJS: false
disableShare: false
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
5. **CompletableFuture**
6. Virtual Threads

# Introdução e "Join"
Suponha esse código:

*StoreService*
```java
package DCompletableFuture;  
  
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Future;  
import java.util.concurrent.ThreadLocalRandom;  
import java.util.concurrent.TimeUnit;  
  
public class StoreService {  
    private final ExecutorService executorService;  
  
    public StoreService(ExecutorService executorService) {  
        this.executorService = executorService;  
    }  
  
    public Double getPricesSync() throws InterruptedException {  
        return getPrices();
    }  
  
    public Future<Double> getPricesAsync() {  
        return executorService.submit(this::getPrices);  
    }  
    private Double getPrices() throws InterruptedException {  
        System.out.println("Getting prices...");  
        TimeUnit.SECONDS.sleep(1);  
        return ThreadLocalRandom.current().nextDouble(20,200)*3;  
    }  
}
```

*Main*
```java
package DCompletableFuture;  
  
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  
  
public class Main {  
    public static void main(String[] args) {  
        var ex = Executors.newFixedThreadPool(2);  
        ex.submit(Main::searchPricesAsync);  
        ex.submit(Main::searchPricesSync);  
    }  
    private static void searchPricesSync(){  
        final ExecutorService ex = Executors.newFixedThreadPool(3);  
        StoreService storeService = new StoreService(ex);  
        try {  
            System.out.println(storeService.getPricesSync());  
            System.out.println(storeService.getPricesSync());  
            System.out.println(storeService.getPricesSync());  
            System.out.println(storeService.getPricesSync());  
            System.out.println(storeService.getPricesSync());  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
    }  
    private static void searchPricesAsync() {  
        final ExecutorService ex = Executors.newFixedThreadPool(5);  
        StoreService storeService = new StoreService(ex);  
        try {  
            System.out.println(storeService.getPricesAsync().get());  
            System.out.println(storeService.getPricesAsync().get());  
            System.out.println(storeService.getPricesAsync().get());  
            System.out.println(storeService.getPricesAsync().get());  
            System.out.println(storeService.getPricesAsync().get());  
        }  
        catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
}
```

No caso, estamos executando as operações de busca sync e async simultaneamente (uma em cada thread) e para cada uma delas, estamos submetendo as tarefas as threads e pegando seus resultados de maneira síncrona e sem usar todas nossas threads.

Pera, mas se getPricesAsync nos retorna uma Future, como isso é síncrono? Embora o processamento esteja sendo realizado paralelamente, nosso código fica realmente bloqueado, esperando pela resposta do `storeService.getPricesAsync()`, e <mark style="background: #D2B3FFA6;">ele só faz o pedido de busca do próximo código assíncrono quando o anterior termina</mark>, portanto, não estamos submetendo todas as tasks antes de esperá-las.

> [❗] **Atenção**
> Note que aqui, o processameto síncrono é o problema pois estamos bloqueando a thread que realiza searchPricesAsync, <mark style="background: #D2B3FFA6;">fazendo com que ela só peça que a próxima execução aconteça quanto a anterior terminar</mark>. Mesmo que isso esteja sendo executado em uma thread auxiliar, estamos esperando a thread terminar seu trabalho para continuarmos o nosso.

A solução seria dividir em duas partes:
```java
private static void searchPricesAsync() {  
    final ExecutorService ex = Executors.newFixedThreadPool( 5);  
    StoreService storeService = new StoreService(ex);  
    Future<Double> pricesAsyncFuture1 = storeService.getPricesAsync();  
    Future<Double> pricesAsyncFuture2 = storeService.getPricesAsync();  
    Future<Double> pricesAsyncFuture3 = storeService.getPricesAsync();  
    Future<Double> pricesAsyncFuture4 = storeService.getPricesAsync();  
    Future<Double> pricesAsyncFuture5 = storeService.getPricesAsync();  
    try {  
        System.out.println(pricesAsyncFuture1.get());  
        System.out.println(pricesAsyncFuture2.get());  
        System.out.println(pricesAsyncFuture3.get());  
        System.out.println(pricesAsyncFuture4.get());  
        System.out.println(pricesAsyncFuture5.get());  
    }  
    catch (Exception e) {  
        e.printStackTrace();  
    }  
}
```
E agora sim temos essa belezinha de resultado:
```
Getting prices... - pool-2-thread-4
Getting prices... - pool-2-thread-3
Getting prices... - pool-2-thread-1
Getting prices... - pool-2-thread-2
Getting prices... - pool-1-thread-2
Getting prices... - pool-2-thread-5
```
Apesar disso, o código não é tão bonito assim, né? Vamos ver uma evolução dos `Futures` - `CompletableFutures`
```java
public CompletableFuture<Double> getPricesWithCompletableFuture() {  
    return CompletableFuture.supplyAsync(() -> {  
        try {  
            return getPrices();  
        } catch (InterruptedException e) {  
            throw new RuntimeException(e);  
        }  
    });  
}
```

Aqui, note que só temos esse try porque getPricesRealmente pode jogar uma InterruptedException.

> Perceba também que não passamos o executor para o supplyAsync (poderíamos), mas ele possuí um executor "nativo":
> **According to the [official documentation](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html), if we use the async methods without explicitly providing an _Executor_, the functions will be executed using _ForkJoinPool.commonPool()._** Therefore, if we run the code snippet, we should expect to see one of the common _ForkJoinPool_ workers: in my case, “_ForkJoinPool.commonPool-worker-1″._ - https://www.baeldung.com/java-completablefuture-threadpool#async-methods

```java
private static void searchPricesAsync() {  
    final ExecutorService ex = Executors.newFixedThreadPool( 5);  
    StoreService storeService = new StoreService(ex);  
    CompletableFuture<Double> pricesAsyncCompletableFuture1 = storeService.getPricesWithCompletableFuture();  
    CompletableFuture<Double> pricesAsyncCompletableFuture2 = storeService.getPricesWithCompletableFuture();  
    CompletableFuture<Double> pricesAsyncCompletableFuture3 = storeService.getPricesWithCompletableFuture();  
    CompletableFuture<Double> pricesAsyncCompletableFuture4 = storeService.getPricesWithCompletableFuture();  
    CompletableFuture<Double> pricesAsyncCompletableFuture5 = storeService.getPricesWithCompletableFuture();  
    
    System.out.println(pricesAsyncCompletableFuture1.join());  
    System.out.println(pricesAsyncCompletableFuture2.join());  
    System.out.println(pricesAsyncCompletableFuture3.join());  
    System.out.println(pricesAsyncCompletableFuture4.join());  
    System.out.println(pricesAsyncCompletableFuture5.join());  
}
```

Aqui, note que já **não precisamos usar try/catch nos *joins***, equivalente ao *.get()* do *CompletableFuture*
## Uso com Streams!

> [☣️] **Cuidado com Streams!**
Ao usarmos CompletableFuture com streams, precisamos ter um cuidado, nossas streams não podem iterar a lista de CompletableFutures usando `join`, senão, voltamos ao cenário do processamento síncrono, o correto é retornar essa stream em duas partes diferentes, uma que realiza o supply das tarefas para as threads e outra que realmente executa o join.

```java
  
public class Main {  
    private static final StoreServiceSync storeService = new StoreServiceSync();  
    private static final Set<String> STORES =  Set.of("BestPrice", "LetsSaveBig", "MyFavoriteShop", "BuyItAll");  
    public static void main(String[] args) {  
        searchPricesWithDiscount();  
        searchPricesWithDiscountAsync();  
    }  
    private static void searchPricesWithDiscount(){  
        long start = System.currentTimeMillis();  
        STORES.stream()  
                .map(store -> storeService.getQuote(store))  
                .peek(System.out::println)  
                .map(quote -> storeService.applyDiscount(quote))  
                .forEach(System.out::println);  
        long end = System.currentTimeMillis();  
        System.out.printf("[SYNC] Time elapsed: %d ms", end - start);  
    }  
  
    private static void searchPricesWithDiscountAsync(){  
        long start = System.currentTimeMillis();  
        List<CompletableFuture<Quote>> quotesFutures = STORES.stream()  
                .map(store -> CompletableFuture.supplyAsync(() -> storeService.getQuote(store)))  
                .toList();  
  
        List<Quote> quotes = quotesFutures.stream()  
                .map(quoteCompletableFuture -> quoteCompletableFuture.join())  
                .peek(System.out::println)  
                .toList();  
  
        List<CompletableFuture<String>> pricesFutures = quotes.stream()  
            .map(currQuote -> CompletableFuture.supplyAsync(() -> storeService.applyDiscount(currQuote)))  
            .toList();  
  
        List<String> prices = pricesFutures  
                .stream()  
                .map(priceFuture -> priceFuture.join())  
                .peek(System.out::println)  
                .toList();  
  
        long end = System.currentTimeMillis();  
        System.out.printf("[ASYNC] Time elapsed: %d ms", end - start);  
    }  
}
```
Aqui, note que tanto as operações de `getQuote` quanto as de `applyDiscount` são assíncronas, portanto, elas foram divididas em duas partes cada
Resultados:
```
Quote[store=MyFavoriteShop, price=60.0, discountCode=GOLD]
MyFavoriteShop price is 54.00
Quote[store=BuyItAll, price=430.0, discountCode=DIAMOND]
BuyItAll price is 344.00
Quote[store=LetsSaveBig, price=1300.0, discountCode=DIAMOND]
LetsSaveBig price is 1040.00
Quote[store=BestPrice, price=150.0, discountCode=SILVER]
BestPrice price is 142.50
[SYNC] Time elapsed: 16031
```

```
Quote[store=MyFavoriteShop, price=4430.0, discountCode=DIAMOND]
Quote[store=BuyItAll, price=180.0, discountCode=DIAMOND]
Quote[store=LetsSaveBig, price=690.0, discountCode=PLATINUM]
Quote[store=BestPrice, price=1670.0, discountCode=SILVER]
MyFavoriteShop price is 3544.00
BuyItAll price is 144.00
LetsSaveBig price is 586.50
BestPrice price is 1586.50
[ASYNC] Time elapsed: 4014 ms
```
O código funcionou bem, mas está bem verboso, vamos melhorá-lo:
# Encadeando chamadas:
## ThenCompose
O método `thenCompose` é utilizado para encadear operações de forma que a segunda operação dependa do resultado da primeira, **retornando um novo `CompletableFuture`**. No exemplo a seguir, utilizamos `thenCompose` para aplicar o desconto após obter a cotação de forma assíncrona:
```java
private static void searchPricesWithDiscountAsyncNew(){  
    long start = System.currentTimeMillis();  
    List<CompletableFuture<String>> stringFutures = STORES.stream()  
            .map(store -> CompletableFuture.supplyAsync(() -> storeService.getQuote(store)))  
            .map(cf -> cf.thenCompose(quote -> CompletableFuture.supplyAsync(() -> storeService.applyDiscount(quote))))  
            .toList();  
  
    stringFutures.stream()  
            .map(stringFuture -> stringFuture.join())  
            .forEach(System.out::println);  
  
    long end = System.currentTimeMillis();  
    System.out.printf("[NEW ASYNC] Time elapsed: %d ms\n", end - start);  
}
```
Neste exemplo, `thenCompose` é utilizado para encadear a obtenção da cotação (`getQuote`) com a aplicação do desconto (`applyDiscount`) de forma assíncrona, resultando em um código mais conciso e legível.

## ThenApply
O método `thenApply` é utilizado quando queremos transformar o resultado de um `CompletableFuture` de forma independente do resultado de outras operações, retornando um novo `CompletableFuture`. Supondo que precisássemos formatar a string do preço com desconto de uma forma específica, poderíamos utilizar `thenApply` da seguinte maneira:

```java
private static void searchPricesWithDiscountAsyncNew() {
    long start = System.currentTimeMillis();
    List<CompletableFuture<String>> stringFutures = STORES.stream()
            .map(store -> CompletableFuture.supplyAsync(() -> storeService.getQuote(store)))
            .map(cf -> cf.thenCompose(quote -> CompletableFuture.supplyAsync(() -> storeService.applyDiscount(quote))))
            .map(cf -> cf.thenApply(discountedQuote -> String.format("Discounted price: %s", discountedQuote)))
            .toList();

    stringFutures.stream()
            .map(stringFuture -> stringFuture.join())
            .forEach(System.out::println);

    long end = System.currentTimeMillis();
    System.out.printf("[NEW ASYNC] Time elapsed: %d ms\n", end - start);
}

```
### ThenApply vs ThenCompose 
Uma analogia para entendermos a diferença é a seguinte:
* `thenApply` é como Function.apply() enquanto `thenCompose()` é como compor funções
* `thenApply` é como um `map` e `thenCompose` é como um `flatMap`:
```java
public CompletableFuture<UserInfo> getUserInfo(userId)
public CompletableFuture<UserRating> getUserRating(UserInfo)


CompletableFuture<CompletableFuture<UserRating>> apply =
    userInfo.thenApply(this::getUserRating);

CompletableFuture<UserRating> compose  =
    userInfo.thenCompose(this::getUserRating);

```

Ou seja, no geral usamos thenApply se temos uma função de mapeamento síncrona e thenCompose no caso oposto.

> You would use `thenCompose` when you have an operation that returns a `CompletionStage` and `thenApply` when you have an operation

### ThenApplyAsync, ThenComposeAsync

Aqui, prefiro referenciar um conteúdo muito bem escrito encontrado [aqui](https://stackoverflow.com/a/46063193):

Imagine for a moment that you have an application that allows users to register themselves and upon registration they will receive a confirmation email to confirm their account.

You don't want the user to be waiting for ever if the mail server is down or if it takes a long time to compose the email or perform additional checks.

You would then use `thenApplyAsync` to fire off the send email logic because it is not crucial to your system. A user can always go back and say "send me another email"

```java
static CompletionStage<String> register(String username) {
    throw new UnsupportedOperationException();
}

static void sendConfirmationEmail(String username) {
    throw new UnsupportedOperationException();
}

public static void main(String[] args) throws InterruptedException {

    register("user").thenAcceptAsync(username -> sendConfirmationEmail(username));

}
```

Here your system will respond when the registration is complete but it will not wait for the email to be sent resulting in improved responsiveness of your system.

## AllOf e AnyOf
Outro ponto importante ao trabalhar com `CompletableFuture` é o uso dos métodos estáticos `allOf` e `anyOf`, que são muito úteis para lidar com múltiplos `CompletableFutures` de forma eficiente.
### `allOf`

O método `allOf` é usado quando você precisa esperar pela conclusão de todos os `CompletableFutures` em uma lista. Ele retorna um **novo** `CompletableFuture` que é concluído somente quando todos os `CompletableFutures` na lista são concluídos, independentemente de sucesso ou falha.

Por exemplo, suponha que você tenha uma lista de `CompletableFutures` que representam operações assíncronas e deseja executar uma ação após a conclusão de todas elas:

```java
List<CompletableFuture<Void>> futures = new ArrayList<>(); futures.add(CompletableFuture.runAsync(() -> System.out.println("Task 1"))); futures.add(CompletableFuture.runAsync(() -> System.out.println("Task 2"))); futures.add(CompletableFuture.runAsync(() -> System.out.println("Task 3")));  CompletableFuture<Void> allFutures = CompletableFuture.allOf(         futures.toArray(new CompletableFuture[0]) );  allFutures.thenRun(() -> System.out.println("All tasks completed"));`
```

Neste exemplo, `allFutures` será concluído quando todas as tarefas assíncronas representadas pelos `CompletableFutures` na lista `futures` forem concluídas.
### `anyOf`

Por outro lado, o método `anyOf` é usado quando você deseja esperar pela conclusão de apenas um dos `CompletableFutures` em uma lista. Ele retorna um novo `CompletableFuture` que é concluído assim que um dos `CompletableFutures` na lista é concluído, independentemente de sucesso ou falha.

```java
List<CompletableFuture<Void>> futures = new ArrayList<>(); futures.add(CompletableFuture.runAsync(() -> System.out.println("Task 1"))); futures.add(CompletableFuture.runAsync(() -> System.out.println("Task 2"))); futures.add(CompletableFuture.runAsync(() -> System.out.println("Task 3")));  

  
CompletableFuture<Object> anyFuture = CompletableFuture.anyOf( futures.toArray(new CompletableFuture[0]) );

anyFuture.thenAccept(result -> System.out.println("One task completed"));
```

Neste exemplo, `anyFuture` será concluído assim que uma das tarefas assíncronas representadas pelos `CompletableFutures` na lista `futures` for concluída.

# Referências

https://www.baeldung.com/java-completablefuture-threadpool#async-methods
https://www.youtube.com/watch?v=ZKDgjM_x4bo&list=PL62G310vn6nFIsOCC0H-C2infYgwm8SWW&index=241&ab_channel=DevDojo
https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html
https://dzone.com/articles/understanding-lazy-evaluation-in-java-streams
---
title: "Classes Thread Safe em Java -  Conceito e Introdução"
date: 2024-04-1T19:25:03+00:00
# weight: 1
aliases: ["/threadsafe", "/atomic-integer"]
tags: ["Concorrencia", "JAVA"]
series: ["Concorrencia em Java!"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "O JAVA nos traz diversos facilitadores para trabalharmos com concorrência em Java, aqui vamos descobir o que são as classes thread-safe."
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
Seja bem vindo, esse daqui é o terceiro de 6 posts sobre concorrência em Java. Nosso roteiro é:

1. Threads! Processando em Paralelo e Ganhando Throughput
2. Sincronização de Threads - DeadLocks, Zonas Críticas e Condições de Corrida
**3. Concorrência, agora melhor - Classes Thread Safe**
4. Executors E Thread Pools
5. CompletableFuture
6. Virtual Threads

---
**Disclaimer:** Esse post em específico usa como principal referência o curso **grátis** de JAVA Do [DevDojo](https://www.youtube.com/@DevDojoBrasil/videos), chamado Java virado no jiraya, **que está publicado no youtube!**. Recomendo muito assistirem os vídeos e fazerem os exemplos com o grandessísimo William Suane.

# Introdução

Até agora, estávamos trabalhando e vendo threads de maneira mais cru, criando threads na mão e trabalhando com ela em um nível bem baixo. Apesar disso, essa não é a maneira convencional de trabalhar com ćodigos concorrentes em Java. A linguagem adicionou um pacote, em `java.util.concurrent` que é uma camada de abstração para trabalhar com esses recursos de maneira mais eficiente, vamos começar a entender esse pacote:

# AtomicInteger

Suponha que temos um jogo que trabalha de maneira que precisam ser spawnados 45 mil personagens em um mapa no início do jogo, e para isso, vamos usar algumas threads para fazer esse spawn:
```java
public class Spawner implements Runnable {  
    private final int amountToSpawn;  
  
    public Integer getAmountSpawned() {  
        return amountSpawned;  
    }  
  
    private Integer amountSpawned = 0;  
  
    public Spawner(int amountToSpawn) {  
        this.amountToSpawn = amountToSpawn;  
    }  
  
    @Override  
    public void run() {  
        System.out.println("Spawning...");  
        for (int i = 0; i < amountToSpawn; i++) {  
            this.amountSpawned++;  
        }  
    }  
}

public class Main {  
    public static void main(String[] args) throws InterruptedException {  
        System.out.println("Spawnando:");  
        Spawner spawner = new Spawner(15000);  
  
        var t1 = new Thread(spawner);  
        var t2 = new Thread(spawner);  
        var t3 = new Thread(spawner);  
  
        t1.start();t2.start();t3.start();  
        t1.join();t2.join();t3.join();  
  
        System.out.println("Spawned: " + spawner.getAmountSpawned());  
    }  
}```

Curiosamente, nosso resultado é:

`Spawned: 30718`

Mas sincronizando, temos a quantidade correta:

```java
public class Spawner implements Runnable {  
    private final Integer amountToSpawn;  
    private final Object sync = new Object();  
  
    public Integer getAmountSpawned() {  
        return amountSpawned;  
    }  
  
    private Integer amountSpawned = 0;  
  
    public Spawner(int amountToSpawn) {  
        this.amountToSpawn = amountToSpawn;  
    }  
  
    @Override  
    public void run() {  
        System.out.println("Spawning...");  
        for (int i = 0; i < amountToSpawn; i++) {  
            synchronized (sync){  
                this.amountSpawned++;  
            }  
        }  
    }  
}
// Spawned: 45000
```

Apesar disso, fica meio obvio que nossa performance está degradada, ao invés disso, podemos usar `AtomicInteger` e esquecer os problemas:

```java
import java.util.concurrent.atomic.AtomicInteger;  
  
public class Spawner implements Runnable {  
    private final Integer amountToSpawn;  
    public AtomicInteger getAmountSpawned() {  
        return amountSpawned;  
    }  
  
    private final AtomicInteger amountSpawned = new AtomicInteger(0);  
  
    public Spawner(int amountToSpawn) {  
        this.amountToSpawn = amountToSpawn;  
    }  
  
    @Override  
    public void run() {  
        System.out.println("Spawning...");  
        for (int i = 0; i < amountToSpawn; i++) {  
            this.amountSpawned.incrementAndGet();  
        }  
    }  
}

```
Esses objetos em java que encapsulam essas lógicas de maneira geral possuem mais performance por implementar algoritmos que evitem condições de corrida de maneiras eficientes. é sempre legal utilizá-los

# Lock e ReentrantLock

No exemplo anterior de AtomicInteger, quando usamos synchronized para tornar o método viável, note que usei um objeto cru, somente para cuidar da questão do sincronismo.
Os locks na realidade servem exatamente para isso, mas com algumas outras vantagens:

* Fairness: No construtor, podemos especificar um `fairness` booleano, que diz que o mecanismo de lock deve **tentar** "passar o bastão" para a thread que está esperando a entrada na zona crítica a mais tempo

* Mecanismo de "tentativa": Tente acessar o recurso por x tempo, caso contrário, vá embora

* Possibilidade de interromper a thread que espera pelo recurso

```java
package BLocks;  
  
import java.util.concurrent.locks.Lock;  
import java.util.concurrent.locks.ReentrantLock;  
  
public class Spawner implements Runnable {  
    private final Integer amountToSpawn;  
    private final Lock lock = new ReentrantLock();  
    public Integer getAmountSpawned() {  
        return amountSpawned;  
    }  
  
    private Integer amountSpawned = 0;  
  
    public Spawner(int amountToSpawn) {  
        this.amountToSpawn = amountToSpawn;  
    }  
  
    @Override  
    public void run() {  
        System.out.println("Spawning...");  
        for (int i = 0; i < amountToSpawn; i++) {  
            lock.lock();  
            this.amountSpawned++;  
            lock.unlock();  
        }  
    }  
}
```

Como uma exceção pode ocorrer no pedaço bloqueado, o ideal é usarmos try/finally!

```java
    try {  
        lock.lock();  
        this.amountSpawned++;  
  
    }  
    finally {  
        lock.unlock();  
    }
```

**Problema**: o código fica feio pra caralho! Normalmente usamos o synchronized por conta disso, a menos que você precise de `fairness` ou das outras funções que especificamos.

**Um ponto importante aqui, que vai ser explicado com mais detalhes no capitulo sobre virtual threads é que o uso de `synchronized` pode ser um grande problema ao trabalhar com virtual threads, incluidas oficialmente no Java 21+**

Se você é adepto ao lombok, saiba que foram adicionadas duas annotations bem recentemente: `Locked` e `synchronized`, especialmente locked, pode tornar seu código mais bonito.

Para mais detalhes sobre Locks, Conditions, CopyOnWriteArrayList, ArrayBlockingQueue, LinkedTransferQueue, ReentrantReadWriteLock (Toma conta de leitura e escrita), etc, recomendo ver a playlist do devDojo, vou parar de roubar os exemplos dele (nesse assunto) por aqui!
# Referências
* Andre Leon, meu professor de S.O!!
* https://www.youtube.com/@DevDojoBrasil
* https://fidelissauro.dev/concorrencia-paralelismo/

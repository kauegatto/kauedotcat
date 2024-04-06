---
title: "1. Concorrência em Java: Threads! Processando em Paralelo e Ganhando Throughput"
date: 2024-03-29T19:00:03+00:00
# weight: 1
aliases: ["/concorrencia-threads"]
tags: ["Concorrencia", "JAVA"]
series: ["Concorrencia em Java!"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Vamos explorar como fazer testes em java!"
canonicalURL: "https://canonical.url/to/page"
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

Seja bem vindo, esse daqui é o primeiro de 6 posts sobre concorrência em Java. Nosso roteiro é:

**1. Threads! Processando em Paralelo e Ganhando Throughput**
2. Sincronização de Threads - DeadLocks, Zonas Críticas e Condições de Corrida
3. Concorrência, agora melhor - Classes Thread Safe
4. Executors, Thread Pools e Futures
5. CompletableFuture
6. Virtual Threads

# Contexto

<mark style="background: #D2B3FFA6;">Threads são unidades de execução dentro de um processo</mark>. <mark style="background: #D2B3FFA6;">Um processo é um programa em execução que contém pelo menos uma thread.</mark> As threads permitem que um programa execute várias tarefas ao mesmo tempo, alocando uma thread em cada processador disponível.

# **Vantagens de programar com múltiplas threads:**
Uma das principais razões para usar múltiplas threads é melhorar o desempenho de um programa. Tarefas pesadas e demoradas podem ser divididas em threads separadas, permitindo que diferentes partes do programa sejam executadas em paralelo. Isso pode levar a uma utilização mais eficiente dos recursos da CPU e, consequentemente, a um tempo de resposta mais rápido.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/penw7eg1a1bfdvjuuew8.png)

No entanto, programar com threads também traz desafios, como a necessidade de lidar com concorrência (quando várias threads tentam acessar ou modificar os mesmos recursos ao mesmo tempo) e a possibilidade de erros difíceis de depurar (como as condições de corrida), pois os resultados de um mesmo código não serão necessariamente os mesmos (não determinísticos).
# O multithreading ajuda ou não?

**1. Operações de I/O:**
Quando um programa precisa realizar operações de entrada/saída -- I/O (e elas são o gargalo), como leitura/gravação de arquivos, comunicação com bancos de dados ou solicitações de rede, <mark style="background: #D2B3FFA6;">há frequentemente momentos em que a CPU fica ociosa</mark>, esperando que os dados sejam lidos ou escritos.
Nessa situação, se uma nova thread tomasse conta da situação, ela não seria mais executada pelo processador enquanto estivesse ociosa, pois aconteceria o que chamamos de troca de contexto, que é basicamente fazer com que outra thread seja processada. Isso permite que outras threads que necessitem de processamento real tenham suas operações executadas pelos núcleos da CPU, ou até mesmo lançar (ou usar) mais threads para já lançar outras chamadas que também exigem esse tempo de espera, conhecidas como <mark style="background: #D2B3FFA6;">bloqueantes</mark>. Isso ajuda a aproveitar melhor o tempo da CPU, **melhorando a eficiência geral do programa.**
Imagine um contexto onde você precisa ler dois arquivos .txt, essa operação poderia ser realizada paralelamente se lançássemos duas threads, uma para ler cada arquivo, sendo cada uma processada em um núcleo, diminuindo o tempo de execução essencialmente pela metade

**2. Código CPU-bound:**
Quando o programa está executando tarefas intensivas em CPU, como cálculos matemáticos complexos, simulações ou processamento de imagem, uma única thread pode não ser capaz de aproveitar totalmente a capacidade de processamento da CPU. Dividir essas tarefas em threads separadas permite que múltiplos núcleos da CPU trabalhem em paralelo, acelerando o processamento.
Nesse caso, devemos tomar cuidado, pois a quantidade de tarefas que pode ser paralelizada realmente é igual a quantidade de núcleos do seu processador (lógicos + físicos).


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bstgdyjfdfyg8rj92gjj.png)


A imagem acima representa a troca de contexto, note que esse processo não é necessariamente instantâneo e resulta em possível perda de cache, o que pode ser agressor à performance - [Fonte](https://wiki.inf.ufpr.br/maziero/lib/exe/fetch.php?media=socm:socm-05.pdf).
> A frequência de trocas de contexto tem impacto na eficiência do sistema operacional: quanto menor o número de trocas de contexto e menor a duração de cada troca, mais tempo sobrará para a execução das tarefas em si. Assim, é possível definir uma medida de eficiência E do uso do processador, em função das durações médias do quantum de tempo *t* e da troca de contexto *c*.

# Java: Threads!
## O Objeto Thread

O objeto `java.lang.Thread` é um *wrapper* em cima das threads do sistema operacional

> [❗] **Importante**
> Note que as Threads são objetos wrappers em torno das threads do SO, portanto, se essas threads do S.O são pesadas (e são), as Threads em Java também são.

Em Java, podemos trabalhar com threads de algumas maneiras, a primeira que veremos é com a classe Thread, essas classes precisam dar o override do método `run`:

```Java
class ThreadExample extends Thread{
  char c;
  public ThreadExample(char c) {
    this.c = c;
  }

  @Override
  public void run() {
    System.out.printf("\nComeçouuu!: %s\n", c);
    for (int i = 0; i < 100 ; i++) {
      System.out.print(c);
    }
  }
}
```

```Java
public static void main(String[] args) {
    /* Todo programa em execução é "feito" de threads, esse não é uma exceção*/
    Thread.currentThread().getName();
    ThreadExample t1 = new ThreadExample('A');
    ThreadExample t2 = new ThreadExample('B');
    ThreadExample t3 = new ThreadExample('C');
    t1.run();
    t2.run();
    t3.run();
  }
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9ed6s7jk91lxnzgyrl4p.png)



Pronto! (Só que não) → Note que os objetos thread ainda estão rodando na mesma thread, nesse caso, usar `Thread.run()` executa o método run, nao inicia a thread, nesse caso, devemos rodar `start()`!

```Java
public class Thread01 {
  public static void main(String[] args) {
    /* Todo programa em execução é "feito" de threads, esse não é uma exceção*/
    Thread.currentThread().getName();
    ThreadExample t1 = new ThreadExample('A');
    ThreadExample t2 = new ThreadExample('B');
    ThreadExample t3 = new ThreadExample('C');
    t1.start();
    t2.start();
    t3.start();
  }
}
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ys8g283085yvfm7jr8am.png)

> [❓] **Reflexão**
> Criar um objeto do tipo thread faz sentido? Você está especializando uma thread realmente? A herança faz sentido nesse caso? [[2. SOLID]]
## 1. Interface Runnable

Nesse caso, acho válido começar diferente, vamos ler uma parte da javadoc da classe runnable
## 0. Javadoc

> The Runnable interface should be implemented by any class whose instances are intended to be executed by a thread. The class must define a method of no arguments called run.
>
> In addition, Runnable provides the means for a class to be active while not subclassing Thread. A class that implements `Runnable` can run without subclassing Thread by instantiating a Thread instance and passing itself in as the target.
> **<mark style="background: #D2B3FFA6;">In most cases, the Runnable interface should be used if you are only planning to override the run() method and no other Thread methods. This is important because classes should not be subclassed unless the programmer intends on modifying or enhancing the fundamental behavior of the class.</mark>**

A documentação do JAVA responde perfeitamente a reflexão anterior, se você discorda, pode seguir em frente, mas *particularmente* acho que é um argumento difícil de rebater.

Exemplo:
```Java
class ThreadRunnable implements Runnable {
  char c;

  public ThreadRunnable(char c) {
    this.c = c;
  }

  @Override
  public void run() {
    System.out.printf("\nComeçouuu!: %s\n", c);
    for (int i = 0; i < 100; i++) {
      System.out.print(c);
    }
  }
}

public class Thread01 {
  public static void main(String[] args) {
    /* Todo programa em execução é "feito" de threads, esse não é uma exceção*/
    Thread.currentThread().getName();

    var t1Runnable = new ThreadRunnable('a');
    var t2Runnable = new ThreadRunnable('b');
    var t3Runnable = new ThreadRunnable('c');
    Thread t1 = new Thread(t1Runnable);
    Thread t2 = new Thread(t2Runnable);
    Thread t3 = new Thread(t3Runnable);
    t1.start();
    t2.start();
    t3.start();
  }
}
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xtufpjsnxr59ynqmzoky.png)

# Estados de uma thread

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jh06kptf646zydrr2cge.png)

É interessante sabermos disso, pois podemos dar dicas para o S.O como dizer para que uma thread running pare, ou notificando que uma thread se tornou *Runnable*.
# Melhorando o Código
Se não precisarmos de construtor! podemos usar uma  lambda, pois Runnable é uma `@FunctionalInterface`:

```Java
Thread t1 = new Thread( () -> {/*codigo*/});
```
Ou, um pouco mais verboso:
```java
Runnable simplerRunnable = () -> {
      System.out.printf("\nComeçouuu!: %s\n", c);
      for (int i = 0; i < 100; i++) {
        System.out.print(c);
      }
};
```
## Prioridade

Prioridades podem ser atribuídas à threads, conforme mostra o código:

```Java
Thread t3 = new Thread(t3Runnable,"nomeC");
t3.setPriority(Thread.MAX_PRIORITY);
```


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zt3ej75r8mh5riv86y5z.png)


> [❗] **Importante**
> Note que prioridades são indicações do que você deseja para o scheduler, uma thread de prioridade 1 pode rodar andar da prioridade 10, você não deve desenvolver um código baseado em prioridade

## Sleep

Imagine que você deseja que uma thread ocorra sem fim, mas rode a cada 2 minutos, como pode fazer isso? 🤔

Uma das maneiras é usar um `Thread.sleep(milis)` e pedir para que a thread pare por algum tempo, note que é importante esse código estar dentro de um try-catch, por sua possibilidade de gerar uma exceção (caso a thread seja interrompida, por exemplo)

```Java
@Override
  public void run() {
    System.out.printf("\nComeçouuu!: %s\n", c);
    for (int i = 0; i < 100; i++) {
      System.out.print(c);
    }
    try {
      Thread.sleep(1000);
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    }
  }
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wv4gzoo2utb9e5boydkr.png)


## Yield

Yield serve para indicarmos / darmos uma **<mark style="background: #D2B3FFA6;">dica</mark>** para o scheduler do JVM faça a thread voltar para Runnable (pare) por um tempo. [[2. Começando com o Código]]

O *yield* é um dos principais elementos que permitem a existência de *Virtual Threads*.
## Join


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9i0k24gf578muvwe4mud.png)

Join serve para avisarmos a thread main que ela deve **esperar** para continuar seu fluxo q uando as operações terminarem
Quando você chama o método `join` em uma determinada (thread), você está essencialmente dizendo: "Pera ai, só continua quando essa tarefa terminar". Isso é útil quando você tem partes do programa que precisam estar totalmente prontas antes que outras partes possam prosseguir.
Um exemplo seria um cenário onde você precisa comparar 3 pesquisas de viagem de avião para conseguir ver o preço mais barato, você pode dar o `join` nas 3 threads que rodaram essa operação de I/O (a ordem não irá importar, pois estaremos limitados pela última de qualquer jeito) e então depois comparamos os resultados

Resumidamnete, o `join` é especialmente útil quando você precisa garantir a ordem correta das operações ou quando precisa coletar resultados de várias threads antes de prosseguir.

```Java
// t1 roda antes de t1 e t2
Thread t1 = new Thread(new ThreadRunnableYieldJoin('A'));
Thread t2 = new Thread(new ThreadRunnableYieldJoin('B'));
Thread t3 = new Thread(new ThreadRunnableYieldJoin('C'));

t1.start();
try {
  t1.join();
} catch (InterruptedException e) {
  throw new RuntimeException(e);
}
var threads = List.of(t2,t3);
threads.forEach(Thread::start);
```

```Java
public static void main(String[] args) {
// t1 e t2 em paralelo
    Thread t1 = new Thread(new ThreadRunnableYieldJoin('A'));
    Thread t2 = new Thread(new ThreadRunnableYieldJoin('B'));
    Thread t3 = new Thread(new ThreadRunnableYieldJoin('C'));

    t1.start();
    t2.start();

    try {
      t1.join();
      t2.join();
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    }

    t3.start();
  }
}
```
# Referências

> [📚] **Qual a finalidade do Transient e Volatile no Java?**
> _As vezes quando vou declarar meus atributos noto o transient e o volatile._
> [https://pt.stackoverflow.com/a/116080](https://pt.stackoverflow.com/a/116080)

> [📚] **Maratona Java Virado no Jiraya**
> _Melhor, maior, e o mais completo curso de Java em português grátis de toda Internet está de volta._
> [https://www.youtube.com/playlist?list=PL62G310vn6nFIsOCC0H-C2infYgwm8SWW](https://www.youtube.com/playlist?list=PL62G310vn6nFIsOCC0H-C2infYgwm8SWW)>)


---
title: "2. Sincronização de Threads - DeadLocks, Zonas Críticas e Condições de Corrida"
date: 2024-03-31T19:16:03+00:00
# weight: 1
aliases: ["/deadlocks", "/race-conditions", "desafios"]
tags: ["Concorrencia", "JAVA"]
series: ["Concorrencia em Java!"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Os benefícios associados à programação paralela e concorrente são muitos, mas os perigos também!"
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

Seja bem vindo, esse daqui é o segundo de 6 posts sobre concorrência em Java. A série é focada em Java, mas esse post em especial apresenta conceitos relevantes para literalmente todas as linguagens e também não é uma leitura muito extensiva :).

Nosso roteiro é:

1. Threads! Processando em Paralelo e Ganhando Throughput
2. **Sincronização de Threads - DeadLocks, Zonas Críticas e Condições de Corrida**
3. Concorrência, agora melhor - Classes Thread Safe
4. Executors, Thread Pools e Futures
5. CompletableFuture
6. Virtual Threads

# Sincronização de Threads

Um assunto muito abordado em diversos cursos e disciplinas, até mesmo arquitetura de computadores e sistemas operacionais é o sincronismo de threads? Mas por quê?
Esse tópico vai ser relativamente teórico, mas bem importante, juro.

# Exemplo:

Um problema clássico de sincronismo (nesse caso sendo retratado por uma implementação Do [DevDojo](https://www.youtube.com/@DevDojoBrasil)) é exemplo que envolve saque monetário em uma mesma conta, por threads diferentes.
Quando threads diferentes acessam um mesmo recurso, acontece o que chamamos de _condição de corrida_. Esse fenômeno pode ocasionar em erros gravíssimos e difíceis de se perceber, olhe:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6fdl3s5facb7yzz8nxdh.png)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u6pqyujbeodxke9jr4le.png)

Note que ambos vão sacar dinheiro ao mesmo tempo quando tem 10 reais an conta, caindo na condição da balance ser igual a quantia, apesar disso, como ambos estão correndo e acessando uma mesma _zona crítica_, acontece uma inconsistência, onde uma thread desconta 10 de 10, e o saldo fica 0, e a outra desconta 10 de 0, deixando o saldo negativo.

Como podemos resolver isso?!

## Operações Atômicas

Uma operação atômica, (na programação concorrente e/ou paralela), é uma operação indivisível que é executada em sua totalidade ou não é executada de forma alguma. Isso significa que, quando uma operação é marcada como atômica, **==ela é tratada como uma unidade indivisível de execução, mesmo em um ambiente com múltiplas threads ou processos concorrentes.==** Isso é importante pois ao trabalharmos com programação paralela ou concorrente, tentamos dividir a carga em partes menores, enviadas para as threads

A atomicidade é fundamental para evitar condições de corrida e garantir a consistência dos dados compartilhados entre threads ou processos. Em uma operação atômica, não há possibilidade de que outra thread ou processo interrompa a operação no meio do caminho, o que reduz significativamente o risco de conflitos e resultados indesejados.

Exemplos:
1. **Incremento e Decremento**: A operação de incremento ou decremento de uma variável é geralmente implementada como uma operação atômica para evitar condições de corrida ao modificar a mesma variável de diferentes threads.
2. **Troca (Swap)**: A operação de troca de valores entre duas variáveis é frequentemente implementada de forma atômica para garantir que a troca ocorra completamente sem interrupções.
3. **Teste e Definição (Test-and-Set)**: Uma operação que verifica o valor de uma variável e a define para um novo valor se a condição for atendida, tudo de forma atômica.
4. **Operações de Bloqueio e Desbloqueio (Locking/Unlocking)**: Operações de bloqueio e desbloqueio são frequentemente usadas para garantir que uma seção crítica do código seja executada por apenas uma thread por vez, evitando conflitos. Essas operações são normalmente implementadas de forma atômica.

## A zona crítica

A "_zona crítica_" diz respeito à uma seção de código onde uma thread acessa ==recursos compartilhados==, como variáveis, memória ou objetos, que **não** devem ser modificados por outras threads concorrentes. Essa região protegida, normalmente, é acessada por uma thread / programa de cada vez.
O objetivo é tornar a operação sobre o recurso compartilhado [atômica](https://pt.wikipedia.org/wiki/Transa%C3%A7%C3%A3o_at%C3%B4mica "Transação atômica"). Uma região crítica geralmente termina num tempo específico, e uma linha de execução ou [processo](https://pt.wikipedia.org/wiki/Processo_(inform%C3%A1tica) "Processo (informática)") só precisa esperar um tempo específico para entrá-la. Alguns mecanismos de [sincronização](https://pt.wikipedia.org/wiki/Sincroniza%C3%A7%C3%A3o "Sincronização") são necessários para implementar a entrada e a saída de uma região crítica para assegurar o uso exclusivo, como por exemplo um [semáforo](https://pt.wikipedia.org/wiki/Sem%C3%A1foro_(computa%C3%A7%C3%A3o) "Semáforo (computação)"), é o que veremos mais a frente.

1. Condição de Corrida
	i. Acontece quando duas ou mais threads tentam modificar um recurso compartilhado ao mesmo tempo, resultando em resultados não determinísticos e possivelmente errôneos.
2. Deadlocks
    i. Muitas vezes os próprios synchronizers (algorítmos que alternam o acesso à recursos por threads) causam o deadlock.
    ![Imagem mostrando um deadlock, os recursos t1 está do lado esquerdo de um circulo, e o t2 no lado direito, o recurso t1 tenta acessar o recurso r1 e obtém o lock, o t2 faz o mesmo com r2, mas r1 espera infinitamente por r2 e t2 por r1](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mjjhbdumdmprb9d4h6i1.png)



3. Inconsistências

Um dos fatores para isso acontecer é o dado cache! Podemos usar recursos do java como "Volatile" que diz que sempre que formos acessar aquele recurso, ele tem que ser verificado de novo!

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8cdcx5pejtjzl1t14a8c.png)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/msv5ltlzzodf41ixmuku.png)




- Caso StackOverflow - **Muiito Didático (Explica volatile)**
- [https://pt.stackoverflow.com/a/116080](https://pt.stackoverflow.com/a/116080)

    ```Java
    long i = 0;
    void thread1() {
      ...
      i++;
      ...
    }
    void thread2() {
      ...
      if (i == 1) {
        fazAlgo();
      }
      ...
    }
    ```

No exemplo, as duas threads acessam a mesma variável. Assumindo que a leitura da `thread2`  ocorre, numa sequência de tempo, exatamente após o incremento da `thread1`, você acha que pode ocorrer algum problema de concorrência, considerando que o incremento **parece uma operação atômica**?

Uma análise ingênua diria que está tudo bem com as _threads_ pois as duas executam operações atômicas de escrita e leitura, logo `fazAlgo` seria executado sem problemas.

**Errado.**

Cada _thread_ **pode** estar sendo executada em um processador diferente. Cada processador **pode** ter um _cache_ próprio. Variáveis são lidas e gravadas primeiro no cache local antes de irem para a memória principal. Então, **é possível** que a segunda _thread_ leia o valor antigo da variável.

O cenário problemático ocorreria assim:

1. T1 lê o valor de `i = 0` da memória principal e faz o incremento; o novo valor `i = 1` é armazenado no cache local, mas não na memória principal.
2. T2 lê o valor de `i = 0` da memória principal e não entra no `if`.
Pior que isto, variáveis de 64 bits como `long` e `double` podem ter sua escrita em memória dividida pela JVM em dois ciclos de 32 bits, o que poderia levar uma leitura completamente corrompida de seus valores.

**Tais cenários são relativamente raros, mas extremamente difíceis de identificar em softwares complexos, causando aquele tipo de problema intermitente e ocasional que acaba sendo varrido para debaixo do tapete.**

A solução, neste caso, é simples:

```Java
volatile long i = 0;
```

Um atributo volátil tem garantia de que o valor atualizado estará sempre disponível para outras _threads_, sendo gravado na memória principal assim que atualizado, de forma atômica.

Isso significa que, sempre que o valor for modificado em um processador, ocorrerá um _flush_ para a memória principal, portanto as outras _threads_ vão ver sempre o valor mais atualizado e não um possível valor defasado.

Claro que isso não é gratuito. Fazer o `flush` do cache para a memória principal penaliza o desempenho, afinal existe uma razão para os fabricantes de hardware colocarem caches nos processadores. É muito mais rápido acessar um registrador ou cache primário do que acessar a memória RAM.

Uma solução alternativa seria usar métodos de sincronização como um bloco `synchronized` ou variáveis atômicas como `AtomicLong`, os quais podem ser necessários quando há modificação concorrente, mas que são mais lentos.

No caso de escrita concorrente, como bem lembrado pelo Rafael na outra resposta, uma variável `volatile` ainda poderia incorrer em condição de corrida pois as duas threads podem ler o mesmo valor da memória principal, e o valor final dependeria de qual das threads escreveria ele por último.
# Mecanismos que regulam acesso à zonas críticas
Existem diversos mecanismos que regulam acessos às zonas críticas do software, evitando condições de corridas - *race conditions*:
## Locks

Locks são o mecanismo padrão, que basicamente dizem se alguém tem pode entrar ou não na zona crítica, é como aquele banheiro químico, ou você pode entrar, ou tem alguém lá dentro e você tem que esperar pra entrar.
## Semáforos
* Apesar de ter sido inventado em 1965, por E. Dijkstra, os semáforos são a técnica mais usada atualmente
* Um semáforo é uma variável (s), associada a uma região crítica, sobre a qual podem incidir duas operações:
1. Operação Down: verifica se o valor de s é maior que zero.  Se for, o valor é decrementado. Senão, a tarefa é bloqueada e o valor de s permanece zero.
2. Operação Up: incrementa o valor de s, e desbloqueia as demais tarefas se o valor for zero.

Ou seja, um semáfaro com valor *s* inicial de 3, permite que 3 tarefas entrem na zona crítica (3 Downs, descendo o valor para 2,1 e 0)

Também existem Semáforos Conhecidos como binários ou *mutex*, que é basicamente um semáforo de valor *s*=1, ou seja, acomoda apenas um único thread, é um lock convencional :p.
# Mecanismos em Java:
### Syncronized
O mais comum é usarmos a keyword `syncronized` antes da declaração de algum método ou variável, indicando que ela tem um lock, ou seja, apenas uma thread pode acessá-la por vez, como uma passagem de bastão
Então, no exemplo anterior, adicionar `synchronized` resolve nosso problema:
```Java
private synchronized void withdrawal(int amount) {
    if (account.getBalance() >= amount) {
      System.out.println(getThreadName() + " está indo sacar dinheiro");
      account.withdrawal(amount);
      System.out.println(getThreadName() + " completou o saque, valor atual da conta " + account.getBalance());
    } else {
      System.out.println("Sem dinheiro para " + getThreadName() + " efetuar o saque " + account.getBalance());
    }
  }
```


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6orr8qa7n38oi2x8e53n.png)


> [❗] **Importante**
> Sleeps e outras operações demoradas não liberam o lock / passa o bastão para outra thread, na realidade, essas threads esperando sua vez ficam bloqueadas na "fila"

Podemos também usar o synchronized assim: dividindo uma parte da operação como atômica, mas deixando a outra aberta para múltiplas threads. **Nesse caso marcamos qual objeto sofrerá o lock e a operação**

```Java
private void withdrawal(int amount) {
        System.out.println(getThreadName() +" #### fora do synchronized");
        synchronized (account) {
            System.out.println(getThreadName() +" **** dentro do synchronized");
            if (account.getBalance() >= amount) {
                System.out.println(getThreadName() + " está indo sacar dinheiro");
                account.withdrawal(amount);
                System.out.println(getThreadName() + " completou o saque, valor atual da conta " + account.getBalance());
            } else {
                System.out.println("Sem dinheiro para " + getThreadName() + " efetuar o saque " + account.getBalance());
            }
        }
    }
```

Nesse caso, estamos sincronizando apenas o objeto account, um ponto de atenção aqui é não trocar a referência desse objeto: como fazer `account = new Account()` por isso, uma boa prática é marcar objetos sincronizados como `final`.

> [🤓☝️] **Mutex / Locks Distribuídos**
> Alguns sistemas podem optar por usar locks distribuídos entre diferentes processos, isso pode ser feito direto no banco de dados (se for uma aplicação como uma API com múltiplas instâncias rodando) ou usando soluções como o ***Apache Zookeper***, que possuí recursos avançados para tomar conta dos seus processos, evitando *starvation* & *race conditions*

# Thread-Safe

Classes "thread-safe" são classes ou componentes de software projetados para funcionar de maneira segura em ambientes multithread, onde várias threads podem acessá-los e manipulá-los simultaneamente. Em outras palavras, uma classe thread-safe é projetada para evitar condições de corrida, deadlocks e outras situações problemáticas que podem ocorrer quando várias threads acessam recursos compartilhados.


> [❗] **Threads safe que... não são thread safe**
> Usar uma classe como Collections.synchronizedList não garante que a classe é thread-safe se uma camada a mais de código não thread-safe for colocada em cima dela (Exemplo, uma classe que faz o add e remove, mas não se importa com a sincronização desses métodos).

Uma classe realmente Thread safe:
```Java
class ThreadSafeNames {
    private final List<String> names = new ArrayList<>();

    public synchronized void add (String name){
        names.add(name);
    }

    public synchronized void removeFirst(){
        if(names.size() > 0){
            System.out.println(Thread.currentThread().getName());
            System.out.println(names.remove(0));
        }
    }
}

public class ThreadSafeTest01 {
    public static void main(String[] args) {
        ThreadSafeNames threadSafeNames = new ThreadSafeNames();
        threadSafeNames.add("Junkrat");
        Runnable r = threadSafeNames::removeFirst;
        new Thread(r).start();
        new Thread(r).start();
    }
}
```

> [🤓☝️] **Classes Úteis**
> Diversas classes do package java.util.concurrent são úteis para concorrência, exemplos:
> 1. ConcurrentHashMap
> 2. CopyOnWriteArrayList (Uma lista thread-safe em que as operações de leitura não requerem sincronização, tornando-as eficientes para leitura intensiva.)
> 3. AtomicInteger (Uma classe que fornece operações atômicas para incrementar e atualizar inteiros.)
> 4. Semaphore - Semáforos!
> 5. Exchanger: Uma classe que permite que duas threads troquem objetos em um ponto de encontro, facilitando a comunicação entre threads.

## Exemplo: Envio de Email
Podemos ter um serviço para envio de email, onde temos uma thread colocando emails em uma fila, e sempre notificando as worker-threads quando um email novo chegar, assim agilizando o envio por múltiplas threads.
```Java
public class Members {
    private final Queue<String> emails = new ArrayBlockingQueue<>(10);
    private boolean open = true;

    public boolean isOpen() {
        return open;
    }

    public int pendingEmails() {
        synchronized (emails) {
            return emails.size();
        }
    }

    public void addMemberEmail(String email) {
        synchronized (this.emails) {
            String threadName = Thread.currentThread().getName();
            System.out.println(threadName + " Adicionou email na lista");
            this.emails.add(email);
            this.emails.notifyAll();
        }
    }

    public String retrieveEmail() throws InterruptedException {
        System.out.println(Thread.currentThread().getName() + " checking if there are emails");
        synchronized (this.emails) {
            while (this.emails.size() == 0) {
                if (!open) return null;
                System.out.println(Thread.currentThread().getName() + " Não tem email disponível na lista, entrando em modo de espera");
                this.emails.wait();
            }
            return this.emails.poll();
        }
    }

    public void close() {
        open = false;
        synchronized (this.emails) {
            System.out.println(Thread.currentThread().getName() + " Notificando todo mundo que não estamos mais pegando emails");
        }
    }
}
```

> [❗] **Atenção!**
> addMemberEmail e retrieveEmail podem ser usados ao mesmo tempo: sim, eles podem ser usados ao mesmo tempo. Embora ambos usem o mesmo objeto (this.emails) como bloqueio, eles estão bloqueando diferentes partes críticas do código. Enquanto addMemberEmail está bloqueando para adicionar um email, retrieveEmail está bloqueando para verificar e remover um email. Isso permite que esses métodos sejam chamados simultaneamente sem interferir um no outro.

```Java
public class EmailDeliveryService implements Runnable{
    private final Members members;

    public EmailDeliveryService(Members members) {
        this.members = members;
    }

    @Override
    public void run() {
        String threadName = Thread.currentThread().getName();
        System.out.println(threadName +" starting to deliver emails...");
        while(members.isOpen() || members.pendingEmails() > 0){
            try {
                String email = members.retrieveEmail();
                if(email == null) continue;
                System.out.println(threadName + " enviando email para " + email);
                Thread.sleep(2000);
                System.out.println(threadName + " enviou email com sucesso para "+ email);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("Todos os emails foram enviados com sucesso!");
    }
}
```

```Java
public class EmailDeliveryTest01 {
    public static void main(String[] args) {
        Members members = new Members();
        Thread jiraya = new Thread(new EmailDeliveryService(members), "Jiraya");
        Thread kakashi = new Thread(new EmailDeliveryService(members), "Kakashi");
        jiraya.start();
        kakashi.start();
        while(true){
            String email = JOptionPane.showInputDialog("Entre com seu email");
            if(email == null || email.isEmpty()){
                members.close();
                break;
            }
            members.addMemberEmail(email);
        }
    }
}
```

# Exemplo Clássico : Jantar Dos Filósofos (Deadlock)

https://blog.pantuza.com/artigos/o-jantar-dos-filosofos-problema-de-sincronizacao-em-sistemas-operacionais

O jantar dos filósofos foi pensado por Dijkstra (esse cara realmente foi foda né)

Cinco filósofos estão sentados em uma mesa redonda para jantar. Cada filósofo tem um prato com espaguete à sua frente. Cada prato possui um garfo para pegar o espaguete. O espaguete está muito escorregadio e, para que um filósofo consiga comer, será necessário utilizar dois garfos.

Lembre-se que é apenas uma analogia. Nesse sentido, cada filósofo alterna entre duas tarefas: **comer** ou **pensar**. Quando um filósofo fica com fome, ele tenta pegar os garfos à sua esquerda e à sua direita; um de cada vez, independente da ordem. Caso ele consiga pegar **dois garfos**, ele come durante um determinado tempo e depois recoloca os garfos na mesa. Em seguida ele volta a pensar.

_Você é capaz de propor um **algoritmo** que implemente cada **filósofo** de modo que ele execute as tarefas de **comer** e **pensar** sem **nunca** ficar **travado**?_

Não vou colocar a solução aqui, mas se te deixou curioso, acesse o link acima, é uma ótima explicação. A solução normalmente aceita é usar os semáforos que falamos anteriormente

# Agradecimentos Especiais
* Obrigado André Leon, professor de S.O que me introduziu bem à esses conceitos, me perdoe se não usei algum termo corretamente professor 🙏.
* Obrigado especial ao [Matheus Fidelis](https://fidelissauro.dev/concorrencia-paralelismo/), que fez um post super completo sobre concorrência e paralelismo, mais desvinculado da linguagem
* Obrigado especial também ao [William Suane](https://www.youtube.com/@DevDojoBrasil), um dos responsáveis por uma nova geração de Javeiros competentes no mundo
# Referências
* https://www.youtube.com/@DevDojoBrasil
* https://fidelissauro.dev/concorrencia-paralelismo/
* https://blog.pantuza.com/artigos/o-jantar-dos-filosofos-problema-de-sincronizacao-em-sistemas-operacionais
* https://pt.stackoverflow.com/a/116080
* https://pt.wikipedia.org/wiki/Regi%C3%A3o_cr%C3%ADtica
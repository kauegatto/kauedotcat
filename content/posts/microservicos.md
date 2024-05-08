---
title: "O que são microserviços? Para que servem e quando usar?"
date: 2024-05-07T20:14:12+00:00
# weight: 1
aliases: ["/ms", "microservices"]
tags: ["ARCHITECTURE", "Microservicos"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Introdução boa sobre microserviços! Aqui temos 🤓☝️"
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
## Disclaimer:
Esse artigo é baseado totalmente no Livro _Building Microservices_, do Sam Newman!
## Visão Geral

**<mark style="background: #FF5582A6;">Microserviços são partes independentes entre si que são modeladas em torno de uma regra de negócio.</mark>** Um serviço encapsula uma funcionalidade e permite que ela seja acessível por uma rede através de requisições REST.

Microserviços são um tipo de arquiteturas orientadas a serviço, onde fronteiras entre serviços devem ser traçadas, mas apesar disso o release independente é chave.

Do lado de fora, um microserviço é uma caixa preta, exceto por seus endpoints expostos.

> **[❗] Importante**
> Como o ideal é que a implementação interna do microserviço seja alheia ao mundo exterior, cada microserviço idealmente deve se importar apenas com sua linguagem de programação, com seu banco de dados e outros detalhes internos de implementação. Não necessariamente isso será a verdade em um mundo real, nesses casos, precisamos nos preocupar principalmente com acessos concorrentes. Locks distribuídos podem ser uma tacada boa para lidar com recursos compartilhados entre múltiplos serviços 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ja2qxphiurwbuhr1uq61.png)

Como microserviços se baseiam muito na ocultação de informação, expondo somente o necessário através de interfaces externas, permitem uma maior visão no que pode ou não mudar facilmente. O código interno de um microserviço pode mudar a vontade desde que as interfaces externas não mudem seu comportamento.
---

# Pontos-Chave sobre microserviços!

## Deploy Independente

Essa é a ideia que conseguimos mudar um microserviço e dar deploy nele sem termos que fazer deploy em qualquer outro microserviço, na realidade, é o jeito que devemos tratar os microserviços, não somente o que podemos fazer.

Para isso acontecer, devemos ter certeza que nossos microserviços estão pouco acoplados . Isso significa que o contrato entre os serviços devem ser explícitos e estáveis. Algumas escolhas de implementação podem tornar isso inviável, como por exemplo o compartilhamento de banco de dados.

## Modelados ao redor de um domínio do negócio

Microserviços são modelados ao redor de seu domínio, para resolver um problema em específico. Isso ajuda bastante na separação e no entendimento do domínio.

## Donos de seu próprio estado

Microserviços devem ter dentro de si todos os seus dados importantes, por isso o tópico anterior é importante na hora de delimitar as funções de um microserviço. Um microserviço deve ser capaz de conversar com outro para pedir a ele alguma informação que não possui, isso deve acontecer via a interface externa desse outro microserviço, o que garante o deploy independente desses serviços

## Flexibilidade

Microserviços te compram opções, no sentido que elas tem um preço. Mais microserviços, na via de regra, te oferecem mais flexibilidade, contudo, também são mais difíceis de manusear. O tamanho dos microserviços não é algo tão importante quanto a pergunta de quantos microserviços você consegue administrar e de que forma você faz isso sem que eles sejam uma bagunça acoplada.

---

### Maneira de trabalho com microserviços


![Maneira tradicional. Quando não temos um microserviço, uma mudança no back ou no front de uma aplicação monolítica afeta tudo!](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1ycgj3wpdmwcgmr7ytvm.png)


![Maneira boa!](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ulq7ql93xse070dlldal.png)

## Tecnologias são importantes, mas cuidado

Normalmente é mais produtivo ir incluindo novas tecnologias que dão suporte à microserviços conforme seu sistema cresça e mais serviços apareçam, contudo, algumas ferramentas são essenciais, como por exemplo um agregador de log, ou outras ferramentas que são capazes de visualizar traces entre multiplo serviços, detectar gargalos, etc.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3km5575n2fhulg5mq8hx.png)

---

## Containers e Kubernetes

Idealmente, cada microserviço deve ser rodado isoladamente, garantindo que um problema em um microserviço não possa afetar outro, por isso, containers são a maneira mais viável de se utilizar os microserviços.

Depois de implementar os microserviços. provavelmente será dificícil de administrá-los, por isso, é ideal a utilização de ferramentas que servem para a orquestração desses containers, <mark style="background: #FF5582A6;">kubernetes</mark> é a ferramenta mais conhecida para tal.

## Streaming

Embora microserviços sejam uma maneira de migrar de bases de dados monolíticas, ainda precisamos encontrar maneiras de transferir informações entre microserviços, ao mesmo tempo que organizações buscam cada vez mais feedbacks em tempo real. Existem muitas ferramentas que ajudam em tal cenário, o <mark style="background: #FF5582A6;">apache kafka e rabbitMQ</mark> são ferramentas comuns para isso.

# Vantagens de microserviços

## Tecnologias Heterogêneas

Com um sistema composto por diversas partes que conseguem trabalhar isoladamente e apenas externalizar uma interface, conseguimos programar essas partes com tecnologias totalmente diferentes, desde que o contrato entre os serviços sejam mantidos. Isso nos permite escolher a ferramenta correta para cada serviço.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xryykiiz5hszvihaz7v6.png)

## Robustez

Microserviços são capazes de lidar com falhas em determinados setores específicos sem com que toda a aplicação morra ( o que usualmente acontece em monolitos ). Com os microserviços implementados corretamente, conseguimos lidar com uma falha total em um serviço e apenas fazer com que uma parte do sistema não funcione perfeitamente. Se o sistema de busca da netflix falhar, você ainda é capaz de assistir filmes, criar contas, realizar pagamentos e afins.

## Scaling

Uma arquitetura baseada em microserviços possibilita o “scaling” de coisas isoladamente, onde podemos dedicar mais hardware a partes que realmente necessitam daquilo, e sistemas mais fracos a grupos de serviços menos utilizados.

O autor dá um exemplo do marketplace multimilionário focado em modas “Gilt’, e diz que essa foi a principal razão para a Gilt aderir aos microserviços, o mesmo diz que a empresa hoje possui mais de 450 microserviços.

## Facilidade no Deploy

Uma alteração em uma linha de código em um sistema monolito normalmente pede que o sistema todo passe por um deploy novamente, contudo, graças a característica de deploys independentes, isso não acontece com os microserviços

## Alinhamento organizacional

Essa questão é óbvia para times grandes de desenvolvimento, separar equipes em partes específicas que trabalham com produtos que podem ser melhorados sem necessitar grande movimentações de outras equipes desde que o contrato pré estabelecido continue sendo respeitado é maravilhoso.

## Composição

Microserviços podem ser usados em diferentes ambientes e usados em conjuntos para obtermos resultados específicos, ajudam no reúso de código e na eficiência.

# **Pontos de Dor** / _Fraquezas_

> [📋 Citação]
> _We’ll be covering many of these issues in depth throughout the rest of the book—in fact, I’d argue that the bulk of this book is about dealing with the pain, suffering, and horror of owning a microservice architecture.”_ - Autor do Livro Building Microservices, usado como base.

## Experiência do Desenvolvedor.

Quando você vai incluindo mais serviços, cria-se uma situação mais complexa para rodar os serviços em máquinas locais, 5/6/7 serviços baseados em JVM podem rodar, mas quando chegamos nas dezenas, a situação é mais trabalhosa, inclusive com runtimes que são menos taxativos que a JVM.

Outro ponto importante é que mudanças em charts, terraform, vulnerabilidades que devem ser aplicadas em múltiplos serviços podem ser uma dor de se administrar.

## Sobrecarga de tecnologias

Não é porque microserviços te dão a opção de rodar diferentes tecnologias, bancos, linguagens que você deve fazê-lo, são possibilidades, não requisitos.

Além disso, diversas empresas, equipes ou desenvolvedores individuais desenvolvem um amor por novas tecnologias que às vezes não valem a pena, que servirão apenas para incluir mais complexidade em um escopo definido.

## Report de Dados

Report de dados com microserviços são mais complicados, principalmente pelo fato de termos (idealmente) banco de dados quebrados em diferentes partes, onde uma simples operação de join não resolve as coisas de uma vez.

### Segurança, Monitoramento, Testes

São basicamente a mesma coisa, temos diversos microserviços conversando entre si, vários fluxos a serem monitorados, os testes e2e ficam muito difíceis. Esses assuntos vao ser tratados com mais detalhes depois.

## Consistência de dados

Respeitar a consistência de dados em um sistema muito mais distribuído, com muitos processos independentes, que muitas vezes cuidam de uma base de dados única é difícil. Transações distribuídas existem mas costumam ser problemáticas, o ideal é utilizarmos tecnologias como sagas.

# Onde é complicado usar?

- Empresas que possuem times excessivamente pequenos
- Empresas que não conseguem arcar com os custos iniciais de colocar diversas máquinas na cloud rodando serviços
- Empresas as quais o produto não está pensado, está muito sucetível a mudanças
    - Vai lascar bastante com os boundaries

1. Microservices need to talk to each other through n/w packets. Monolith is mostly IPC and hence faster.
2. Monitoring all the Microservices (100s) and checking their health to ensure smooth running is a pain.
3. Monolith need not worry about consistency since they primarily operate on the same data store.
4. Monolith in general have better latency since the non-determinism of microservices is not present

# Onde eles funcionam bem?

- Empresas com diversos desenvolvedores
    - Permite que um dev ou um time não entre no caminho do outro;
- Empresas que possuem um aporte inicial bom de dinheiro
- Software que precisa da possibilidade de escalar, e idealmente para diversas regiões do mundo
- Softwares que precisam de flexibilidade.

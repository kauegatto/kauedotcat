---
title: "[WIP] Replicação de Banco de Dados"
date: 2024-01-03T16:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["ARCHITECTURE"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Vamos explorar como fazer testes em java!"
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
# [WIP] Replicação de Banco de Dados

A Replicação refere-se ao processo de manter uma cópia dos mesmos dados em várias máquinas conectadas através de uma rede. Existem várias razões para replicar dados:

- Para manter os dados geograficamente próximos aos usuários, reduzindo a latência.
- Para manter a funcionalidade do sistema mesmo que partes falhem (Tolerância à falhas), aumentando a disponibilidade.
- Para escalar o número de máquinas read-only, aumentando assim o throughput de leitura.

Para esse artigo, vamos supor que o conjunto de dados seja pequeno o suficiente para que cada máquina possa conter uma cópia completa. Em outro momento podemos discutir o particionamento de banco de dados (sharding).

Se os dados à serem replicados forem estáticos, a replicação é direta: Copie os dados para cada nó uma vez. No entanto, os dados frequentemente mudam ao longo do tempo.

Vamos explorar três algoritmos populares para replicar mudanças entre nós: replicação com um único líder, múltiplos líderes e sem líder.

<aside>
💡 Réplica: Cada nó armazenando uma cópia do banco de dados.

</aside>

# Líderes e Seguidores

Como garantimos que todos os dados sejam replicados com precisão? Uma solução comum é a chamada replicação baseada em líder:

**Papel do Líder:** Uma réplica é designada como líder (também conhecida como mestre ou primário). **Para escritas** no banco de dados, **os clientes devem enviar solicitações ao líder**, que grava os novos dados no seu armazenamento local.

**Papel do Seguidor:** As outras réplicas, conhecidas como seguidores (réplicas de leitura), recebem mudanças de dados do líder como parte de um log de replicação ou fluxo de mudança. Cada seguidor atualiza seu banco de dados local aplicando todas as escritas na mesma ordem que o líder.

<aside>
💡 Os clientes podem ler do banco de dados consultando o líder ou qualquer um dos seguidores. No entanto, operações de escrita são aceitas apenas no líder (seguidores são somente leitura do ponto de vista do cliente).

</aside>

# 1. Replicação Baseada em único Líder

A replicação baseada em único líder é um recurso integrado nativamente na maioria dos grandes bancos de dados relacionais e alguns bancos de dados não relacionais. Ela também é usada em corretores de mensagens distribuídas como Kafka e RabbitMQ.

## Replicação Síncrona Versus Assíncrona

Um aspecto importante dos sistemas replicados é se a replicação é síncrona ou assíncrona. Considere um usuário atualizando a imagem do seu perfil em um site. A sequência de eventos é:

- O cliente envia a solicitação de atualização ao líder.
- O líder recebe a solicitação e encaminha a mudança de dados aos seguidores.
- O líder notifica o cliente sobre a atualização bem-sucedida.

O momento que o sucesso será dado ao usuário depende da sincronização dos seguidores.

- Se a replicação for síncrona, os seguidores devem primeiros ficar consistentes para que o usuário receba seu ok.
- Se for assíncrona, os dados **eventualmente serão consistentes** - Isso significa que alguns bancos de dados ainda manterão os dados antigos por um tempo, mas eles serão atualizados para alcançar o estado do líder eventualmente
    - A letra “**E”** do acrônimo BASE, usado frequentemente para explicar um tipo específico de banco de dados que preza por disponibilidade diz respeito à Eventual Consistency - A Consistência eventual
- Há circunstâncias em que os seguidores podem ficar vários minutos ou mais atrás do líder; por exemplo, se um seguidor estiver se recuperando de uma falha, se o sistema estiver operando próximo à capacidade máxima, ou se houver problemas de rede entre os nós.

## Totalmente Síncrono ou Totalmente Assíncrono

Frequentemente, a replicação baseada em líder é completamente assíncrona - Isso significa que uma escrita **não é garantida como durável** (se o líder falhar e não for recuperável, **suas mudanças não serão propagadas mesmo que tenham sido confirmadas ao cliente**). No entanto, **uma configuração totalmente assíncrona tem a vantagem de que o líder pode continuar processando escritas, mesmo que todos os seus seguidores estejam atrasados.**

- Isso é bom para certos cenários onde queremos alta disponibilidade e velocidade, e perder alguns registros não é o fim do mundo
- Em sistemas com muitas leituras, o atraso não é necessariamente um problema e pode ser considerado usar diversas cópias assíncronas, receber um tweet com 2 minutos de atraso não é grande coisa,
- Enfraquecer a durabilidade dos dados pode parecer uma má troca, mas a replicação assíncrona é amplamente utilizada, especialmente se houver muitos seguidores ou se eles estiverem geograficamente distribuídos.

A replicação síncrona garante que os dados sejam replicados **para um número especificado de seguidores antes de confirmar o sucesso ao cliente**. Isso significa que a configuração não é binária,  e você não precisa seguir a linha de seguidores totalmente síncronos.

Na prática, ter todos os seguidores síncronos é impraticável, pois qualquer falha de nó paralisaria o sistema. Tipicamente, um seguidor é síncrono, com os outros sendo assíncronos. **Isso garante uma cópia atualizada dos dados em pelo menos dois nós: o líder e um seguidor síncrono.** Se o seguidor síncrono se tornar indisponível ou lento, um dos seguidores assíncronos é feito síncrono.

## Falha do Líder: Failover

Lidar com a falha do líder é um desafio: um dos seguidores precisa ser promovido a novo líder, os clientes precisam ser reconfigurados para enviar suas escritas ao novo líder, e os outros seguidores precisam começar a consumir mudanças de dados do novo líder. Esse processo é chamado de failover.

O failover pode acontecer manualmente (um administrador é notificado de que o líder falhou e toma as medidas necessárias para criar um novo líder) ou automaticamente. 

O processo de failover se dá dessa forma:

- Determinar que o Líder Falhou.
- Escolher um Novo Líder. Isso pode envolver um processo de eleição entre as réplicas restantes, ou um novo líder pode ser nomeado por um nó controlador previamente eleito. O melhor candidato é frequentemente a réplica com as mudanças de dados mais atualizadas do antigo líder, para minimizar a perda de dados.
- Reconfigurar o Sistema para o Novo Líder. Os clientes agora precisam enviar suas solicitações de escrita ao novo líder. Se o antigo líder voltar, ele deve se tornar um seguidor e reconhecer o novo líder, já que pode ainda acreditar que é o líder.

### Problemas

Se estiver usando replicação assíncrona, o novo líder pode não ter todas as escritas do antigo líder antes da sua falha. Se o antigo líder se reintegrar, há uma questão sobre o que acontece com suas escritas não replicadas, especialmente se o novo líder recebeu escritas conflitantes. Comumente, as escritas não replicadas do antigo líder são descartadas, violando potencialmente as expectativas de durabilidade, o que já discutimos.

Descartar escritas é perigoso se estiver coordenando com sistemas de armazenamento externos fora do banco de dados. Por exemplo, um incidente no GitHub envolveu a promoção de um seguidor desatualizado do MySQL a líder. O banco de dados usava um contador autoincrementado para chaves primárias, mas o contador do novo líder estava atrasado, levando à reutilização de chaves primárias previamente atribuídas pelo antigo líder. Essas chaves também eram usadas em um armazenamento Redis, resultando em inconsistências entre o MySQL e o Redis e causando a divulgação de dados privados para usuários incorretos.

## Problemas com Atraso na Replicação

Ser capaz de tolerar falhas de nós é apenas uma razão para querer replicação. Outras razões incluem escalabilidade (processar mais solicitações do que uma única máquina pode lidar) e latência (colocar réplicas geograficamente mais próximas dos usuários).

A replicação baseada em líder exige que todas as escritas passem por um único nó, mas consultas somente de leitura podem ser direcionadas a qualquer réplica. Em cargas de trabalho que são principalmente de leitura com é comum criar muitos seguidores e distribuir as solicitações de leitura entre eles. Essa abordagem reduz a carga sobre o líder e permite atender solicitações de leitura por réplicas próximas.

Infelizmente, ler de um seguidor assíncrono pode resultar em informações desatualizadas se o seguidor estiver atrasado. Isso leva a inconsistências aparentes no banco de dados: a mesma consulta executada no líder e em um seguidor simultaneamente pode produzir resultados diferentes porque nem todas as escritas foram refletidas no seguidor. 

Essa **inconsistência temporária** é conhecida como consistência eventual. O termo “eventualmente” é intencionalmente vago. Geralmente, não há limite para o quanto uma réplica pode ficar para trás.

Quando o atraso se torna significativo, pode se tornar um problema real, é vamos ver alguns problemas e soluções a seguir

### Ler Suas Próprias Escritas

Muitas aplicações permitem que os usuários enviem dados (como um registro em um banco de dados de clientes ou um comentário em um fórum de discussão) e depois vejam o que enviaram. Novos dados devem ser enviados ao líder, mas a visualização dos dados pode ser feita a partir de um seguidor. Isso é apropriado se os dados são frequentemente visualizados, mas apenas ocasionalmente escritos.

Com a replicação assíncrona, surge um problema: Se o usuário visualizar os dados logo após uma escrita, os novos dados podem ainda não estar na réplica. Para o usuário, parece que sua alteração não foi feita com sucesso.

Nessa situação, é válida a consistência de de ler-suas-próprias-escritas. Isso garante que, se o usuário recarregar a página, ele sempre verá quaisquer atualizações que tenha enviado. Isso não promete visibilidade imediata das atualizações de outros usuários, mas não dá a falsa impressão que suas alterações foram um fracasso.

Ao ler algo que o usuário possa ter modificado, obtenha-o do líder (Ou de um seguidor síncrono!). Caso contrário, use um seguidor. Isso requer um método para determinar modificações potenciais sem realmente consultar. Por exemplo, em uma rede social, o perfil de um usuário normalmente é editável apenas por ele mesmo. Assim, uma regra simples é: sempre leia o perfil do próprio usuário do líder e os perfis de outros usuários de um seguidor.

## Leituras Monotônicas

Uma segunda anomalia com seguidores assíncronos é a possibilidade de os usuários perceberem o tempo retrocedendo. Isso ocorre se um usuário ler de diferentes réplicas em momentos diferentes. Por exemplo, um usuário pode consultar um seguidor com pouco atraso e depois um seguidor com maior atraso, resultando nele vendo dados mais recentes primeiro e depois dados mais antigos.

Para alcançar leituras monotônicas, garanta que cada usuário sempre leia da mesma réplica, embora diferentes usuários possam ler de réplicas diferentes.

### Leituras de Prefixo Consistentes

O terceiro exemplo de anomalias devido ao atraso na replicação envolve violações de causalidade. Se algumas partições replicarem mais lentamente do que outras, é possível observar o resultado (resposta) antes da causa (pergunta).

Para mais detalhes sobre os últimos tópicos, consulte "Designing Data-Intensive Applications" (DDIA), página 164 e 165.

# 2. Replicação Multi-Líder

A principal limitação da replicação baseada em líder é sua restrição de único líder: todas as escritas devem passar por um líder. Se você não puder se conectar ao líder, talvez devido a uma interrupção de rede, não poderá escrever no banco de dados.

Uma extensão natural é a replicação multi-líder (também conhecida como replicação ativa/ativa), onde mais de um nó pode aceitar escritas. A replicação ocorre como de costume: cada nó que processa uma escrita encaminha a mudança de dados para todos os outros nós.

## Casos de Uso para Replicação Multi-Líder

Usar uma configuração multi-líder dentro de um único datacenter raramente é benéfico devido à sua complexidade. No entanto, pode ser vantajoso em certos cenários:

### Operação Multi-datacenter

Considere um banco de dados com réplicas em vários datacenters, seja para tolerância a falhas ou proximidade com os usuários. Em uma configuração padrão baseada em líder, um datacenter abriga o líder, e todas as escritas devem passar por ele.

Em uma configuração multi-líder, cada datacenter pode ter seu próprio líder, enquanto o líder de cada datacenter replica mudanças para os líderes nos outros datacenters.

Vamos comparar como as configurações de líder único e multi-líder se saem em uma implantação multi-datacenter:

### Clientes com Operação Offline

Aplicações que precisam funcionar enquanto desconectadas da internet, como aplicativos de calendário em telefones celulares e laptops. Esses aplicativos requerem a capacidade de visualizar e inserir dados a qualquer momento, independentemente da conectividade com a internet, com mudanças sincronizadas quando o dispositivo estiver novamente online.
**Implementação:** Cada dispositivo tem um banco de dados local atuando como um líder, com um processo de replicação multi-líder assíncrona entre todas as réplicas do dispositivo. O atraso na replicação pode variar de horas a dias.

Semelhante à replicação multi-líder entre datacenters, mas com cada dispositivo como um "datacenter".

## Desvantagens da Replicação Multi-Líder:

Modificações concorrentes podem levar a conflitos de escrita que precisam ser resolvidos, isso não necessariamente é simples e é o principal lado negativo da replicação com múltiplos líderes. Esse padrão é até mesmo considerado negativo na maior parte dos casos.

Chaves autoincrementadas, gatilhos e restrições de integridade podem ser problemáticos, tornando a replicação multi-líder uma escolha arriscada para casos de uso comuns.

> *For example, autoincrementing keys, triggers, and integrity constraints can be problematic. For this reason, multi-leader replication is often considered dangerous territory that should be avoided if possible.*

Se o seu caso de uso pede (ou realmente vê benefícios na arquitetura com múltiplos líderes, recomendo a leitura do livro DDIA no capítulo correspondente para entender com detalhes os desafios e recomendações de como lidar com os conflitos.

## Comparativos

### Desempenho

Configuração de Líder Único: Cada escrita deve passar pela internet até o datacenter com o líder, potencialmente adicionando latência significativa e contrariando o propósito de múltiplos datacenters.
Configuração Multi-Líder: As escritas são processadas no datacenter local e replicadas de forma assíncrona para outros datacenters, ocultando o atraso da rede inter-datacenter e potencialmente melhorando o desempenho percebido.

### Tolerância a Falhas de Datacenter

Líder Único: Se o datacenter com o líder falhar, o failover pode promover um seguidor em outro datacenter a líder.
Multi-Líder: Cada datacenter continua operando independentemente, com a replicação se atualizando quando o datacenter com falha voltar a funcionar.

# Replicação Sem Líder

As abordagens de replicação discutidas até agora — líder único e multi-líder — baseiam-se no envio de solicitações de escrita para um nó (o líder), com o sistema de banco de dados copiando essa escrita para outras réplicas.

O líder determina a ordem das escritas, e os seguidores as aplicam nessa ordem. A replicação sem líder, uma abordagem diferente, abandona o conceito de líder e permite que qualquer réplica aceite escritas diretamente dos clientes. Este estilo, repopularizado pelo sistema Dynamo da Amazon, também é encontrado outras soluções como o Cassandra.

Em alguns sistemas sem líder, os clientes enviam escritas diretamente para várias réplicas, enquanto em outros, um nó coordenador faz isso em nome do cliente.

## Escrevendo no Banco de Dados Quando um Nó Está Inativo

Considere um banco de dados com três réplicas, sendo uma delas atualmente indisponível. Em uma configuração baseada em líder, a continuação do processamento de escritas pode exigir um failover. Em contraste, uma configuração sem líder não tem failover. O cliente envia a escrita para todas as três réplicas, e as duas disponíveis a aceitam. Se o reconhecimento de duas em três réplicas for suficiente, a escrita é considerada bem-sucedida, apesar de uma réplica ter perdido.

Quando o nó indisponível volta a funcionar, ele não possui as escritas feitas durante seu tempo de inatividade. Leituras desse nó podem retornar valores desatualizados. Para resolver isso, **as solicitações de leitura são enviadas a vários nós simultaneamente, com números de versão usados para determinar o valor mais recente.**

## Reparo de Leitura e Anti-Entropia

Para garantir a consistência eventual, dois mecanismos são usados:

- Reparo de Leitura: Quando um cliente lê de vários nós e detecta respostas desatualizadas, ele escreve o valor mais novo de volta para o nó com informações desatualizadas. Isso é eficaz para valores frequentemente lidos.
- Processo Anti-Entropia: Alguns repositórios de dados executam um processo em segundo plano para encontrar diferenças de dados entre réplicas e copiar os dados ausentes de acordo. Este processo não segue uma ordem específica para copiar escritas e pode ter um atraso significativo antes que os dados sejam replicados.

## Operação Multi-Datacenter

A replicação entre datacenters, conforme discutido no contexto da replicação multi-líder, também é aplicável na replicação sem líder. Este modelo é projetado para lidar com escritas concorrentes conflitantes, interrupções de rede e picos de latência.

### Implementação em Cassandra

Em Cassandra, o suporte multi-datacenter é integrado ao modelo sem líder:

O número total de réplica, *n*, inclui nós em todos os datacenters.

- A configuração permite especificar o número de réplicas em cada datacenter.
- Cada escrita do cliente é enviada para todas as réplicas, independentemente da localização do datacenter.
- Os clientes normalmente aguardam o reconhecimento de um quórum de nós dentro do seu datacenter local, minimizando o impacto dos atrasos de link entre datacenters.
- As escritas para outros datacenters costumam ser configuradas para serem assíncronas, com alguma flexibilidade de configuração.

## Detectando Escritas Concorrentes

Bancos de dados estilo *Dynamo*, que permitem escritas concorrentes na mesma chave por vários clientes, enfrentam conflitos semelhantes à replicação multi-líder ("Tratamento de Conflitos de Escrita" na página 171). Conflitos podem surgir durante o reparo de leitura ou a transferência sugerida.

Atrasos variáveis na rede e falhas parciais podem levar a diferentes nós recebendo eventos em ordens diferentes. Por exemplo:

1. O Nó 1 recebe uma escrita do Cliente A, mas perde a escrita do Cliente B devido a uma falha.
2. O Nó 2 primeiro recebe a escrita do Cliente A, seguida pela do Cliente B.
3. O Nó 3 recebe primeiro a escrita do Cliente B e depois a do Cliente A.

Se cada nó simplesmente sobrescrever valores ao receber solicitações de escrita, inconsistências permanentes surgem, como ilustrado no cenário final de solicitação de obtenção: O Nó 2 vê o valor final de X como B, enquanto os outros o veem como A.

Para alcançar a consistência eventual e convergir para o mesmo valor, é necessário um mecanismo de resolução de conflitos adequado. Infelizmente, muitas implementações de banco de dados exigem um profundo entendimento de seus mecanismos internos de tratamento de conflitos por parte do desenvolvedor da aplicação para evitar perda de dados.

Para informações detalhadas sobre a resolução de conflitos de escrita, consulte o livro de referência na página 171.

# Bibliografia

Designing Data-Intensive Applications (DDIA) - Martin Kleppmann. https://dataintensive.net/

https://fidelissauro.dev/teorema-cap/
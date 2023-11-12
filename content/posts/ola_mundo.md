---
title: "Como eu fiz meu blog!"
date: 2023-11-11T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["Blog"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Aprenda como consegui construir e deployar um blog em poucas horas."
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
# Contexto
Escrevo conteúdos há um tempo para mim mesmo em ferramentas como notion ou também para outros, em sites como dev.to. Apesar disso, sempre senti a vontade de ter um domínio próprio. Olhando o famoso [http.cat](http.cat) percebi que existe o domínio do catalão `.cat`, meu sobrenome sendo gatto e já tendo ouvido "kaue cat" por ai, só comprei o domínio. Não queria gastar mais do que os 20 reais do domínio, então decidi hospedar no github, ai que começa a jornada de um site estático.
# SSG's
Sites estáticos são compostos por conteúdos e arquivos que possuem uma estrutura específica definida nele e sempre irão produzir o mesmo resultado, independente de um servidor backend, banco de dados, etc. O github permite a hospedagem de sites estáticos gratuitamente, portanto, esse era o caminho a ser seguido!

SSG's são geradores literalmente `Static Sites Generators` e permitiriam que eu alcancasse meu objetivo, procurei por uns projetos open-source e acabei com o hugo, vi alguns temas que achei interessante, o setup parecia relativamente simples, então só fui adiante.

## Hugo
O hugo é um dos SSG's open source mais conhecidos, ele possue temas focados em portfolios, sites empresariais e principalmente blogs. É relativamente fácil de usar e permite a construção de configuração de um site dentro de um tema já existente bem rápido. A maior parte das configurações acontece através do arquivo `config.yaml` ou toml, dependendo do formato escolhido, podemos também criar arquétipos de posts e outros tipos de conteúdos, para facilitar a nossa vida.

A maior parte dos temas do hugo são responsivos, documentados rápidos e bem implementados, estou usando papermod no caso, caso tenha interesse, pode conferir o rodapé da página

# Instalando o tema Papermod para o Hugo
Aqui a tarefa foi simples, conhecendo um pouco como o hugo funciona, segui o passo a passo das docs, peguei a pipeline de actions [desse vídeo](https://www.youtube.com/watch?v=_QSr2_pxIJs&t=1s) e as coisas aconteceram, para o deployment ocorrer, você deve criar uma branch chamada `gh-pages`, para onde será enviado o build da aplicação hugo.

# DNS

Depois disso, apenas configurar o DNS não foi muito difícil, entrei na hostinger e setei os 4 ip's apontados pelo github como a names, além do cname. Obriguei o uso de https e aparentemente está tudo certo :)
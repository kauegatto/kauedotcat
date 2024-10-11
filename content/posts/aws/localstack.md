---
title: "[WIP][AWS] Configurando sua própria AWS com LocalStack"
date: 2024-10-10T16:30:03+00:00
weight: 2
aliases: ["/localstack"]
tags: ["AWS","IAC","TERRAFORM"]
series: ["AWS"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Não é uma surpresa para a maioria que usar a AWS pode ser perigoso no seu bolso, provisionar um recurso e esquecer é um risco, usar uma senha fraca e ser invadido por bots chineses também... Portanto, configure sua própria AWS"
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
# 1. Localstack - Configurando sua própria AWS
A AWS é o maior plataforma de computação em nuvem do mundo. Muito usada, amada e temida.
Não é uma surpresa para a maioria que usar a AWS pode ser perigoso no seu bolso, provisionar um recurso e esquecer é um risco, usar uma senha fraca e ser invadido por bots chineses também...

![Meme - Dont forget your ec2 on or you'll be poor](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kallmdg4wwsoaybm3rsa.png)

Felizmente, existem algumas maneiras de praticarmos AWS, sua *cli*, *cloudformation*, *terraform* e outras ferramentas de infraestrutura sem nos preocupar com custos, *dependendo do que você quiser fazer!*
Neste artigo, exploraremos o LocalStack, uma ferramenta poderosa que nos permite criar uma "AWS local" para testes e desenvolvimento. Também introduziremos o uso do Terraform com o LocalStack para provisionar recursos de forma programática.
## Pré-requisitos

Antes de começarmos, certifique-se de ter instalado:
- Docker
- AWS CLI
- Terraform (opcional, mas recomendado para a segunda parte do tutorial)

> **Dica**: Você pode instalar a AWS CLI usando o gerenciador de versões `asdf`.
## Criando credenciais AWS
Mesmo que estejamos usando uma AWS "falsa", o LocalStack requer credenciais da AWS configuradas. Você pode usar credenciais reais ou fictícias (eu sempre opto pelas fictícias :p).
1. Execute o comando: `aws configure`
2. Preencha os campos de Access Key e Secret Key (podem ser valores fictícios)
3. Especifique uma região AWS válida (ex: us-east-1)
![[{78FB1BEF-487C-4D8C-AEFC-D578124D8DB9}.png]]
## Criando os nossos arquivos serviço
``` bash
mkdir .localstack && touch docker-compose.yml
```
## Criando os Arquivos e Executando o LocalStack
* Você pode baixar o LocalStack e sua CLI se quiser realmente rodar em seu computador, apesar disso, eu costumo optar por usar o docker para rodar ferramentas desse tipo quando possível. Felizmente, aqui temos essa opção:

Dentro do `docker-compose.yml`, preencha:
```yml
version: "3.8"

services:
  localstack:
    container_name: "${LOCALSTACK_DOCKER_NAME:-localstack-main}"
    image: localstack/localstack
    ports:
      - "127.0.0.1:4566:4566" # LocalStack Gateway
      - "127.0.0.1:4510-4559:4510-4559" # external services port range
    environment:
      - DEBUG=${DEBUG:-0}
      - SERVICES=s3
      - PERSISTENCE=/tmp/localstack/data
    volumes:
      - "${LOCALSTACK_VOLUME_DIR:-./volume}:/var/lib/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
```

Inicie o LocalStack:
`docker-compose up -d`

> Localstack costumava ter uma visualização web de fácil acesso, mas eles removeram isso das imagens mais recentes, se quiser ter a visualização via web-app do que está rodando, pode logar no sistema deles: https://app.localstack.cloud/i - Não acho necessário e nesse tutorial usarei a cli da AWS para observar meus recursos.
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hy7eryf810dtqkk5pl5e.png)


# Configurando e usando o LocalStack

**Minha Recomendação**: Crie um `alias` na configuração de seu shell preferido (ex: `.bashrc` ou `.zshrc`):

`alias awslocal='aws --endpoint-url=http://localhost:4566'

Se for de seu interesse, pode usar um [wrapper da localstack](https://github.com/localstack/awscli-local) que faz isso oficialmente, pode ser útil se estiver seguindo esse tutorial no windows (mesmo usando wsl 🤔)


Outra alternativa é sempre que for usar a cli da aws, usar :  `--endpoint-url=http://localhost:4566`
## Testando o LocalStack

Vamos criar um bucket S3 para testar nossa configuração:

`awslocal s3 mb s3://demo-bucket`

Para listar os buckets:

`awslocal s3 ls`

> Lembre-se que `awslocal` é a cli da aws, apnas com o endpoint setado por conveniência, pode usar seus comandos de costume aqui!
# Terraform
O Terraform é uma ferramenta de Infrastructure as Code (IaC) que nos permite definir e provisionar recursos de infraestrutura de forma declarativa.
## Como criar recursos no Terraform
Crie um diretório para os arquivos Terraform:
```bash
mkdir infra && cd infra
touch main.tf
```
A criação de recursos Terraform é bem simples, seguindo o seguinte formato para o módulo da aws:
```json
resource "tipo do recurso" "nome do recurso"{
	#propriedades
}
```
Para sabermos quais os nomes dos recursos padrão da aws, você pode instalar extensões no seu editor de texto / IDE favoritos que te auxiliem, mas siga a documentação oficial para preencher corretamente os recursos:
https://registry.terraform.io/providers/hashicorp/aws/latest/docs

Adicione o seguinte conteúdo ao arquivo `main.tf`:

--- todo: buscar o main.tf correto com as configs do localstack
```json
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region = "us-east-1"
}
```
>ℹ️ Nota:
>Se for executar o Terraform em um ambiente aws real, garanta que seu `aws configure` foi executado com credenciais verdadeiras, e possui chaves de acesso válidas no `~.aws/credentials`
>Para mais informações: https://registry.terraform.io/providers/hashicorp/aws/latest/docs#shared-configuration-and-credentials-files
## Criando seu primeiro recurso
Como anteriormente criamos um bucket para testarmos o localstack, vamos fazer o mesmo, mas com o Terraform:
* Primeiro removerei meu bucket criado via aws cli, apenas por conveniência  `aws s3 rm {nome-bucket}` :

Vamos dar uma olhada na documentação, e copiar o código do recurso básico, apenas mudando o nome!

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/k6k5jwzpzck7t8zbwotg.png)

Cole isso dentro do seu `main.tf` :
```json
resource "aws_s3_bucket" "bucket-terraform" {
  bucket = "bucket-terraform"

  tags = {
    Name        = "bucket"
    Environment = "Dev"
  }
}
```
Rolando para baixo, conseguimos ver todas as configurações possíveis associadas à um bucket s3, mas não precisamos de nada disso, principalmente para fins de teste.
## 4.4 Rodando o Terraform
1. Inicialize o Terraform:
    `terraform init`
2. Verifique o plano de execução:
    `terraform plan`
3. Aplique as mudanças:
    `terraform apply`

> ℹ️ Nota:
> A ênfase desse post é trabalhar com o localstack, conseguir usar a cli e usar Terraform, não considere o código de Terraform aqui o melhor do mundo ~nem do bairro~.
> Recomendo ao menos a leitura desse link: [Terraform Backend State in S3](https://developer.hashicorp.com/terraform/language/settings/backends/s3)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/iz3mfzrx52aqiq581pki.png)


# Observações Finais
- A versão gratuita do LocalStack tem limitações em termos de serviços disponíveis. Serviços como ECS, ECR e EKS não estão disponíveis na versão grátis (e open source).
- Para casos de uso mais avançados ou em produção, considere usar a versão paga do LocalStack ou migrar para uma conta AWS real (considero melhor) com as devidas precauções de segurança e monitoramento de custos - *Sei que é assustador, mas dentro do Terraform, temos acesso à um comando destroy, que deixa as coisas menos piores*.
# Referencias
https://dev.to/goodidea/how-to-fake-aws-locally-with-localstack-27me
[IaC on AWS with Terraform: Provision ECR / ECS Infra to deploy a Node.js App in a Docker Container - YouTube](https://www.youtube.com/watch?v=cgTPxw2oGI8)
https://docs.localstack.cloud/getting-started/quickstart/
https://dev.to/rotirotirafa/como-usar-terraform-localstack-com-docker-h44
https://registry.terraform.io/providers/hashicorp/aws/latest/docs
[Como criar VPC com Subnet Pública e Privada na AWS | Redes para DevOps, Cloud e Devs - YouTube](https://www.youtube.com/watch?v=bd4ribSTs-Y&t=1s)
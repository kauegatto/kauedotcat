---
title: "Encapsulamento: O Básico que todo jr. precisa saber!"
date: 2023-11-12T16:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["SOLID","POO","ARCHITECTURE"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Encapsulamento é um conceito básico da OOP, mas muitas vezes pensado ou ensinado de maneira errada, vamos discutir esse conceito!"
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

# Mudanças em Cascata, Menos pontos de contato

Encapsulamento é um princípio fundamental da programação orientada a objetos (POO) que ajuda a controlar o acesso e a modificação de dados dentro de uma classe. Ele se refere à prática de esconder os detalhes de implementação de uma classe de códigos externos e expor apenas uma interface pública para interagir com a classe. Isso pode ajudar a evitar mudanças em cascata em um sistema de software limitando o número de pontos de contato entre diferentes partes do código.

Uma maneira de alcançar o encapsulamento (pelo menos em uma linguagem OO) é usando modificadores de acesso como "private" ou "protected" em campos e métodos de classe. Isso pode evitar que códigos externos acessem ou modifiquem diretamente o estado interno da classe e forçar o uso de métodos ou propriedades públicas que fornecem uma forma controlada de interagir com a classe.

Essa não é a única maneira e nem todos os atributos de uma classe devem ter cegamente getters e setters, faz sentido uma pessoa depois de sua criação ter seu id alterado? Esse setter tem que existir, realmente? Outra ocasião importante que indica falta de encapsulamento é comentada no ponto 1 do capítulo de [3. Refatoração → Casos Usuais](https://www.notion.so/3-Refatora-o-Casos-Usuais-03076ae1673a41a39e0c8182df7b230c?pvs=21).

```java
class MyClass {
    private int data;

    public int getData() {
        return data;
    }

    public void setData(int newData) {
        data = newData;
    }
}
```

O encapsulamento também pode ser usado para evitar mudanças em cascata mantendo o número de pontos de contato entre diferentes partes do código o mínimo possível. Ou seja, ao invés de termos em 5 lugares diferentes o código `Float.parseFloat(getData())` poderíamos encapsular isso dentro de um método na classe Data se isso for uma regra de Data ou um comportamento relacionado à ela (mesmo que não seja, provavelmente é um comportamento de alguma outra classe). Depois dessa refatoração, ao mexermos nesse método, cuidaremos dos 5 pontos que o utilizam, ao invés de ficar buscando por ai a linha de Float.parseFloat(getData()) e quebrarmos o código pois não encontramos uma ocorrência.

## Devo abstrair um determinado parâmetro/retorno?

Já dissemos muito sobre abstrações e encapsulamento, mas quando você deve abstrair um certo parâmetro ou retorno? Aquele atributo String CPF na sua classe é realmente prejudicial? Se sim, vale a pena criar uma classe para defini-lo?

Suponha que você precise criar um objeto Usuário, que tem como parâmetro do construtor um nome e senha.

```java
class Usuario{
	...
	public Usuario (String nome, String senha){}
}
Usuario user1 = new Usuario("kaue", "123456");
```

Nesse caso, como você pode garantir que usuário deve receber sua senha como plain text, e não depois de passar por um hash? Ou que a senha deve ter ao menos 6 caracteres, isso foge da lógica e semântica estabelecida pela tipo String, apesar disso, não é uma validação que dá muito trabalho, um simples length já resolveria o problema do tamanho, nesse caso, não acho que valeria criar uma classe somente para isso, mas quanto ao problema do hash, como validaríamos que o cliente usou corretamente o construtor e passou uma senha como plain text?

É simples, somente ver a implementação da classe Usuário, mas se isso não está claro, temos ai um *code smell,* uma espécie de acoplamento mental. Se você não tivesse acesso à Usuário, não teria como adivinhar.  Nesse caso em específico, mudar o nome da variável para plainText seria uma solução caso. 

Contudo, se a solução não for tão simples, acredito que normalmente passa a valer a pena criar uma classe de domínio para o parâmetro (ou retorno). Uma classe Senha com um construtor privado e métodos factory seria uma alternativa.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/01652d07-158d-4164-9834-1fd6c1bbe164/Untitled.png)

Trazendo isso para um exemplo real, durante o handling de metadatas as guardávamos como `Map<String,Object>` , repetidos em diversos lugares, apesar disso, usávamos o Map em seu exato contexto, sem a necessidade de métodos a mais além dos próprios do Map e o seu contexto e significado era exatamente o que um map representava, nesse caso, não sentimos necessidade de abstrair o tipo.

# Protegendo Fronteiras

As fronteiras são a parte do seu software que agem como portas ao mundo externo. No caso de uma API, os seus clientes e pessoas que chamam a API também são externos, nenhum dado externo deve ser confiado.

Proteger as fronteiras de um software refere-se a garantir que as interfaces externas e entradas do software sejam validadas e sanitizadas adequadamente para evitar ataques maliciosos ou entradas inesperadas. Isso é uma parte importante da segurança de software e pode ajudar a prevenir problemas como SQL Injection, cross-site scripting (XSS) e outros tipos de ataques de injeção.

Uma das principais partes de proteger as fronteiras de software é a validação de entrada (input validation). Isso envolve verificar todos os dados de entrada para garantir que eles atendam a certos critérios antes de serem processados. Por exemplo, uma entrada de formulário pode ser necessária para ter um certo comprimento ou estar em um formato específico. Isso pode ajudar a prevenir erros e comportamentos inesperados, além de proteger contra entradas maliciosas.

Outra parte importante de proteger as fronteiras de software é a sanitização. Isso envolve remover ou modificar qualquer elemento potencialmente prejudicial de dados de entrada. Por exemplo, uma aplicação web pode sanitizar a entrada de usuários para remover qualquer código JavaScript ou HTML que possa ser usado para realizar um ataque cibernético.

O DDD (Domain Driven Design) também desempenha um papel importante na proteção das fronteiras de software. O DDD é uma abordagem de design de software que se concentra no domínio empresarial e enfatiza a importância de criar uma separação clara entre a lógica do domínio e os detalhes técnicos da implementação. Isso pode ajudar a garantir que as interfaces externas do software sejam bem definidas e fáceis de entender, o que pode tornar mais fácil validar e sanitizar entradas.

### Serializar classes de domínio → não fazer

Ao expor os objetos do domínio através de uma API, serializá-los e enviá-los diretamente para o cliente como uma resposta, pode levar a uma série de problemas:

- O cliente pode potencialmente modificar o estado do objeto do domínio e violar as regras do domínio.
- O objeto do domínio pode conter informações sensíveis que não devem ser expostas ao cliente.
- O objeto do domínio pode conter informações que não são relevantes para o cliente e podem levar à coleta excessiva de dados.

Para evitar esses problemas, é recomendado criar um DTO (Objeto de Transferência de Dados) separado que é especificamente projetado para a API, ele deve conter apenas as informações que o cliente precisa e não deve conter nenhum comportamento ou métodos que possam mudar o estado do objeto do domínio. Isso ajudará a proteger a integridade do modelo de domínio e manter o cliente desacoplado dos detalhes de implementação interna dos objetos do domínio.

---

Princípio: **Favorecemos coesão através do encapsulamento**

## O Básico

O Encapsulamento é basicamente o ato de juntar comportamentos e estados que fazem sentido no mesmo lugar, garantindo maior coesão ao código, é um conceito básico que precisa ser dominado. Veja esse código em um repositório de um framework da Apache

```java
for (Address address : vcard.getAddresses()) {
                    boolean workAddress = false;
                    for (AddressType addressType : address.getTypes()) {
                        if (AddressType.PREF.equals(addressType) || AddressType.WORK.equals(addressType)) {
                            workAddress = true;
                            break;
                        }
                    }
                    if (!workAddress) continue;
```

Sem entender muito do código e de seu contexto, já somos capaz de refatorar isso de uma maneira melhor, poderíamos simplesmente usar:

```java
if(!adress.hasWorkAdress()) continue;
```

Com isso, o código escrito na classe ficaria mais legível e a função de descobrir se há endereço de trabalho ou não, passa a ser da classe `Adress` e pode ser replicado sem problemas através de toda a aplicação. Se decidirmos mudar a regra de negócio no estado atual, teríamos que verificar por esse imenso código esparramado por todo o programa, o que não acontece no código refatorado.

Apenas com essa alteração:

- O service fica mais legível
- O service fica mais testável
- O código é reaproveitado e muda junto com apenas um ponto de contato (dentro da classe Adres)
- O service tem maior complexidade
    - A complexidade não surge, ela é distribuída, a regra já existe.

Como podemos detectar isso? Um forte indicativo que algo está estranho é estarmos usando um estado interno e aplicando lógica em cima desse estado interno fora de sua classe.

Métodos privados também podem indicar esse tipo de comportamento, talvez até mesmo a necessidade do nascimento de novas entidades.

```java
private Person createUserAccount(String username, Collection<GrantedAuthority> authorities,PersonAttributesLookup personAttributesLookup) {
		Person person = null;

		if (hasAccountCreationPermission(authorities)) {
			person = new Person();
			person.setEnabled(true);
			person.setUsername(username);

			try {
				// Get the Person Attributes to create the person
				final PersonAttributesResult attr =
						personAttributesLookup.lookupPersonAttributes(username);
				person.setSchoolId(attr.getSchoolId());
				person.setFirstName(attr.getFirstName());
				person.setLastName(attr.getLastName());
				person.setPrimaryEmailAddress(attr.getPrimaryEmailAddress());

				ensureRequiredFieldsForDirectoryPerson(person);
				person = create(person);
				externalPersonService.updatePersonFromExternalPerson(person, false);
				LOGGER.info("Successfully Created Account for {}", username);

			} catch (final ObjectNotFoundException onfe) {
				...
			}
```

Refatorando:

```java
				person.setSchoolId(attr.getSchoolId());
				person.setFirstName(attr.getFirstName());
				person.setLastName(attr.getLastName());
				person.setPrimaryEmailAddress(attr.getPrimaryEmailAddress());
// passa a se tornar
public Class Person{
	...
	Person setAttributesBasedOnAttributesResult(PersonAttributesResult attr){
				this.person.setSchoolId(attr.getSchoolId());
				this.person.setFirstName(attr.getFirstName());
				this.person.setLastName(attr.getLastName());
				this.person.setPrimaryEmailAddress(attr.getPrimaryEmailAddress());

	}
}
// seria usado:
person.setAttributesBasedOnAttributesResult(attr);
```

Somente nessa alteração, já travamos setters que podem não ser interessantes para o negócio (como alterar o schoolId sem alterar o emailAdress, se isso for uma regra existente), ou seja, o código antigo estava desprotegido e acoplado mentalmente à alguma regra externa, precisamos sempre que atualizarmos o primeiro nome atualizar também o segundo? não sei. Se sim, podemos fazer com que o encapsulamento garanta isso.

Outro ponto de refatoramento em Person é essa parte:

```java
		person = new Person();
		person.setEnabled(true);
		person.setUsername(username);
```

Talvez os métodos de setEnabled e setUsername sejam necessários, ou seja, não faz sentido criarmos um usuário sem Username e desabilitado, por que isso não faz parte do construtor? Esses setters provavelmente nem deveriam existir, com os atributos setados no próprio construtor. Tratamos isso no capítulo de refatoração 

O código final poderia ficar:

```java
private Person createUserAccount(String username, Collection<GrantedAuthority> authorities,PersonAttributesLookup personAttributesLookup) {
		Person person = null;
		if (hasAccountCreationPermission(authorities)) {
			person = new Person();
			try {
				// Get the Person Attributes to create the person
				final PersonAttributesResult attr =
						personAttributesLookup.lookupPersonAttributes(username);
				person.setAttributesBasedOnAttributesResult(attr);

				ensureRequiredFieldsForDirectoryPerson(person);
				person = create(person);
				externalPersonService.updatePersonFromExternalPerson(person, false);
				LOGGER.info("Successfully Created Account for {}", username);

			} catch (final ObjectNotFoundException onfe) {
				...
			}     
```

## Disclaimer!
Não leve tudo que leu aqui como verdades absolutas ou regras a serem seguida em 100% dos casos, não existe martelo de ouro nem bala de prata. Se houver algum questionamento ou sugestão, pode deixar aqui!

# Referências:

[Encapsulamento para ganhar mais coesão: O feijão com arroz que precisa estar dominado](https://www.youtube.com/watch?v=2Bso1Yg3xUA)

OOP E Solid para ninjas - Casa do código - Aniche 

Desbravando SOLID - Casa do Código - Aquiles
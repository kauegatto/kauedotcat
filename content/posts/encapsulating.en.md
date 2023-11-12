---
title: "Encapsulation: The Basics Every Junior Developer Needs to Know!"
date: 2023-11-12T16:30:03+00:00
tags: ["SOLID", "OOP", "ARCHITECTURE"]
description: "Encapsulation is a fundamental concept in OOP, often misunderstood or incorrectly taught. Let's discuss this concept!"
canonicalURL: "https://canonical.url/to/page"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: true
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
    Text: "Propose Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

# Cascading Changes, Fewer Points of Contact
Encapsulation is a fundamental principle of object-oriented programming (OOP) that helps control access and modification of data within a class. It involves hiding the implementation details of a class from external code and exposing only a public interface to interact with the class. This can help avoid cascading changes in a software system by limiting the number of points of contact between different parts of the code.

One way to achieve encapsulation (at least in an OO language) is by using access modifiers like "private" or "protected" on class fields and methods. This can prevent external code from directly accessing or modifying the internal state of the class and enforce the use of public methods or properties that provide a controlled way to interact with the class.

However, blindly adding getters and setters for all class attributes is not the only way to achieve encapsulation. Does it make sense for a person's ID to be changed after creation? Does that setter need to exist? Another important occasion indicating a lack of encapsulation is discussed in point 1 of the chapter on 3. Refactoring → Common Cases.

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
Encapsulation can also be used to avoid cascading changes by keeping the number of points of contact between different parts of the code to a minimum. Instead of having the code Float.parseFloat(getData()) in five different places, we could encapsulate this inside a method in the Data class if it's a rule of Data or a behavior related to it (even if it's not, it's probably a behavior of some other class). After this refactoring, when we modify this method, we only need to take care of the five points that use it, instead of searching for the line Float.parseFloat(getData()) throughout the code and risking breaking it because we couldn't find an occurrence.

# Should I Abstract a Certain Parameter/Return?
We've talked a lot about abstractions and encapsulation, but when should you abstract a certain parameter or return? Is that String attribute CPF in your class really harmful? If so, is it worth creating a class to define it?

Suppose you need to create a User object, which has a name and password as constructor parameters.

``` java
class User {
	...
	public User(String name, String password){}
}
User user1 = new User("kaue", "123456");
```
In this case, how can you ensure that the user should receive their password as plain text, not after being hashed? Or that the password must be at least 6 characters long? Despite this, it's not a validation that is very difficult; a simple length check would solve the size problem. In this case, I don't think it would be worth creating a class just for this. But regarding the hash problem, how would you validate that the client correctly used the constructor and passed a plain text password?

It's simple, just look at the implementation of the User class. But if this is not clear, there's a code smell, a kind of mental coupling. If you didn't have access to User, you wouldn't be able to guess. In this specific case, changing the variable name to plainText would be a solution.

However, if the solution is not so simple, I believe it usually becomes worthwhile to create a domain class for the parameter (or return). A Password class with a private constructor and factory methods would be an alternative.

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
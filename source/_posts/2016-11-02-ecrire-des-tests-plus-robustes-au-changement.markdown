---
layout: post
title: "Ecrire des tests plus robustes au changement"
date: 2016-11-02 17:31:42 +0100
comments: true
categories: [TDD, Tests, Craftsmanship]
---

Un des principaux freins à l'écriture de tests est quand ceux-ci doivent être mis à jour à chaque fois qu'une classe ou qu'une méthode est modifiée. Parfois, les tests à mettre à jour n'ont rien à voir avec la classe ou la méthode modifiée, mais comme ils s'en servent, ils s'en retrouvent affectés eux aussi. Ce problème survient quand le test est trop couplé à l'implémentation.
Dans cet article, nous allons voir comment découpler les tests des classes qu'ils utilisent et ainsi les rendre plus robustes.

<!-- more -->

## Méthode 0 : La méthode naïve

Soit une classe qui doit tester si une personne a accès à une ressource ou non. L'API de la classe `Person` est maintenue par une autre équipe et est donc hors de contrôle.
La façon la plus naïve d'écrire le test pourrait être la suivante :

```java
@Test
public void should_deny_access_when_underaged() {
  Person kid = new Person();
  kid.setAge(17);

  AuthorizationPolicy policy = new AuthorizationPolicy();

  Authorization authorization = policy.authorize(kid);

  assertThat(authorization, is(Authorization.DENY));
}
```

Si l'API de la classe `Person` évolue et prend l'âge directement en paramètre du constructeur, ce test ne compilera plus et devra être réécrit, ainsi que tous les autres tests qui instancient directement une `Person`.

## Méthode 1 : Utilisation d'une méthode de création

La première solution serait d'utiliser une méthode de création partagée par tous les tests de la classe :

```java
@Test
public void should_deny_access_when_underaged() {
  Person kid = newPerson(17);
  AuthorizationPolicy policy = new AuthorizationPolicy();

  Authorization authorization = policy.authorize(kid);

  assertThat(authorization, is(Authorization.DENY));
}

private Person newPerson(int age) {
  Person person = new Person();
  person.setAge(17);
  return person;
}
```

Ici, si l'API de `Person` change, seule la méthode est affectée. Tous les tests passeront sans difficulté une fois cette méthode modifiée.

Maintenant, la politique d'autorisation évolue pour refuser les mineurs, mais également les garçons (même majeurs). Le test devient alors :

```java
@Test
public void should_deny_access_when_underaged() {
  Person kid = newPerson(17, Gender.FEMALE);
  AuthorizationPolicy policy = new AuthorizationPolicy();

  Authorization authorization = policy.authorize(kid);

  assertThat(authorization, is(Authorization.DENY));
}

@Test
public void should_deny_access_when_male_and_overaged() {
  Person maleAdult = newPerson(18, Gender.MALE);
  AuthorizationPolicy policy = new AuthorizationPolicy();

  Authorization authorization = policy.authorize(maleAdult);

  assertThat(authorization, is(Authorization.DENY));
}

private Person newPerson(int age, Gender gender) {
  Person person = new Person();
  person.setAge(age);
  person.setGender(gender);
  return person;
}
```

On remarque que le premier test a été modifié, bien que le sexe de la personne ne soit pas pertinent dans ce cas. Le code du premier test est donc moins expressif sur la règle métier testée. Si on augmente le nombre de paramètres dans la méthode `newPerson` l'expressivité et la robustesse des tests décroit de plus en plus. De plus, la méthode `newPerson` n'est pas directement réutilisable dans les autres classes de test.

## Méthode 2 : Utilisation d'un builder

Cette méthode consiste à utiliser un `Builder` pour permettre de construire une instance de `Person` personnalisable à la demande.

Il suffit de créer une classe `PersonBuilder` :

```java

public class PersonBuilder {

  private int age;
  private Gender gender = Gender.FEMALE;

  public static PersonBuilder aPerson() {
    return new PersonBuilder();
  }

  public PersonBuilder withAge(int age) {
    this.age = age;
    return this;
  }

  public PersonBuilder withGender(Gender gender) {
    this.gender = gender;
    return this;
  }

  public Person build() {
    Person person = new Person();
    person.setAge(age);
    person.setGender(gender);
    return person;
  }
}
```

Les tests deviennent alors :

```java
@Test
public void should_deny_access_when_underaged() {
  Person kid = aPerson().withAge(17).build();
  AuthorizationPolicy policy = new AuthorizationPolicy();

  Authorization authorization = policy.authorize(kid);

  assertThat(authorization, is(Authorization.DENY));
}

@Test
public void should_deny_access_when_male_and_overaged() {
  Person maleAdult = aPerson().withAge(18).withGender(Gender.MALE).build();
  AuthorizationPolicy policy = new AuthorizationPolicy();

  Authorization authorization = policy.authorize(maleAdult);

  assertThat(authorization, is(Authorization.DENY));
}
```

Désormais, les tests sont bien plus clairs sur leur intention car seuls les attributs nécessaires sont spécifiés. Les autres peuvent être `null` ou avoir des valeurs par défaut (ici, le sexe est féminin par défaut par exemple). De plus, la logique d'instanciation d'une `Person` est située à un seul endroit dans les tests et seul le builder serait affecté si l'API venait à changer.

## Conclusion

Les méthodes décrites dans cet article permettent de découpler les tests des logiques d'instanciation des objets nécessaires. Elles sont particulièrement utiles lors de l'utilisation d'API tierces qui ne sont pas toujours très stables ni très bien conçues. Grâce à elles, les tests sont plus expressifs, plus robustes et plus concis.

---
layout: post
title: "Quoi de neuf dans Spring 5"
date: 2016-07-30 14:05:38 +0200
comments: true
categories: [Spring, Reactive, Java]
---

Juergen Hoeller a annoncé hier [la sortie de Spring 5.0 M1](https://spring.io/blog/2016/07/28/spring-framework-5-0-m1-released). Cette nouvelle version majeure de Spring arrive avec son lot de nouveautés.
L'objectif de cet article est de découvrir ces nouveautés et ce qu'elles peuvent apporter aux développeurs Spring. Les nouveautés décrites dans cet article sont déjà disponibles dans la version 5.0.M1.

<!-- more -->

## Modification des prérequis

Spring 5 étant une version majeure, l'équipe s'est permise de modifier les pré-requis et d'abandonner le support pour des librairies obsolètes. A noter que la version 4.3 est supportée jusqu'en 2020 environ pour ceux qui ne pourraient pas mettre à jour leur application.

### Java 8 et Java 9

Le changement principal au niveau des prérequis est la réécriture partielle de Spring en Java 8. De fait, il n'est pas possible d'utiliser Spring 5 avec une version antérieure de Java. Ce changement a ravi les développeurs Spring, comme on peut s'en douter (Spring 4.3 était en effet développé pour être compatible avec Java 6 !).

<blockquote class="twitter-tweet" data-partner="tweetdeck"><p lang="en" dir="ltr">Current status: working in the <a href="https://twitter.com/springframework">@springframework</a> codebase with Java8 - I so much feel like a kid in a candy store right now!</p>&mdash; Stéphane Nicoll (@snicoll) <a href="https://twitter.com/snicoll/status/750332049492500480">July 5, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Le passage à Java 8 a permis d'ajouter des méthodes par défaut à certaines interfaces centrales de Spring. Par exemple, l'interface `BeanPostProcessor`, qui permet de pouvoir modifier un bean après son instanciation, voit ses deux méthodes retourner le bean initial par défaut. Il est ainsi possible d'implémenter seulement `postProcessBeforeInitialization`, qui modifie le bean avant l'appel de la méthode `@PostConstruct`, ou bien seulement `postProcessAfterInitialization`, qui modifie le bean après l’appel de la méthode `@PostConstruct`.

```java
public interface BeanPostProcessor {
  default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    return bean;
  }
  default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    return bean;
  }
}
```

De plus, Java 8 a aussi introduit les lambdas, qui sont désormais utilisées dans les API Spring, comme par exemple le message d'erreur des méthodes de la classe `org.springframework.util.Assert` qui peut être évalué de manière _lazy_ grâce à un `Supplier`.

```java
Assert.isTrue(contextPath.startsWith("/"), () -> "contextPath '" + contextPath + "' must start with '/'.");
```

Pour le moment, Java 8 est utilisé avec parcimonie mais nul doute que son usage va se généraliser dans les mois à venir pour améliorer l'expérience développeur des utilisateurs de Spring.

#### Java 7

Comme dit ci-dessus, Spring était compatible avec Java 6 jusqu'à la version 4, le passage à Java 8 a donc également permis de mettre à jour la base de code pour utiliser les fonctionnalités Java 7 (qui sont principalement des sucres syntaxiques) comme par exemple l'opérateur diamant, le _multicatch_, le _try-with-resources_ ou l'utilisation de `StandardCharsets` au lieu de `String` pour définir les encodages.

#### Java 9

Finalement, Spring 5 continue de suivre les avancées de Java 9 pour rester le plus à jour possible dans son support. Par exemple, en Java 9, la méthode `Class.newInstance()` est dépréciée au profit de `Constructor.newInstance()` qui retourne une exception `InvocationTargetException` au lieu de propager l'exception utilisateur. Les classes Spring qui doivent instancier des beans arbitraires devront donc utiliser la nouvelle façon de faire.

### JPA 2.1 et Hibernate 5

JPA 2.1 a presque 4 ans. Spring 5 essaie d'éviter les compromis avec la compatibilité et a donc établi JPA 2.1 comme prérequis. De fait, Hibernate 5 est obligatoire.

### Abandon du support

Dans la même logique d'alléger le support pour des librairies obsolètes ou pas à jour, Spring 5 abandonne le support pour de nombreuses librairies dont PortletMVC, JDO, Guava caching, JasperReports, OpenJPA, Tiles 2, XMLBeans, Velocity.

## Programmation réactive

La programmation réactive a le vent en poupe en ce moment et Spring 5 n'échappe pas à la règle. Le projet Spring Reactive a été mergé dans Spring Framework et permet désormais d'utiliser ses capacités dans Spring. De fait, Spring 5 permet désormais de sérialiser et désérialiser en XML (JAXB) et en Json (Jackson) de manière réactive et fournit un framework web et un `WebClient` réactifs.

Le framework Spring Web Reactive et le bien connu Spring Web MVC ne partagent pas de code car le paradigme de programmation est fondamentalement différent. Cependant, l'API est très proche, pour une plus grande cohérence et une prise en main plus rapide pour les utilisateurs de ce framework.

Voici par exemple à quoi ressemble un controller qui stream des données depuis un serveur de manière non bloquante et réactive.

```java
@GetMapping("/accounts/{id}/alerts")
public Flux<Alert> getAccountAlerts(@PathVariable Long id) {

  return this.repository.getAccount(id)
      .flatMap(account ->
          this.webClient
              .perform(get("/alerts/{key}", account.getKey()))
              .extract(bodyStream(Alert.class)));
}
```

Pour en savoir plus sur la programmation réactive avec Spring, se référer aux billets de blog [Reactive Programming with Spring 5.0 M1](https://spring.io/blog/2016/07/28/reactive-programming-with-spring-5-0-m1) et [Understanding Reactive types](https://spring.io/blog/2016/04/19/understanding-reactive-types).

Pour information, un starter _Reactive Web_ a été ajouté au [starter Spring Boot](http://start.spring.io) en version 1.4+ de Spring Boot. Plus d'informations [ici](https://github.com/bclozel/spring-boot-web-reactive).

## Tests

Finalement, Spring 5 a ajouté le support de JUnit 5, notamment grâce à l'ajout d'une [extension](http://junit.org/junit5/docs/current/user-guide/#extensions) Spring, à utiliser avec `@ExtendWith(SpringExtension.class)` ou alors `@SpringJUnitConfig` qui combine `@ExtendWith(SpringExtension.class)` et `@ContextConfiguration`.

## Conclusion

Spring 5 est une version majeure de Spring et l'équipe de développement en a profité pour faire un grand ménage dans les dépendances et les librairies supportées. Cette version permet de profiter pleinement des nouvelles fonctionnalités de Java sans la frustration parfois apportée par la rétro-compatibilité. De plus, la fusion de Spring Reactive dans le framework offre aux développeurs un outil extrêmement puissant pour tirer bénéfice de la programmation réactive avec Spring.

Pour information, la version RELEASE est prévue pour mars 2017.
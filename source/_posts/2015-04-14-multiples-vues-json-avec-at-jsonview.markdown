---
layout: post
title: "Multiples vues JSON avec @JsonView"
date: 2015-04-14 21:05:31 +0100
comments: true
categories: [Spring, Java, Json]
---

Le format [Json](http://fr.wikipedia.org/wiki/JavaScript_Object_Notation) est de plus en plus utilisé aujourd'hui notamment dans les API de type REST qui offrent des services et retournent leurs résultats sous ce format. Afin d'éviter d'avoir à écrire la transformation en Json manuellement, il existe de nombreuses librairies permettant de faire cette transformation de manière automatique.

Dans cet article, nous allons voir comment utiliser la librairie [Jackson](http://jackson.codehaus.org/) et notamment l'annotation `@JsonView` pour configurer plusieurs transformations pour un même objet. Nous allons également voir succintement comment intégrer cette annotation avec le framework [Spring MVC](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html).

<!-- more -->

## Convertir un objet en Json avec Jackson

Le code suivant permet d'utiliser Jackson pour convertir un objet Java en Json de manière automatique.

```java
public String toJson(Foo foo) {
	ObjectMapper mapper = new ObjectMapper();
	return mapper.writeValueAsString(foo);
}
```


## Introduction à @JsonView

Lorsque Jackson convertit l'objet en Json, il prend en compte tous les champs de cet objet. Or, dans la vie courante, il est rarement pertinent de récupérer tous les champs dans le Json final (par exemple, le mot de passe d'un utilisateur) et il est souvent intéressant d'avoir deux sorties différentes en fonction du contexte (par exemple, un Json résumé et un détaillé avec plus de champs).

L'annotation `@JsonView` permet de définir des vues et associer chaque champ à une ou plusieurs vues. Ainsi, au moment de la conversion, on indique à Jackson quelle vue utiliser et seuls les champs associés à cette vue seront pris en compte.

### Mise en place des vues

La vue Json doit être une classe, ainsi elle peut bénéficier de l'héritage. Si la vue A hérite de la vue B, tous les champs associés à la vue B seront pris en compte lorsqu'on va exporter la vue A.

Les vues peuvent être définies de la façon suivante.

```java
public class JsonViews {

	public static class PublicView { };
	public static class InternalOnlyView extends PublicView { };

}
```

Il n'y a pas besoin de mettre quoi que ce soit dans les classes, une simple déclaration suffit. C'est pourquoi il est plus simple de les regrouper dans une classe. On remarque ici l'héritage qui permet d'indiquer que tout ce qui est public sera visible également dans la vue interne.

### Association d'une vue à un champ

Pour associer une vue à un champ, il faut l'annoter avec `@JsonView` avec la vue en paramètre.

```java
public class Foo {

	@JsonView(JsonViews.PublicView.class)
	private String name;
    
	@JsonView(JsonViews.InternalOnlyView.class)
	private String password;
}
```

Ensuite, pour indiquer à Jackson d'exporter seulement cette vue, il faut lui passer en paramètre au moment de la conversion.

```java
public String toJsonPublic(Foo foo) {
	ObjectMapper mapper = new ObjectMapper();
	return mapper
 		.writerWithView(JsonViews.PublicView.class)
		.writeValueAsString(foo);
}
```


_NB : Il est possible de définir plusieurs vues pour un champ en les mettant entre crochets `@JsonView({View1.class, View2.class})`. Ainsi, ce champ sera pris en compte dans chacune des vues_


## Intégration avec Spring MVC

Dans un controller Spring MVC, il n'est pas utile d'utiliser le code de conversion écrit précédemment. En effet, Jackson est intégré avec Spring MVC. Il suffit d'annoter la méthode du controller avec `@JsonView` pour lui indiquer la vue à utiliser.

```java
@RestController
public class FooController {

	@JsonView(JsonViews.PublicView.class)
	@RequestMapping (value = "/public/foo", method = RequestMethod.GET)
 	public Foo getFoo() {
 		return new Foo("name", "password");
 	}
}
```

Un appel à `/public/foo` va automatiquement retourner un objet Json qui représente la vue publique de l'objet instancié dans la méthode.


## Conclusion

`@JsonView` est un outil très simple et très puissant qui permet d'avoir plusieurs représentations du même objet sans avoir à écrire de code autre que quelques annotations. De plus, son intégration avec Spring MVC le rend très peu intrusif et ne pollue pas le code avec des opérations de conversion.
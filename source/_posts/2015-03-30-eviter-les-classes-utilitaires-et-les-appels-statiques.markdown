---
layout: post
title: "Eviter les classes utilitaires et les appels statiques"
date: 2015-03-30 21:14:16 +0100
comments: true
categories: [Craftsmanship, Java]
---

De manière générale, les appels statiques sont des mauvaises pratiques car ils induisent notamment un fort couplage entre les classes et rendent les méthodes appelantes difficiles à tester. Cependant, il existe un cas d'utilisation qui utilise abondamment des appels statiques : il s'agit des classes utilitaires.

Cet article a pour objectif de voir comment et quand se passer des appels statiques aux classes utilitaires.

<!-- more -->

## De l'utilisation des classes utilitaires

Les classes utilitaires sont utilisées absolument partout dans les programmes Java. Que ce soit les classes du JDK telles que `Math` ou `Arrays` ou bien celles de librairies externes comme `CollectionUtils` ou `FileUtils`, vous en avez forcément déjà utilisé.
Comme leur nom l'indique, ces classes sont très utiles pour effectuer des opérations courantes et sans état. 

## Cas d'utilisation

Pour illustrer mes propos, je vais prendre un cas d'utilisation assez classique : calculer le hash d'un mot de passe avant insertion en base de données. Ce cas d'utilisation semble être éligible à l'utilisation d'une classe utilitaire.

### Utilisation d'une méthode statique

#### Implémentation

En suivant la logique des classes utilitaires, voici une implémentation possible de ce comportement.

Nous allons créer une classe utilitaire nommée `SecurityUtils` qui dispose d'une méthode statique `sha256` qui hash un texte en clair.

```java
public class SecurityUtils {

   public static String sha256(String plainText) {
     try {

        MessageDigest md;
        md = MessageDigest.getInstance("SHA-256");
        md.update(plainText.getBytes("iso-8859-1"), 0, plainText.length());
        byte[] shaHash = md.digest();
        return toHexadecimal(shaHash); 

     } catch (UnsupportedEncodingException | NoSuchAlgorithmException e) {
        logger.error(e);
     } 
   }

}
```

Ensuite, la classe du service qui va appeler cette méthode

```java
public class UserService {

   public User save(User user, String plainPassword) {
    
      user.setPassword(SecurityUtils.sha256(plainPassword));        
      repository.save(user);
      
      return user;
   }

}
```

Ce code semble correct et est très courant dans les programmes Java. Cependant, il me pose un problème conséquent. En effet, comment tester la méthode `save` de manière unitaire ?

#### Tests

Tester unitairement une méthode qui effectue des appels statiques est compliqué. Un test unitaire compliqué est souvent un symptôme d'un code trop couplé et mal conçu. 


Voici une implémentation naïve du test unitaire de la méthode `save`.

```java
public class UserServiceTest {

   @Test
   public void should_hash_password_when_save() {
     
      User user = new UserService().save(new User(), "pass_word");
      
      assertThat(user.getPassword).isEqualTo(SecurityUtils.sha256("pass_word"));
   }
}
```

Ce test a le mérite de ne pas dépendre de l'implémentation de `sha256`. Cependant, si celle-ci lance une exception, ou bien retourne toujours une chaine vide par exemple, le test peut générer des faux négatifs ou des faux positifs sans que l'on sache si cela provient du service ou de la classe utilitaire.


Il est bien évidemment possible de mocker des méthodes statiques avec des librairies comme [PowerMock](https://code.google.com/p/powermock/) mais je préfère éviter d'utiliser ce genre de raccourci. C'est pourquoi je préfère me passer de méthodes statiques et utiliser des méthodes d'instances à la place.

### Utilisation d'une méthode d'instance

#### Implémentation
Pour le même cas d'utilisation, voici l'implémentation que j'utiliserais.

Tout d'abord, je définis une interface qui sera le contrat de ma classe utilitaire.

```java
public interface SecurityProvider {

   public String sha256(String plainText);

}
```

Ensuite, il faut implémenter cette interface.

```java
public class SecurityProviderImpl implements SecurityProvider {

   public String sha256(String plainText) {
     try {

        MessageDigest md;
        md = MessageDigest.getInstance("SHA-256");
        md.update(plainText.getBytes("iso-8859-1"), 0, plainText.length());
        byte[] shaHash = md.digest();
        return toHexadecimal(shaHash); 

     } catch (UnsupportedEncodingException | NoSuchAlgorithmException e) {
        logger.error(e);
     } 
   }

}
```

Finalement, pour utiliser cette classe, il faut l'injecter dans le service. L'injection peut se faire à la main, via Spring ou JEE mais cela pourrait être l'objet d'un autre article donc je ne rentrerai pas dans les détails.


```java
public class UserService {

   private final SecurityProvider securityProvider;
   
   public UserService(SecurityProvider securityProvider) {
      this.securityProvider = securityProvider;
   }

   public User save(User user, String plainPassword) {
    
      user.setPassword(securityProvider.sha256(plainPassword));        
      repository.save(user);
      
      return user;
   }

}
```

#### Tests

Cette implémentation est beaucoup plus simple à tester unitairement. En effet, il suffit de mocker l'interface pour vérifier le bon comportement du service. Ce mock peut être fait manuellement ou à l'aide d'un framework comme [Mockito](http://mockito.org/).

```java
public class UserServiceTest {

   @Test
   public void should_hash_password_when_save() {
   
      SecurityProvider securityProvider = new SecurityProvider() {
      
         @Override
         public String sha256(String plainText) {
            return "Hashed :" + plainText;
         }
      
      }
     
      User user = new UserService(securityProvider).save(new User(), "pass_word");
      
      assertThat(user.getPassword).isEqualTo("Hashed :pass_word");
   }
}
```

Ce test est à 100% découplé de l'implémentation réelle de la méthode `sha256` et permet de tester de manière réellement unitaire la méthode `save` du service.

## Bonnes pratiques et pragmatisme

L'utilisation des classes utilitaires et des méthodes statiques est globalement à proscrire car cela rend le code fortement couplé et difficile à tester. Cependant, un développeur se doit d'être pragmatique. 
En effet, créer une interface, une implémentation et une injection pour chaque appel à une méthode utilitaire peut sembler un peu trop long et verbeux. 

Je pense qu'il s'agit de trouver le bon compromis entre une utilisation sage des méthodes statiques et les bonnes pratiques de programmation. Par exemple, il n'est pas utile de mettre en place ce pattern pour des calculs simples comme une valeur absolue, un test sur la présence ou non d'élément dans une liste ou le calcul d'un minimum entre deux nombres. Par contre, quand il s'agit de traitements plus lourds comme des calculs complexes ou la manipulation d'un fichier, ce travail supplémentaire peut porter ses fruits et doit être mis en place.
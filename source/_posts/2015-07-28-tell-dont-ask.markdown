---
layout: post
title: "Tell don't ask"
date: 2015-07-28 20:57:56 +0100
comments: true
categories: [OOP, Craftsmanship]
---

Quand j'étais à l'école et que j'apprenais la programmation orientée objet (POO), on nous disait que les classes permettaient d'encapsuler des comportements et de cacher leur implémentation au reste du monde. Ainsi, le code qui appelle une méthode attend d'elle qu'elle se comporte comme elle a été spécifiée, quelle que soit son implémentation.

<!-- more -->

Prenons l'exemple d'un panier, dont le coût total doit être calculé selon certaines règles (somme de tous les articles, ajout éventuel de taxes, charges, etc.). En POO, on pourrait s'attendre à ce que la classe `Basket` ait une méthode `calculateTotalCost` qui va calculer ce coût. Après tout, le code appelant ne veut pas savoir comment calculer un coût, il veut juste le récupérer.

Pourtant, lors de mes premiers mois dans le monde du travail, je suis tombé sur ce genre ce code, au détour d'une méthode de la _couche service_ (je vous fais grâce de la classe `Basket`)

```java
public int calculateTotalCost(Basket basket) {

   int totalCost = 0;
   
   for (Article article : basket.getArticles()) {
      totalCost += article.getUnitPrice() * article.getQuantity();
   }
   
   totalCost = totalCost + TAX_RATE * totalCost;
   
   basket.setTotalCost(totalCost);
   
   return totalCost;
}
```

Pourquoi en est-on arrivé là ?

- A cause de Java qui encourage les _getters_ et _setters_ sur les beans ? 
- A cause des librairies comme [lombok](https://projectlombok.org/), qui automatisent la création de ces accesseurs ? 
- A cause de l'architecture en couches qui a été usée et abusée ?
- A cause de l'héritage des langages procéduraux comme le C, et la fainéantise des développeurs pour changer de paradigme ?
- Ou peut-être n'y a-t-il pas de problème et qu'il s'agit d'une bonne manière de coder ?

Selon moi, il ne s'agit pas d'une manière correcte de coder ce genre de fonctionnalités et c'est pourquoi j'ai décidé d'écrire cet article.

## Le principe _Tell don't ask_

Ce principe stipule qu'au lieu de demander (_ask_) à un objet des informations pour les exploiter, il vaut mieux dire (_tell_) à cet objet ce que l'on veut faire et il s'en chargera lui même.

Selon ce principe, la classe `Basket` utilisée ci-dessus devient
```java
public class Basket {

   private List<Article> articles;

   // ...

   public int calculateTotalCost() {
   	
      int totalCost = 0;
      
      for (Article article : articles) {
      	totalCost += article.priceWithoutTaxes();
      }
      
      return totalCost + TAX_RATE * totalCost;   
   }
}
```

et le service devient

```java
public int calculateTotalCost(Basket basket) {
   return basket.calculateTotalCost();
}
```

Plusieurs remarques peuvent être faites à la lecture de ce nouveau code :

- Il n'y a plus d'accesseurs dans la classe `Basket`. Ses champs sont encapsulés et non exposés aux quatres vents : l'objet redevient responsable de son état.
- Le comportement est rapproché des données qu'il va utiliser. Ainsi, il est bien plus simple de comprendre le code (et donc de le maintenir) car les données traitées sont proches de leur traitement et elle ne sont plus utilisées ailleurs.
- La [Loi de Demeter](https://fr.wikipedia.org/wiki/Loi_de_D%C3%A9m%C3%A9ter) est respectée. Ainsi, un changement dans une méthode de calcul de prix n'affectera personne d'autre que la classe dont le prix doit être calculé.
- La question quant à l'utilité du service est bel et bien posée. Il ne s'agit plus que d'un passe-plat totalement inutile à part à pourrir la base de code.

## De l'utilité des services et des couches

En suivant ce genre de principes, il devient évident que le _service_, ce fameux [God Object](https://fr.wikipedia.org/wiki/God_object) qui faisait la pluie et le beau temps dans le code, est réduit à peau de chagrin. Il est même possible de s'en passer très souvent.

Cependant, il reste indispensable dans le cas où les objets doivent collaborer entre eux. Globalement, si je devais indiquer la _règle_ que j'essaie de suivre au quotidien, je dirais ceci :

- Si l'objet dispose de toutes les informations pour effectuer un traitement, il doit le faire.
- Si le traitement porte sur plusieurs objets différents, une classe externe (typiquement un service) peut s'en occuper.


## Restons pragmatiques

Comme toujours, les principes se confrontent à la réalité et ils ne gagnent pas toujours. Il faut parfois savoir faire des petites entorses à la règle et ne pas être dogmatique sur ce genre de sujets. Qui sait, les bonnes pratiques d'aujourd'hui seront peut-être les anti-patterns de demain.

Cependant, après quelques temps de pratique, je pense qu'il est possible d'appliquer le principe de _Tell Don't Ask_ au quotidien. Il est même possible de commencer dès aujourd'hui et sans devoir refactoriser l'intégralité du code source existant. 

Pensez à ça lors de vos prochains développement et n'hésitez pas à en parler sur [Twitter (@RnowIf)](https://twitter.com/RnowIf), dans un commentaire sur cette page ou bien lors d'un [Meetup Craftsman](http://www.meetup.com/fr/Software-Craftsmanship-Lyon/) à Lyon ou ailleurs.

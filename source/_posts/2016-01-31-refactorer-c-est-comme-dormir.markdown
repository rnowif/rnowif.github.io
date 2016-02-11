---
layout: post
title: "Refactorer, c'est comme dormir"
date: 2016-01-31 19:16:09 +0100
comments: true
categories: [refactoring, craftsmanship]
---

Robert est un développeur. On a demandé à Robert d'ajouter une nouvelle fonctionnalité à l'application. Robert aimerait bien en profiter pour refactorer un peu le code car il ne respecte pas les bonnes pratiques qu'il a pu voir lors du dernier meetup Software Craftsmanship. On dit alors à Robert qu'il ne peut pas refactorer car, en ce moment, il n'y a pas le temps pour ça et qu'il doit produire. Robert se dit qu'il n'y a de toutes façons jamais le temps...

Cette histoire vous semble familière ? En effet, le refactoring est souvent perçu comme n'apportant aucune valeur. D'autres choses dans la vie peuvent sembler n'apporter aucune valeur. Il y en a notamment une qui prend près du tiers de la vie d'une personne pendant laquelle cette personne ne fait littéralement rien : *dormir* !

<!-- more -->

Le nombre de choses que l'on pourrait faire si l'on ne dormait pas ! Cependant, au bout de quelques jours, voire quelques heures, des événements désagréables finiraient par survenir. Sans sommeil, la moindre des tâches peut devenir très longue à effectuer et des erreurs étranges peuvent arriver (comme verser du jus d'orange dans son bol de céréales). Après un certain temps, le sommeil prend le dessus de manière incontrôlée, ce qui peut conduire à des accidents (à cause d'une somnolence au volant par exemple).

De manière similaire, repousser le refactoring nuit au projet. Les développements peuvent prendre beaucoup plus de temps que nécessaire et des bugs peuvent survenir sur des fonctionnalités très éloignées de celles qui sont modifiées. De plus, il arrive un moment où le refactoring s'impose : le code ne peut plus être modifié sans risquer d'importantes régressions. Ceci peut se produire à un moment critique du projet et donc le mettre en péril. A ce moment là, la sentence tant redoutée peut tomber : « il faut tout réécrire ».

De manière globale, refactorer régulièrement est nécessaire à la bonne santé du projet, tout comme dormir chaque nuit l'est pour la notre. Dans la suite de cet article, nous allons définir ce que signifie refactorer dans ce contexte, quand est-il bon de le faire, comment bien le préparer et que faire si le temps manque.

## Que signifie refactorer ?

Selon Michael Feathers, le refactoring est l'acte d'améliorer le _design_ du code sans changer son comportement.

Il ne s'agit pas forcément de changer toute une hiérarchie de classes ou de mettre en place un _design pattern_ très complexe mais cela peut être aussi simple que de renommer une variable, une méthode, une classe, d'extraire une méthode privée dans une classe externe, de regrouper les attributs d'une méthode dans un objet, etc.

De plus, nous considérons aussi qu'ajouter des tests à du code existant est du refactoring. En effet, un code testable est un premier pas vers un meilleur _design_.

## Quand faut-il refactorer ?

L'idée n'est pas d'essayer de corriger l'intégralité du système à chaque fois. Ceci serait contre productif et impossible à mettre en place. De plus, il serait très difficilement justifiable de provoquer des régressions dans une partie du code très éloignée de celle qui est censée être modifiée.

Le refactoring est très efficace lorsqu'il est ciblé sur le code qui est concerné par le développement d'une fonctionnalité. De plus, il nous semble préférable de refactorer avant d'ajouter du nouveau code afin de démarrer sur des bases saines.

## Comment le préparer ?

La première chose à faire avant de refactorer est de s'assurer qu'il y a des tests en place. Ces tests permettent de vérifier que le refactoring ne génère pas de régression. Si ces tests ne sont pas déjà en place, il faut les ajouter avant de commencer.

Il peut arriver qu'il soit impossible de tester une portion de code. Dans ce cas, il faut effectuer le refactoring minimum nécessaire à la mise en place des tests.

## Que faire si le temps manque ?

Si, comme Robert, le temps vous manque, il faut faire preuve de pragmatisme. Il peut être intéressant d'effectuer de petits refactoing à chaque développement. Cela permet d'améliorer le _design_ sans consommer trop de temps et de préparer le terrain pour de plus gros refactoring.

## Conclusion

Le refactoring est essentiel dans un projet et doit être effectué régulièrement. Il permet d'assurer que les nouvelles fonctionnalités pourront être développées dans un temps raisonnable et de limiter les régressions en améliorant le _design_. Par ailleurs, il permet aussi aux développeurs de (re)prendre du plaisir à faire évoluer le produit.

Nous aimerions conclure avec la règle du boy scout qui indique que l'on doit toujours laisser le code dans un meilleur état que lorsqu'on l'a trouvé. L'application de cette règle conduit à l'amélioration de la qualité globale du code et à l'inversion de la dette technique.

---
_Cet article a été écrit en collaboration avec Nadia Humbert-Labeaumaz ([@nphumbert](https://www.twitter.com/nphumbert))_
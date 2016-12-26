---
layout: post
title: "Pourquoi utiliser le TDD est une bonne idée pour apprendre un nouveau langage"
date: 2016-12-26 17:44:15 +0100
comments: true
categories: ["TDD", "TDL", "Ruby"]
---

Durant ma dernière mission, j'ai eu à écrire du code dans des langages que je ne connaissais pas du tout, ou très peu (Ruby, Perl, Python, Go). J'ai donc expérimenté une technique qui consiste à apprendre un langage par les tests. Cette technique se nomme le _Test Driven Learning_.

L'objectif de cet article est de décrire pourquoi il peut être intéressant d'utiliser le TDD pour apprendre un nouveau langage de manière efficace.

<!-- more -->

## Découverte du langage

Écrire un test est suffisamment complexe pour découvrir pas mal de spécificités du langage mais suffisamment simple pour ne pas avoir besoin de trop chercher.
En effet, il faut bien créer un fichier, éventuellement une classe, instancier un objet, créer et appeler une fonction, définir une variable, etc. Cependant, il est très rapidement possible d'écrire un test sans avoir besoin de connaître le langage en détail : pas besoin de boucle, de structure conditionnelle ou bien d'héritage. De plus, exécuter un test est en général bien plus simple que de déployer le code (sur un serveur ou en local) pour le tester "à la main".

Par ailleurs, le fait de ne pas connaître le langage garantit que le test sera focalisé sur le comportement et donc totalement découplé de l'implémentation !

Ceci est un test en Ruby écrit avec le framework [rspec](http://rspec.info/). Ce test est pratiquement un copié / collé de l'exemple du tutoriel, adapté à mon contexte :

```ruby
describe GoogleQueryParser do
  describe "extract query" do
    it "should return the query when other parameters are set afterwards" do
      parser = GoogleQueryParser.new
      expect(parser.parse("/search?q=kibana&hl=fr")).to eq("kibana")
    end
  end
end
```

Après quelques essais, la structure est créée, le code est compilé et les tests sont lancés. L'implémentation peut commencer.

## Fil d'ariane

Une fois le test écrit, il sert de fil rouge pour guider l'implémentation. Ceci est également vrai quand le TDD est utilisé avec un langage que l'on maîtrise mais, quand le langage est inconnu, il est plus facile de se détourner du droit chemin. Le test permet donc de se concentrer sur un objectif fixé et clairement exprimé. C'est un véritable bac à sable où il est possible d'expérimenter sans risque.

## Refactoring facilité

Les premières lignes de code dans un nouveau langage sont rarement parfaites. Au fur et à mesure de l'apprentissage, il est possible de faire mieux. Cela tombe bien, car le TDD encourage justement à écrire une première implémentation qui fait seulement passer le test et refactorer ensuite.

Sans test, la découverte d'un nouveau langage peut être périlleuse. En effet, il est très simple d'écrire du code "qui marche" et de ne surtout plus le toucher après, de peur de tout casser. Dès son écriture, ce code est donc voué à être un fardeau pour toute l'équipe.

## Conclusion

J'ai très souvent entendu des gens dire qu'ils ne faisaient pas de TDD car ils ne maîtrisaient pas assez le langage. Je pense au contraire que le TDD est un excellent moyen pour apprendre un langage ou un paradigme. Il permet de s'assurer un harnais de sécurité et d'expérimenter sans aucun risque. Tous les avantages du TDD semblent exacerbés lorsque le langage cible est peu ou pas connu. En somme, je pense qu'avoir appris le Ruby par le TDD m'a permis d'être à la fois meilleur en Ruby **et** en TDD, tout en étant bien plus productif, malgré le gap technique.

Si vous souhaitez aller plus loin dans le TDL, je vous conseille de lire l'[article](https://ldez.github.io/blog/2015/12/04/test-driven-learning-go/) de [Ludovic Fernandez](https://twitter.com/ludnadez), où il apprend le Go par les tests.

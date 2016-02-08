---
layout: post
title: "Distribuer son application java sur Mac OSX grâce à Maven"
date: 2015-03-20 21:21:43 +0100
comments: true
categories: [Maven, Java, MacOSX]
---

Dans un article précédent, nous avons vu comment distribuer son application Java sur Windows grâce à Maven. Cet article vise à faire la même chose sur MacOSX.
Nous n'allons donc pas revenir sur l'alternative qui vise à distribuer directement le jar et passer directement à la partie qui nous intéresse, à savoir distribuer un exécutable Mac OSX.

<!-- more -->

## Distribuer son application sur Mac OSX

Sous Mac OSX, l'installation d'une application se fait via un fichier dmg. Le plugin [osxappbundle](http://mojo.codehaus.org/osxappbundle-maven-plugin/) permet de générer automatiquement ce fichier.

### Générer le fichier dmg sous Mac

L'utilisation de ce plugin est cependant différente selon qu'il est exécuté sous Mac ou sous un autre OS. En effet, le fichier dmg ne peut être généré *que sous Mac*.

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>osxappbundle-maven-plugin</artifactId>
  <version>1.0-alpha-1</version>
  <configuration>
    <bundleName>MyApp</bundleName>
    <mainClass>com.foo.MyApplication</mainClass>
    <internetEnable>true</internetEnable>
    <diskImageFile>${project.build.directory}/MyApp.dmg</diskImageFile>
  </configuration>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
      	<goal>bundle</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

### Générer le fichier app sous Windows ou Linux

Sous un autre OS, il n'est pas possible de générer le fichier dmg. Le plugin va seulement générer un fichier zip qui, une fois dézippé, va se transformer en application Mac (format app).
Ce fichier app pourra être exécuté en double cliquant dessus simplement.

Pour fonctionner, le plugin a besoin du fichier `JavaApplicationStub` qui se trouve dans le dossier `/System/Library/Frameworks/JavaVM.framework/Resources/MacOS` de MacOS. Dans l'exemple ci-dessous, ce fichier a été déposé dans le dossier `src/main/resources` du projet.

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>osxappbundle-maven-plugin</artifactId>
  <version>1.0-alpha-1</version>
  <configuration>
    <bundleName>MyApp</bundleName>
    <mainClass>com.foo.MyApplication</mainClass>
    <internetEnable>true</internetEnable>
    <javaApplicationStub>${basedir}/src/main/resources/JavaApplicationStub</javaApplicationStub>
    <buildDirectory>${project.build.directory}/MyApp.app</buildDirectory>
    <diskImageFile>${project.build.directory}/MyApp.dmg</diskImageFile>
    <zipFile>${project.build.directory}/MyApp.zip</zipFile>
  </configuration>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
      	<goal>bundle</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

## Conclusion

Générer un fichier exécutable pour MacOS n'est pas très compliqué. Cependant, afin de générer un fichier dmg, il faut absolument que la machine qui utilise le plugin maven tourne sous MacOS. Dans le cas contraire (un serveur d'intégration continue sous Linux par exemple), il faudra se contenter d'un fichier zip qui, une fois dézippé, pourra être ouvert comme une application normale.

---
layout: post
title: "Distribuer son application java sur Windows grâce à Maven"
date: 2015-03-19 21:25:16 +0100
comments: true
categories: [Maven, Java, Windows]
---

Vous avez développé une application java qui va révolutionner le monde. Bravo ! Maintenant, il vous faut la distribuer au monde entier afin que chacun puisse en profiter. Cet article a pour but de montrer comment le faire grâce à l'outil [Maven](http://maven.apache.org/). Il ne s'agit pas d'un article sur Maven en lui-même ni sur la mise en place d'un projet avec Maven.

<!-- more -->

## Distribuer directement l'archive jar
Lorsque l'on cherche à distribuer une application, le premier réflexe est de générer une archive jar. Voici une façon de générer un jar autoporteur avec Maven. Celui-ci va contenir toutes les dépendances que votre application pourrait avoir besoin.

```xml
<plugin>
   <artifactId>maven-assembly-plugin</artifactId>
   <executions>
      <execution>
         <id>application-assembly</id>
         <phase>package</phase>
         <goals>
            <goal>single</goal>
         </goals>
         <configuration>
            <descriptorRefs>
               <descriptorRef>jar-with-dependencies</descriptorRef>
            </descriptorRefs>
            <archive>
               <index>true</index>
               <manifest>
                  <addClasspath>true</addClasspath>
                  <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
                  <mainClass>com.foo.MyApplication</mainClass>
               </manifest>
               <manifestEntries>
                  <Application-Name>${project.name}</Application-Name>
                  <Permissions>all-permissions</Permissions>
               </manifestEntries>
            </archive>
         </configuration>
      </execution>
   </executions>
</plugin>
```


L’utilisateur devra alors récupérer le fichier `MyApplication-jar-with-dependencies.jar` et l’exécuter en lancant la commande suivante :

```
java -jar MyApplication-jar-with-dependencies.jar
```

Pas très pratique n’est-ce pas ? Dans la suite de l’article, nous verrons comment distribuer ce jar sous un format plus courant pour les utilisateurs.

## Distribuer son application sur Windows
### Fichier exécutable

L’utilisateur lambda de Windows est habitué à manipuler des fichiers exécutables *.exe où il n’a qu’à double-cliquer pour lancer. Maven est capable de générer cet exécutable grâce au plugin [launch4j](https://github.com/lukaszlenart/launch4j-maven-plugin).

```xml
<plugin>
   <groupId>com.akathist.maven.plugins.launch4j</groupId>
   <artifactId>launch4j-maven-plugin</artifactId>
   <executions>
      <execution>
         <id>l4j-clui</id>
         <phase>package</phase>
         <goals>
            <goal>launch4j</goal>
         </goals>
         <configuration>
            <dontWrapJar>false</dontWrapJar>
            <headerType>gui</headerType>
            <outfile>${project.build.directory}/MyApp.exe</outfile>
            <jar>${project.build.directory}/MyApplication-jar-with-dependencies.jar</jar>
            <jre>
               <minVersion>1.6.0</minVersion>
            </jre>
            <versionInfo>
               <fileVersion>1.0.0.0</fileVersion>
               <txtFileVersion>${project.version}</txtFileVersion>
               <fileDescription>Description de l'application</fileDescription>
               <copyright>Copyright FooCompany 2015</copyright>
               <productVersion>1.0.0.0</productVersion>
               <txtProductVersion>${project.version}</txtProductVersion>
               <companyName>FooCompany</companyName>
               <productName>Nom de l'application</productName>
               <internalName>Nom de l'application</internalName>
               <originalFilename>MyApp.exe</originalFilename>
            </versionInfo>
         </configuration>
      </execution>
   </executions>
</plugin>
```

Lors de la phase de package, Maven va alors générer le fichier `MyApp.exe` qui n’est rien d’autre qu’un wrapper autour du fichier jar autoporteur. Il suffira alors de double cliquer dessus pour l’exécuter. Cependant, ce n’est pas toujours suffisant d’avoir un exécutable seul pour lancer l’application. Vous pourriez avoir envie de mettre des raccourcis sur le bureau, dans le menu Démarrer, avoir une jolie icône, etc. Nous allons donc voir comment réaliser toutes ces choses à l’aide d’un installateur.

### Installateur

Pour que Maven génère l’installateur, il est possible d’utiliser le plugin [NSIS](http://mojo.codehaus.org/nsis-maven-plugin/).

Afin de faire fonctionner ce plugin, il faut plusieurs choses :

-  Installer NSIS sur la machine qui va builder l’application
- Modifier le `pom.xml` pour dire à Maven d’exécuter NSIS
- Configurer NSIS pour générer correctement l’installateur

#### Installer NSIS
NSIS peut être téléchargé depuis le site de l’éditeur : http://nsis.sourceforge.net/Main_Page 

#### Modifier le pom.xml
La partie à écrire dans le `pom.xml` est assez brève. Il suffit en fait d’indiquer à Maven les goals du plugin à appeler ainsi que le nom final de l’installateur.

```xml
<plugin>
   <groupId>org.codehaus.mojo</groupId>
   <artifactId>nsis-maven-plugin</artifactId>
   <version>1.0-alpha-1</version>
   <executions>
      <execution>
         <goals>
            <goal>generate-headerfile</goal>
            <goal>make</goal>
         </goals>
      </execution>
   </executions>
   <configuration>
      <outputFile>MyApp-setup.exe</outputFile>
   </configuration>
</plugin>
```

Le goal `generate-headfile` du plugin va générer un fichier `project.nsh` qui va définir des constantes telles que `PROJECT_NAME` ou `PROJECT_VERSION`. Le goal `make` va ensuite récupérer le fichier de configuration de NSIS pour générer l’installateur proprement dit.

#### Créer le fichier de configuration NSIS
Ce fichier, appelé `setup.nsi` doit se trouver à la racine du projet, à coté du `pom.xml`. Il contient toutes les actions à faire au moment de l’installation. Le langage utilisé est un langage de script rudimentaire permettant d’effectuer des actions simples comme copier un fichier, créer un raccourci, créer un dossier, etc.

Voici un exemple simple de setup.nsi qui va s’installer dans les programmes, créer des raccourcis dans le menu démarrer et sur le bureau. Il va également copier un fichier *.ico (qui doit se trouver à la racine du projet également) et l’utiliser comme icône de l’application. Finalement, il va créer un exécutable qui va permettre de désinstaller l’application (suppression de tous les dossiers et icônes créés précédemment).

_NB : Les constantes définies dans le project.nsh sont utilisables dans le setup.nsi_

```
; Installer for MyApp
;======================================================
; Includes
  !include MUI.nsh
  !include Sections.nsh
  !include target\project.nsh
;======================================================
; Installer Information
  Name "${PROJECT_NAME}"

  SetCompressor /SOLID lzma
  XPStyle on
  CRCCheck on
  InstallDir "C:\Program Files\FooCompany\"
  AutoCloseWindow false
  ShowInstDetails show
  Icon "${NSISDIR}\Contrib\Graphics\Icons\orange-install.ico"
  
;======================================================
; Version Tab information for Setup.exe properties

  VIProductVersion 2015.02.15.0
  VIAddVersionKey ProductName "${PROJECT_NAME}"
  VIAddVersionKey ProductVersion "${PROJECT_VERSION}"
  VIAddVersionKey CompanyName "${PROJECT_ORGANIZATION_NAME}"
  VIAddVersionKey FileVersion "${PROJECT_VERSION}"
  VIAddVersionKey FileDescription ""
  VIAddVersionKey LegalCopyright ""

;======================================================
; Modern Interface Configuration

  !define MUI_HEADERIMAGE
  !define MUI_ABORTWARNING
  !define MUI_COMPONENTSPAGE_SMALLDESC
  !define MUI_HEADERIMAGE_BITMAP_NOSTRETCH
  !define MUI_FINISHPAGE
  !define MUI_FINISHPAGE_TEXT "Thank you for installing ${PROJECT_NAME}."
  !define MUI_ICON "${NSISDIR}\Contrib\Graphics\Icons\orange-install.ico"
  
;======================================================
; Modern Interface Pages

  !define MUI_DIRECTORYPAGE_VERIFYONLEAVE
  !insertmacro MUI_PAGE_DIRECTORY
  !insertmacro MUI_PAGE_COMPONENTS
  !insertmacro MUI_PAGE_INSTFILES
  !insertmacro MUI_PAGE_FINISH
  
;======================================================
; Languages

  !insertmacro MUI_LANGUAGE "English"

;======================================================
; Installer Sections

Section "MyApp"
   SetOutPath $INSTDIR
   SetOverwrite on
    
   File target\MyApp.exe
   File MyApp.ico
    
   CreateShortCut "$DESKTOP\MyApp.lnk" "$INSTDIR\MyApp.exe" "" "$INSTDIR\MyApp.ico" 0
    
   CreateDirectory "$SMPROGRAMS\FooCompany"
   CreateShortcut "$SMPROGRAMS\FooCompany\MyApp.lnk" "$INSTDIR\MyApp.exe" "" "$INSTDIR\MyApp.ico" 0
   CreateShortCut "$SMPROGRAMS\FooCompany\Uninstall.lnk" "$INSTDIR\Uninstall.exe" "" "$INSTDIR\Uninstall.exe" 0

   writeUninstaller "$INSTDIR\MyApp_uninstall.exe"
SectionEnd

Section "uninstall"
   ;Delete Files 
   RMDir /r "$INSTDIR\*.*"    
	 
   ;Remove the installation directory
   RMDir "$INSTDIR"
	 
   ;Delete Start Menu Shortcuts
   Delete "$DESKTOP\MyApp.lnk"
   Delete "$SMPROGRAMS\FooCompany\*.*"
   RmDir  "$SMPROGRAMS\FooCompany"
SectionEnd

Function .onInit
   InitPluginsDir
FunctionEnd
```

Pour plus de possibilités, vous pouvez consulter la [documentation](http://nsis.sourceforge.net/Docs/).

## Conclusion

Cet article a permis de voir qu’à l’aide de quelques plugins Maven, il est relativement simple de générer un installateur Windows qui va installer votre application java sur un poste Windows. Il est important de noter que cet installateur est généré même si le build est lancé sous Linux. Ainsi, un serveur d’intégration continue pourra automatiquement générer l’installateur à la seule condition que NSIS soit installé sur la machine.

Dans un prochain article, nous verrons comment générer un fichier exécutable pour distribuer votre application sous Mac OS X.

Et-vous, avez-vous déjà essayé de générer automatiquement des installateurs pour Windows ? Comment avez-vous fait ? N’hésitez pas à déposer un commentaire pour en discuter.
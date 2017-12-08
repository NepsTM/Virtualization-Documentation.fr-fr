---
title: "Créer un exemple d’application"
description: "Découvrez comment créer un exemple d’application en tirant parti des conteneurs"
keywords: docker, conteneurs
author: cwilhit
ms.date: 07/25/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: b1d0c4bcf35cd40e9ca058d4e2a51fa028cade2c
ms.sourcegitcommit: 04c78918c77d2ad6053e6a95dc57bc488efbbf8b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/31/2017
---
# <a name="build-a-sample-app"></a>Créer un exemple d’application

Cet exercice vous guidera tandis que vous utilisez un exemple d’application ASP.net, puis le convertissez afin de l’exécuter dans un conteneur. Pour vous familiariser avec les conteneurs et leur exécution dans Windows10, consultez le [démarrage rapide Windows10](./quick-start-windows-10.md).

Ce démarrage rapide est spécifique à Windows10. Une documentation de démarrage rapide supplémentaire est disponible dans la table des matières affichée sur la gauche de cette page. Dans la mesure où ce didacticiel se rapporte avant tout aux conteneurs, l’écriture du code n’est pas abordée et nous nous concentrerons uniquement sur les conteneurs. Si vous voulez créer entièrement ce didacticiel, il est disponible dans la [documentation de base ASP.NET](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app-xplat/)

Si le contrôle de code de source Git n’est pas installé sur votre ordinateur, vous pouvez l’obtenir ici: [Git](https://git-scm.com/download)

## <a name="getting-started"></a>Démarrage

Cet exemple de projet a été configuré avec [VSCode](https://code.visualstudio.com/). Nous utiliserons également PowerShell. Commençons par récupérer le code de démonstration à partir de github. Vous pouvez cloner le référentiel à l’aide de git ou télécharger le projet directement à partir de [SampleASPContainerApp](https://github.com/cwilhit/SampleASPContainerApp).

```Powershell
git clone https://github.com/cwilhit/SampleASPContainerApp.git
```

Nous allons maintenant accéder au répertoire du projet et créer le fichier Dockerfile. Un [fichier Dockerfile](https://docs.docker.com/engine/reference/builder/) est similaire à un fichier makefile. Il s’agit d’une liste d’instructions décrivant comment une image de conteneur doit être créée.

```Powershell
#Create the dockerfile for our proj
New-Item C:/Your/Proj/Location/Dockerfile -type file
```

## <a name="writing-our-dockerfile"></a>Écriture du fichier Dockerfile

Nous allons ouvrir le fichier Dockerfile que nous avons créé dans le dossier racine du projet (avec l’éditeur de texte de votre choix) et lui ajouter une logique. Ensuite, nous l’examinerons ligne par ligne pour expliquer ce qui se passe.

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

Le premier groupe de lignes déclare à partir de quelle image de base nous allons créer le conteneur. Si cette image ne se trouve pas sur le système local, Docker essaie automatiquement de la récupérer. Aspnetcore-build inclut les dépendances nécessaires pour compiler le projet. Ensuite, nous modifions le répertoire de travail en «/app» dans le conteneur, afin que toutes les commandes ultérieures du fichier Dockerfile soient exécutées à cet emplacement.

_Remarque_: dans la mesure où nous devons créer le projet, le premier conteneur que nous créons est un conteneur temporaire que nous utiliserons spécifiquement à cette fin, avant de le supprimer à la fin de l’opération.

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app
```

Ensuite, nous copions les fichiers.csproj dans le conteneur temporaire situé dans le répertoire «/app». Cette étape est nécessaire, car les fichiers .csproj contiennent la liste des références de package requises par le projet.

Une fois ce fichier copié, dotnet le lit, puis récupère toutes les dépendances et les outils nécessaires pour le projet.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

Une fois toutes ces dépendances récupérées, nous les copions dans le conteneur temporaire. Ensuite, nous demandons à dotnet de publier notre application avec une version finale et nous spécifions le chemin de sortie.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

Le projet devrait avoir été correctement compilé. Nous devons maintenant créer notre conteneur final. Dans la mesure où il s’agit d’une application ASP.NET, nous spécifions une image avec ces bibliothèques comme source. Ensuite, nous copions tous les fichiers du répertoire de sortie de notre conteneur temporaire dans notre conteneur final. Nous configurons notre conteneur afin qu’il s’exécute avec la nouvelle .dll que nous avions compilée lors de son lancement.

_Remarque_: l’image de base de ce conteneur final est similaire mais différente de la commande ```FROM```ci-dessus. Elle ne contient pas les bibliothèques permettant de _créer_ une application ASP.NET, mais uniquement de l’exécuter.

```Dockerfile
FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

Nous venons de réaliser ce que l’on appelle une _création échelonnée_. Nous avons utilisé le conteneur temporaire pour créer l’image, puis déplacé la dll publiée dans un autre conteneur afin de réduire l’encombrement du résultat final. Nous voulons que ce conteneur utilise le moins de dépendances possible pour s’exécuter. Si nous avions continué à utiliser la première image, celle-ci comporterait d’autres couches (pour la création d’applications ASP.NET) non essentielles, ce qui aurait pour conséquence d’augmenter la taille de l’image.

## <a name="running-the-app"></a>Exécution de l’application

Maintenant que nous avons écrit le fichier Dockerfile, il ne reste plus qu’à demander à Docker de créer l’application et d’exécuter le conteneur. Nous spécifions le port de publication, puis ajoutons la balise «myapp» au conteneur. Exécutez les commandes suivantes dans PowerShell.

_Remarque_: le répertoire de travail actuel de votre console PowerShell doit être le répertoire où réside le fichier dockerfile créé ci-dessus.

```Powershell
docker build -t myasp .
docker run -d -p 5000:80 --name myapp myasp
```

Pour voir l’application en cours d’exécution, nous devons accéder à l’adresse à laquelle elle est en cours d’exécution. Nous pouvons obtenir l’adresseIP en exécutant la commande suivante.

```Powershell
 docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" myapp
```

L’exécution de cette commande génère l’adresseIP du conteneur en cours d’exécution. Voici un exemple de ce à quoi peut ressembler la sortie:

```Powershell
 172.19.172.12
```

Entrez cette adresseIP dans le navigateur web de votre choix et vous êtes accueilli par l’application fonctionnant correctement dans un conteneur!

<center style="margin: 25px">![](media/SampleAppScreenshot.png)</center>

En cliquant sur «MvcMovie» dans la barre de navigation, vous êtes redirigé vers une page web où vous pouvez saisir, modifier et supprimer des entrées de vidéo.

## <a name="next-steps"></a>Étapes suivantes

Après avoir récupéré une application web ASP.NET, nous l’avons correctement configurée et générée à l’aide de Docker, puis nous l’avons déployée dans un conteneur en cours d’exécution. Vous pouvez aller encore plus loin! Ainsi, vous pouvez diviser l’application web en plusieurs composants avec, par exemple, un conteneur qui exécute l’API web, un conteneur qui exécute le serveur frontal et un autre conteneur qui exécute le serveur SQL Server.

Maintenant que vous êtes familiarisé avec les conteneurs, il ne vous reste plus qu’à mettre ces informations en pratique et à créer de superbes applications logicielles! Vous trouverez ci-dessous une liste d’autres exemples de conteneurs:

[Exemples de conteneurs](../samples.md)

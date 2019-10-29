---
title: Conteneur d’une application .NET Core
description: Découvrez comment créer un exemple d’application .NET Core avec des conteneurs
keywords: docker, conteneurs
author: cwilhit
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: db3caea3f7911ec6641930302198f976bd61240d
ms.sourcegitcommit: da762ce138467e50dce22d5086ad407138b38e48
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2019
ms.locfileid: "10261828"
---
# <a name="containerize-a-net-core-app"></a>Conteneur d’une application .NET Core

Ce segment suppose que votre environnement de développement est déjà configuré pour l’utilisation de conteneurs. Si vous n’avez pas de environnement configuré pour les conteneurs, consultez la section «[configurer votre environnement](./set-up-environment.md)» pour découvrir comment commencer.

Le système de contrôle de source git doit être installé sur votre ordinateur. Vous pouvez le saisir ici: [git](https://git-scm.com/download)

## <a name="clone-the-sample-code"></a>Cloner l’exemple de code

Tout le code source de l’exemple de conteneur est conservé sous la section [virtualisation-](https://github.com/MicrosoftDocs/Virtualization-Documentation) référentiel samples de la `windows-container-samples`documentation dans un dossier appelé. Clonez ce git référentiel samples dans votre répertoire de travail actuel.

```Powershell
git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
```

Accédez à l’exemple d’annuaire figurant `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` dans et créez un Dockerfile. Un [Dockerfile](https://docs.docker.com/engine/reference/builder/) est semblable à un Makefile, une liste d’instructions qui indiquent au moteur de conteneur la façon dont l’image du conteneur doit être créée.

```Powershell
# navigate into the sample directory
Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

# create the Dockerfile for our project
New-Item -Name Dockerfile -ItemType file
```

## <a name="write-the-dockerfile"></a>Écrire le dockerfile

Ouvrez le dockerfile que vous venez de créer (avec n’importe quel éditeur de texte) et ajoutez le contenu suivant.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

Divisez-le en ligne et expliquez ce que fait chaque instruction.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

Le premier groupe de lignes déclare à partir de quelle image de base nous allons créer le conteneur. Si cette image ne se trouve pas sur le système local, Docker essaie automatiquement de la récupérer. Le `mcr.microsoft.com/dotnet/core/sdk:2.1` Kit de développement logiciel (SDK) .net Core 2,1 est intégré au programme de création de projets principaux ASP .net ciblant la version 2,1. L’instruction suivante transforme le répertoire de travail dans son conteneur `/app`, de sorte que toutes les commandes qui suivent celle-ci s’exécutent sous ce contexte.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

Les instructions suivantes sont ensuite copiées sur les fichiers. csproj `build-env` dans l' `/app` annuaire du conteneur. Après avoir copié ce fichier, .NET en liraa à partir de celui-ci, puis aura accès à toutes les dépendances et outils nécessaires à notre projet.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

Lorsque .NET a extrait toutes les dépendances dans `build-env` le conteneur, l’instruction suivante copie tous les fichiers sources du projet dans le conteneur. Ensuite, nous indiquons à .NET de publier l’application auprès d’une configuration de publication et de spécifier le chemin de sortie dans le.

La compilation doit aboutir. Nous devons maintenant générer l’image finale. 

> [!TIP]
> Ce démarrage rapide génère un projet .NET principal à partir de la source. Lors de la création d’images de conteneur, il est conseillé d’inclure _uniquement_ la charge utile de production et ses dépendances dans l’image du conteneur. Nous ne voulons pas que le kit de développement logiciel (SDK) .NET Core est inclus dans notre image finale, car nous avons uniquement besoin de .NET Core Runtime, de sorte que dockerfile est écrit `build-env` de manière à utiliser un conteneur temporaire fourni avec le SDK appelé pour générer l’application.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

Dans la mesure où notre application est ASP.NET, nous spécifions une image avec ce Runtime inclus. Ensuite, nous copions tous les fichiers du répertoire de sortie de notre conteneur temporaire dans notre conteneur final. Nous configurons notre conteneur pour qu’il s’exécute avec notre nouvelle application comme point d’entrée lors du démarrage du conteneur.

Nous avons rédigé le dockerfile pour effectuer une _Build en plusieurs étapes_. Lorsque le dockerfile est exécuté, il utilise le conteneur temporaire, `build-env`avec le kit de développement logiciel .net Core 2,1 pour générer l’exemple d’application, puis copie les fichiers binaires entrés dans un autre conteneur contenant uniquement le Runtime .net Core 2,1, afin que nous n’ayons plus à limiter la taille du conteneur final.

## <a name="run-the-app"></a>Exécuter l’application

Avec dockerfile, nous pouvons faire pointer l’ancrage sur notre dockerfile et lui dire de générer une image. 

>[!IMPORTANT]
>La commande exécutée ci-dessous doit être exécutée dans le répertoire où se trouve le dockerfile.

```Powershell
docker build -t my-asp-app .
```

Pour exécuter le conteneur, exécutez la commande suivante.

```Powershell
docker run -d -p 5000:80 --name myapp my-asp-app
```

Dissect la commande suivante:

* `-d` indique à l’tunur d’exécuter le conteneur «détaché», ce qui signifie qu’il n’y a pas de console relié à la console à l’intérieur du conteneur. Le conteneur s’exécute en arrière-plan. 
* `-p 5000:80` indique à l’ancrage de mapper le port 5000 sur l’hôte au port 80 dans le conteneur. Chaque conteneur obtient sa propre adresse IP. ASP .NET écoute par défaut sur le port 80. Le mappage de port nous permet d’accéder à l’adresse IP de l’hôte au niveau du port mappé et de l’arrière-plan transférant tout le trafic vers le port de destination à l’intérieur du conteneur.
* `--name myapp` indique à l’expéditeur qu’il doit fournir un nom pratique de requête (au lieu d’avoir besoin de Rechercher l’ID contaienr affecté lors de l’exécution par l’Arrimateur).
* `my-asp-app` est l’image que nous voulons qu’elle exécute. Il s’agit de l’image de conteneur créée en tant que `docker build` «culminement du processus».

Ouvrez un navigateur Web et accédez `http://localhost:5000` à votre application conteneur pour l’accueillir.

>![](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>Étapes suivantes

Nous avons réussi à conteneurr une application Web ASP.NET. Pour afficher d’autres exemples d’applications et leurs dockerfiles associées, cliquez sur le bouton ci-dessous.

> [!div class="nextstepaction"]
> [Voir d’autres exemples de conteneurs](../samples.md)

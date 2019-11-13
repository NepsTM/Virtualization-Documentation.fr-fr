---
title: Conteneur d’une application .NET Core
description: Découvrez comment créer un exemple d’application .NET Core avec des conteneurs
keywords: docker, conteneurs
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: fab0dc46ddcc8c82a010d408032e5f3c4cea8d69
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288067"
---
# <a name="containerize-a-net-core-app"></a>Conteneur d’une application .NET Core

Cette rubrique décrit comment empaqueter un exemple d’application .NET pour déploiement en tant que conteneur Windows, après avoir configuré votre environnement comme décrit dans la rubrique [prise en main: préparer Windows pour les conteneurs](set-up-environment.md)et exécuter votre premier conteneur comme décrit dans [exécuter votre premier conteneur Windows](run-your-first-container.md).

Le système de contrôle de code source git doit également être installé sur votre ordinateur. Pour l’installer, visitez le site [git](https://git-scm.com/download).

## <a name="clone-the-sample-code-from-github"></a>Cloner l’exemple de code de GitHub

Tout le code source de l’exemple de conteneur est conservé sous la [virtualisation-](https://github.com/MicrosoftDocs/Virtualization-Documentation) référentiel git de la documentation (connu sous le nom de référentiel Samples) dans un `windows-container-samples`dossier appelé.

1. Ouvrez une session PowerShell et changez de répertoires pour le dossier dans lequel vous voulez stocker ce référentiel. (D’autres types de fenêtre d’invite de commandes fonctionnent également, mais nos exemples de commandes utilisent PowerShell.)
2. Clonez le référentiel samples dans votre répertoire de travail actuel:

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. Accédez à l’exemple d’annuaire figurant `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` dans et créez un Dockerfile à l’aide des commandes suivantes.

   Un [Dockerfile](https://docs.docker.com/engine/reference/builder/) est comme un Makefile: il s’agit d’une liste d’instructions indiquant au moteur du conteneur comment créer l’image du conteneur.

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Écrire le Dockerfile

Ouvrez le Dockerfile que vous venez de créer à l’aide de l’éditeur de texte de votre choix, puis ajoutez le contenu suivant:

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

Nous avons rédigé le dockerfile pour effectuer une _Build en plusieurs étapes_. Lorsque le dockerfile est exécuté, il utilise le conteneur temporaire, `build-env`avec le kit de développement logiciel .net Core 2,1 pour générer l’exemple d’application, puis copie les fichiers binaires entrés dans un autre conteneur contenant uniquement le Runtime .net Core 2,1, afin que nous n’ayons plus à réduire la taille du conteneur final.

## <a name="build-and-run-the-app"></a>Générer et exécuter l’application

Avec Dockerfile, nous pouvons vous permettre de pointer sur notre Dockerfile et de lui dire de générer et d’exécuter l’image suivante:

1. Dans une fenêtre d’invite de commandes, accédez au répertoire dans lequel se trouve le dockerfile, puis exécutez la commande de build de l' [amarrage](https://docs.docker.com/engine/reference/commandline/build/) pour générer le conteneur à partir du dockerfile.

   ```Powershell
   docker build -t my-asp-app .
   ```

2. Pour exécuter le conteneur nouvellement créé, exécutez la commande d’exécution de l' [ancrage](https://docs.docker.com/engine/reference/commandline/run/) .

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   Dissect la commande suivante:

   * `-d` indique à l’tunur d’exécuter le conteneur «détaché», ce qui signifie qu’il n’y a pas de console relié à la console à l’intérieur du conteneur. Le conteneur s’exécute en arrière-plan. 
   * `-p 5000:80` indique à l’ancrage de mapper le port 5000 sur l’hôte au port 80 dans le conteneur. Chaque conteneur obtient sa propre adresse IP. ASP .NET écoute par défaut sur le port 80. Le mappage de port nous permet d’accéder à l’adresse IP de l’hôte au niveau du port mappé et de l’arrière-plan transférant tout le trafic vers le port de destination à l’intérieur du conteneur.
   * `--name myapp` indique à l’expéditeur qu’il doit fournir un nom pratique de requête (au lieu d’avoir besoin de Rechercher l’ID contaienr affecté lors de l’exécution par l’Arrimateur).
   * `my-asp-app` est l’image que nous voulons qu’elle exécute. Il s’agit de l’image de conteneur créée en tant que `docker build` «culminement du processus».

3. Ouvrez un navigateur Web, puis accédez `http://localhost:5000` à l’application pour afficher votre application conteneur, comme illustré dans la capture d’écran suivante:

   >![Page Web principale de ASP.NET, en cours d’exécution à partir du localhost dans un conteneur](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>Étapes suivantes

1. L’étape suivante consiste à publier votre application Web ASP.NET dans un registre privé à l’aide du registre de conteneur Azure. Cela vous permet de le déployer dans votre organisation.

   > [!div class="nextstepaction"]
   > [Créer un registre de conteneur privé](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   Lorsque vous accédez à la section sur laquelle vous avez [envoyé votre image de conteneur dans le registre](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry), spécifiez le nom de l’application ASP.NET`my-asp-app`que vous venez de empaqueter () avec `contoso-container-registry`le registre de votre conteneur (par exemple:):

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   Pour afficher d’autres exemples d’applications et leurs dockerfiles associées, voir [exemples de conteneurs supplémentaires](../samples.md).

2. Une fois que vous avez publié votre application sur le registre de conteneurs, l’étape suivante consiste à déployer l’application sur un cluster Kubernetes que vous créez avec le service Azure Kubernetes.

   > [!div class="nextstepaction"]
   > [Créer un groupe Kubernetes](https://docs.microsoft.com/azure/aks/windows-container-cli)

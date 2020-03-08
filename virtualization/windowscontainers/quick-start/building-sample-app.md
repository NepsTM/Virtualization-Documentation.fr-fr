---
title: Conteneur d’une application .NET Core
description: Apprenez à créer un exemple d’application .NET Core avec des conteneurs
keywords: docker, conteneurs
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 587e8de5f0d593f92f6301c87bf68e08a8bbd839
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78854003"
---
# <a name="containerize-a-net-core-app"></a>Conteneur d’une application .NET Core

Cette rubrique explique comment empaqueter un exemple d’application .NET existant pour le déploiement en tant que conteneur Windows, après avoir configuré votre environnement, comme décrit dans [prise en main : préparer Windows pour les conteneurs](set-up-environment.md)et exécuter votre premier conteneur comme décrit dans [exécuter votre premier conteneur Windows](run-your-first-container.md).

Le système de contrôle de code source git doit également être installé sur votre ordinateur. Pour l’installer, visitez [git](https://git-scm.com/download).

## <a name="clone-the-sample-code-from-github"></a>Cloner l’exemple de code à partir de GitHub

Tous les exemples de code source de conteneur sont conservés dans le référentiel git de [documentation de virtualisation](https://github.com/MicrosoftDocs/Virtualization-Documentation) (connu de manière informelle en tant que référentiel) dans un dossier appelé `windows-container-samples`.

1. Ouvrez une session PowerShell et accédez au dossier dans lequel vous souhaitez stocker ce dépôt. (Les autres types de fenêtre d’invite de commandes fonctionnent également, mais nos exemples de commandes utilisent PowerShell.)
2. Clonez le référentiel dans votre répertoire de travail actuel :

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. Accédez à l’exemple de répertoire situé sous `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` et créez un fichier dockerfile à l’aide des commandes suivantes.

   Un [fichier dockerfile](https://docs.docker.com/engine/reference/builder/) est comme un Makefile : il s’agit d’une liste d’instructions qui indiquent au moteur de conteneur comment créer l’image de conteneur.

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Écrire le fichier dockerfile

Ouvrez le fichier dockerfile que vous venez de créer avec l’éditeur de texte de votre choix, puis ajoutez le contenu suivant :

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

Divisez-le en ligne par ligne et expliquez ce que fait chaque instruction.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

Le premier groupe de lignes déclare à partir de quelle image de base nous allons créer le conteneur. Si cette image ne se trouve pas sur le système local, Docker essaie automatiquement de la récupérer. Le `mcr.microsoft.com/dotnet/core/sdk:2.1` est fourni avec le kit de développement logiciel (SDK) .NET Core 2,1 installé. il s’agit donc de la tâche de création de projets ASP .NET Core ciblant la version 2,1. L’instruction suivante modifie le répertoire de travail dans notre conteneur pour qu’il soit `/app`, donc toutes les commandes qui suivent celle-ci s’exécutent dans ce contexte.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

Ensuite, ces instructions copient les fichiers. csproj dans le répertoire de `/app` du conteneur `build-env`. Une fois ce fichier copié, .NET le lit, puis récupère toutes les dépendances et tous les outils nécessaires à notre projet.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

Une fois que .NET a extrait toutes les dépendances dans le conteneur `build-env`, l’instruction suivante copie tous les fichiers sources du projet dans le conteneur. Nous indiquons ensuite à .NET de publier notre application avec une configuration Release et de spécifier le chemin de sortie dans le.

La compilation doit être effectuée correctement. Nous devons à présent créer l’image finale. 

> [!TIP]
> Ce guide de démarrage rapide crée un projet .NET Core à partir de la source. Lors de la génération d’images de conteneur, il est conseillé d’inclure _uniquement_ la charge utile de production et ses dépendances dans l’image de conteneur. Nous ne voulons pas que le kit de développement logiciel (SDK) .NET Core soit inclus dans notre image finale, car nous avons uniquement besoin du Runtime .NET Core, donc le fichier dockerfile est écrit pour utiliser un conteneur temporaire qui est empaqueté avec le kit de développement logiciel (SDK) appelé `build-env` pour créer l’application.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

Étant donné que notre application est ASP.NET, nous spécifions une image avec ce Runtime inclus. Ensuite, nous copions tous les fichiers du répertoire de sortie de notre conteneur temporaire dans notre conteneur final. Nous configurons notre conteneur pour qu’il s’exécute avec notre nouvelle application comme son point d’entrée au démarrage du conteneur.

Nous avons écrit le fichier dockerfile pour effectuer une _génération en plusieurs étapes_. Lorsque le fichier dockerfile est exécuté, il utilise le conteneur temporaire, `build-env`, avec le kit de développement logiciel (SDK) .NET Core 2,1 pour générer l’exemple d’application, puis copier les fichiers binaires générés dans un autre conteneur contenant uniquement le Runtime .NET Core 2,1, afin de réduire la taille du conteneur final.

## <a name="build-and-run-the-app"></a>Générer et exécuter l'application

Une fois l’fichier dockerfile écrit, nous pouvons pointer le Dockeur sur notre fichier dockerfile et lui demander de générer et d’exécuter l’image :

1. Dans une fenêtre d’invite de commandes, accédez au répertoire où se trouve le fichier dockerfile, puis exécutez la commande [dockr Build](https://docs.docker.com/engine/reference/commandline/build/) pour générer le conteneur à partir du fichier dockerfile.

   ```Powershell
   docker build -t my-asp-app .
   ```

2. Pour exécuter le conteneur qui vient d’être créé, exécutez la commande [dockr Run](https://docs.docker.com/engine/reference/commandline/run/) .

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   Étudier en détail cette commande :

   * `-d` indique à Dockr exécutez d’exécuter le conteneur’détaché', ce qui signifie qu’aucune console n’est raccordée à la console à l’intérieur du conteneur. Le conteneur s’exécute en arrière-plan. 
   * `-p 5000:80` indique à l’arrimeur de mapper le port 5000 sur l’hôte au port 80 dans le conteneur. Chaque conteneur obtient sa propre adresse IP. ASP .NET écoute par défaut sur le port 80. Le mappage de port nous permet d’accéder à l’adresse IP de l’hôte au niveau du port mappé et l’arrimeur transmet tout le trafic vers le port de destination à l’intérieur du conteneur.
   * `--name myapp` indique à Dockr d’attribuer à ce conteneur un nom pratique à interroger (au lieu d’avoir à Rechercher l’ID contaienr assigné au moment de l’exécution par docker).
   * `my-asp-app` est l’image que l’Assistant de connexion doit exécuter. Il s’agit de l’image de conteneur produite comme l’aboutissement du processus de `docker build`.

3. Ouvrez un navigateur Web et accédez à `http://localhost:5000` pour voir votre application en conteneur, comme illustré dans cette capture d’écran :

   >![ASP.NET Core page Web, exécution à partir de localhost dans un conteneur](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>Étapes suivantes :

1. L’étape suivante consiste à publier votre application Web ASP.NET en conteneur dans un registre privé à l’aide de Azure Container Registry. Cela vous permet de le déployer dans votre organisation.

   > [!div class="nextstepaction"]
   > [Créer un registre de conteneurs privé](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   Lorsque vous accédez à la section où vous envoyez [votre image conteneur vers le registre](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry), spécifiez le nom de l’application ASP.net que vous venez de empaqueter (`my-asp-app`) avec votre registre de conteneurs (par exemple : `contoso-container-registry`) :

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   Pour voir d’autres exemples d’applications et les fichiers dockerfile associés, consultez [exemples de conteneurs supplémentaires](../samples.md).

2. Une fois que vous avez publié votre application dans le registre de conteneurs, l’étape suivante consiste à déployer l’application sur un cluster Kubernetes que vous créez avec le service Azure Kubernetes.

   > [!div class="nextstepaction"]
   > [Créer un cluster Kubernetes](https://docs.microsoft.com/azure/aks/windows-container-cli)

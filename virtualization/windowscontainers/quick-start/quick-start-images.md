---
title: Démarrage rapide du déploiement de conteneurs- Images
description: Démarrage rapide du déploiement de conteneurs
keywords: docker, conteneurs
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
ms.openlocfilehash: 41fa89dcaba38d43d39681240a1a108c9250ba78
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9575170"
---
# <a name="automating-builds-and-saving-images"></a>Automatisation des builds et enregistrement des images

Dans le démarrage rapide précédent de Windows Server, un conteneur Windows était créé à partir d’un exemple .NetCore préalablement créé. Cet exercice montre comment créer votre propre image de conteneur à partir d’un fichier Dockerfile et stocker l’image de conteneur dans le registre public Docker Hub.

Ce démarrage rapide est spécifique aux conteneurs Windows Server sur Windows Server 2019 ou Windows Server 2016 et utilisera l’image de base du conteneur Windows Server Core. Une documentation de démarrage rapide supplémentaire est disponible dans la table des matières affichée à gauche dans cette page.

## <a name="prerequisites"></a>Prérequis

Vérifiez que vous respectez les exigences suivantes:

- Un système informatique (physique ou virtuel) exécutant Windows Server 2019 ou Windows Server 2016.
- Configurez ce système avec la fonctionnalité de conteneur Windows et Docker. Pour une procédure pas à pas sur ces étapes, voir [les conteneurs Windows sur Windows Server](./quick-start-windows-server.md).
- Un ID Docker, utilisé pour transférer (push) une image de conteneur vers Docker Hub. Si vous n’avez pas encore d’ID Docker, demandez-en un sur [Docker Cloud](https://cloud.docker.com/).

## <a name="container-image---dockerfile"></a>Image de conteneur - fichier Dockerfile

Bien qu’un conteneur puisse être créé, modifié, puis capturé manuellement dans une nouvelle image de conteneur, Docker inclut une méthode pour automatiser ce processus à l’aide d’un fichier Dockerfile. Pour effectuer cet exercice, vous avez besoin d’un ID Docker. Si vous n’avez pas encore d’ID Docker, demandez-en un sur [Docker Cloud]( https://cloud.docker.com/).

Sur l’hôte de conteneur, créez un répertoire `c:\build` dans lequel vous créez un fichier nommé `Dockerfile`. Remarque: Le fichier ne doit pas avoir d’extension de fichier.

```console
powershell new-item c:\build\Dockerfile -Force
```

Ouvrez le fichier Dockerfile dans le Bloc-notes.

```console
notepad c:\build\Dockerfile
```

Copiez le texte suivant dans le fichier Dockerfile, puis enregistrez ce dernier. Ces commandes indiquent à Docker de créer une image en se servant de `microsoft/iis` comme base. Le fichier Dockerfile exécute ensuite les commandes spécifiées dans l’instruction `RUN`. Dans le cas présent, le fichier index.html est mis à jour avec le nouveau contenu.

Pour plus d’informations sur les fichiers Dockerfile, voir [Fichiers Dockerfile sur Windows](../manage-docker/manage-windows-dockerfile.md).

```dockerfile
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

La commande `docker build` démarre le processus de génération de l’image. Le paramètre `-t` indique au processus de génération de nommer la nouvelle image `iis-dockerfile`. **Remplacez «user» par le nom d’utilisateur de votre compte Docker**. Si vous n’avez pas encore de compte Docker, demandez-en un sur [Docker Cloud](https://cloud.docker.com/).

```console
docker build -t <user>/iis-dockerfile c:\Build
```

Une fois l’opération terminée, vous pouvez vérifier que l’image a été créée à l’aide de la commande `docker images`.

```console
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

Maintenant, déployez un conteneur avec la commande suivante, en remplaçant «user» par votre ID Docker.

```console
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

Une fois le conteneur créé, accédez à l’adresse IP de l’hôte de conteneur. Vous devez voir l’application hello world.

![](media/dockerfile2.png)

De retour sur l’hôte de conteneur, utilisez `docker ps` pour obtenir le nom du conteneur, puis `docker rm` pour supprimer le conteneur. Remarque: Remplacez le nom de conteneur indiqué dans cet exemple par le nom de conteneur réel.

Obtenez le nom du conteneur.

```console
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

Arrêtez le conteneur.

```console
docker stop <container name>
```

Supprimez le conteneur.

```console
docker rm -f <container name>
```

## <a name="docker-push"></a>Docker Push

Les images de conteneur Docker peuvent être stockées dans un Registre de conteneur. Une fois stockée dans un Registre, une image peut être récupérée pour être utilisée ultérieurement sur plusieurs hôtes de conteneur. Docker fournit un Registre public pour le stockage des images de conteneur sur [Docker Hub](https://hub.docker.com/).

Dans cet exercice, l’image personnalisée «Hello World» est transférée (pushed) vers votre compte sur Docker Hub.

Tout d’abord, connectez-vous à votre compte Docker en utilisant la `docker login command`.

```console
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

Une fois que vous êtes connecté, vous pouvez transférer (push) l’image de conteneur vers Docker Hub. Pour ce faire, utilisez la commande `docker push`. **Remplacez «user» par votre ID Docker**. 

```console
docker push <user>/iis-dockerfile
```

Comme Docker exécute un push de chaque couche jusqu'à Docker Hub, docker ignore les couches qui existent déjà dans le Hub Docker ou autres registres (couches étrangères).  Par exemple, les versions récentes de Windows Server Core qui sont hébergées dans le Registre de conteneur de Microsoft, ou les couches à partir d’un registre d’entreprise privé, seraient être ignorées et non appuyées sur Docker Hub.

L’image de conteneur peut maintenant être téléchargée à partir de Docker Hub sur n’importe quel hôte de conteneur Windows à l’aide de la commande `docker pull`. Pour ce didacticiel, nous allons supprimer l’image existante et l’extraire (pull) de Docker Hub. 

```console
docker rmi <user>/iis-dockerfile
```

L’exécution de `docker images` montre que l’image a été supprimée.

```console
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
modified-iis              latest              51f1fe8470b3        5 minutes ago       7.69 GB
microsoft/iis             latest              e4525dda8206        3 hours ago         7.61 GB
```

Pour finir, nous pouvons utiliser la commande «docker pull» pour extraire l’image et la remettre sur l’hôte de conteneur. Remplacez «user» par le nom d’utilisateur de votre compte Docker. 

```
docker pull <user>/iis-dockerfile
```

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir comment mettre en package un exemple d’application ASP.NET, consultez les didacticiels de Windows10 dont les liens figurent ci-dessous.

> [!div class="nextstepaction"]
> [Conteneurs sur Windows 10](./quick-start-windows-10.md)

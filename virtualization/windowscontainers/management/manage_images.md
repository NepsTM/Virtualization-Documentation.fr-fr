---
title: Images de conteneur Windows
description: "Créez et gérez des images de conteneur avec des conteneurs Windows."
keywords: docker, conteneurs
author: neilpeterson
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
redirect_url: https://docs.docker.com/v1.8/userguide/dockerimages/
translationtype: Human Translation
ms.sourcegitcommit: 59626096d428072dec098c7817e2d6b39c10e9cf
ms.openlocfilehash: 49e29949fc91533cf1d3ed0aef47e829276772f4

---

# Images de conteneur Windows

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

>Les conteneurs Windows sont gérés avec Docker. La documentation des conteneurs Windows s’ajoute à la documentation disponible sur [docsdocker.com](https://docs.docker.com/).

Les images de conteneur sont utilisées pour déployer des conteneurs. Ces images peuvent inclure des applications et toutes les dépendances d’application. Par exemple, vous pouvez développer une image de conteneur qui a été préconfigurée avec Nano Server, IIS et une application s’exécutant dans IIS. Cette image de conteneur peut alors être stockée dans un Registre de conteneur pour être utilisée plus tard, déployée sur un hôte de conteneur Windows (localement, dans le cloud, ou même dans un service de conteneur) et utilisée également comme base pour une nouvelle image de conteneur.

### Installer une image

Avant de travailler avec des conteneurs Windows, une image de base doit être installée. Les images de base sont disponibles avec Windows Server Core ou Nano Server comme système d’exploitation sous-jacent. Pour plus d’informations sur les configurations prises en charge, consultez la page [Configuration requise pour un conteneur Windows](../deployment/system_requirements.md).

Exécutez la commande suivante pour installer l’image de base Windows Server Core :

```none
docker pull microsoft/windowsservercore
```

Pour installer l’image de base Nano Server, exécutez la commande suivante :

```none
docker pull microsoft/nanoserver
```

### Répertorier des images

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/windowsservercore   latest              02cb7f65d61b        9 weeks ago         7.764 GB
microsoft/nanoserver          latest              3a703c6e97a2        9 weeks ago         969.8 MB
```

### Créer une image

Vous pouvez créer une image de conteneur à partir d’un conteneur existant. Pour ce faire, utilisez la commande `docker commit`. L’exemple suivant crée une image de conteneur nommée « windowsservercoreiis ».

```none
docker commit 475059caef8f windowsservercoreiis
```

### Supprimer une image

Les images de conteneur ne peuvent pas être supprimées si un conteneur, même dans un état arrêté, a une dépendance sur l’image.

Quand vous supprimez une image avec docker, les images peuvent être référencées par ID ou nom d’image.

```none
docker rmi windowsservercoreiis
```

### Dépendance d’image

Pour visualiser les dépendances d’image avec Docker, utilisez la commande `docker history`.

```none
docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Hub Docker

Le Registre du hub Docker contient des images prédéfinies qui peuvent être téléchargées sur un hôte de conteneur. Une fois ces images téléchargées, elles peuvent servir comme base pour les applications de conteneur Windows.

Pour afficher la liste des images disponibles à partir du hub Docker, utilisez la commande `docker search`. Remarque : L’image de système d’exploitation de base Windows Server Core ou Nano Server doit être installée avant l’extraction des images dépendantes à partir du hub Docker.

La plupart de ces images ont une version Windows Server Core et Nano Server. Pour obtenir une version spécifique, ajoutez simplement la balise « :windowsservercore » ou « :nanoserver ». La balise « latest » retourne la version Windows Server Core par défaut, sauf si seule une version Nano Server est disponible.


```none
docker search *

NAME                     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/sample-django  Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35       .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/sample-golang  Go Programming Language installed in a Win...   1                    [OK]
microsoft/sample-httpd   Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis            Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/sample-mongodb MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/sample-mysql   MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-nginx   Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-node    Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-python  Python installed in a Windows Server Core ...   1                    [OK]
microsoft/sample-rails   Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/sample-redis   Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-ruby    Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-sqlite  SQLite installed in a Windows Server Core ...   1                    [OK]
```

### docker pull

Pour télécharger une image à partir du hub Docker, utilisez la commande `docker pull`. Pour plus d’informations, voir [docker pull sur Docker.com](https://docs.docker.com/engine/reference/commandline/pull/).

```none
docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

L’image est désormais visible quand vous exécutez la commande `docker images`.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.14300.1000     6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```

> En cas d’échec de docker pull, vérifiez que les dernières mises à jour cumulatives ont été appliquées à l’hôte de conteneur. La mise à jour TP5 est accessible sur [KB3157663]( https://support.microsoft.com/en-us/kb/3157663).

### docker push

Les images de conteneur peuvent également être chargées sur un registre Docker Hub ou un registre DTR (Docker Trusted Registry). Une fois chargées, ces images peuvent être téléchargés et réutilisées dans différents environnements de conteneurs Windows.

Pour charger une image de conteneur sur Docker Hub, connectez-vous tout d’abord au registre. Pour plus d’informations, voir [docker login sur Docker.com]( https://docs.docker.com/engine/reference/commandline/login/).

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: username
Password:

Login Succeeded
```

Une fois connecté à votre registre Docker Hub ou DTR, utilisez `docker push` pour charger une image de conteneur. L’image de conteneur peut être référencée par nom ou ID. Pour plus d’informations, voir [docker push sur Docker.com]( https://docs.docker.com/engine/reference/commandline/push/).

```none
docker push username/containername

The push refers to a repository [docker.io/username/containername]
b567cea5d325: Pushed
00f57025c723: Pushed
2e05e94480e9: Pushed
63f3aa135163: Pushed
469f4bf35316: Pushed
2946c9dcfc7d: Pushed
7bfd967a5e43: Pushed
f64ea92aaebc: Pushed
4341be770beb: Pushed
fed398573696: Pushed
latest: digest: sha256:ae3a2971628c04d5df32c3bbbfc87c477bb814d5e73e2787900da13228676c4f size: 2410
```

À ce stade, l’image de conteneur est désormais disponible et accessible avec `docker pull`.






<!--HONumber=Sep16_HO2-->



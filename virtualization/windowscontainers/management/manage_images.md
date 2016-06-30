---
title: Images de conteneur Windows
description: "Créez et gérez des images de conteneur avec des conteneurs Windows."
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
translationtype: Human Translation
ms.sourcegitcommit: 3db43b433e7b1a9484d530cf209ea80ef269a307
ms.openlocfilehash: 505cc64fa19fb9fc8c2d5c109830f460f09332dd

---

# Images de conteneur Windows

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

Les images de conteneur sont utilisées pour déployer des conteneurs. Ces images peuvent inclure un système d’exploitation, des applications et toutes les dépendances d’application. Par exemple, vous pouvez développer une image de conteneur qui a été préconfigurée avec Nano Server, IIS et une application s’exécutant dans IIS. Cette image de conteneur peut alors être stockée dans un Registre de conteneur pour être utilisée plus tard, déployée sur un hôte de conteneur Windows (localement, dans le cloud, ou même dans un service de conteneur) et utilisée également comme base pour une nouvelle image de conteneur.

Il existe deux types d’images de conteneur :

**Images de système d’exploitation de base** : elles sont fournies par Microsoft et incluent les composants principaux du système d’exploitation. 

**Images de conteneur** : une image de conteneur personnalisée qui est dérivée d’une image de système d’exploitation de base.

## Images de système d’exploitation de base

### Installer une image

Les images de système d’exploitation du conteneur sont accessibles et installées à l’aide du module PowerShell ContainerImage. Vous devez installer ce module avant de l’utiliser. La commande suivante peut être utilisée pour installer le module. Pour plus d’informations sur l’utilisation du module PowerShell OneGet d’images de conteneur, voir [Fournisseur d’images de conteneur](https://github.com/PowerShell/ContainerProvider). 

```none
Install-PackageProvider ContainerImage -Force
```

Une fois l’installation terminée, il est possible de retourner la liste des images de système d’exploitation de base à l’aide de `Find-ContainerImage`.

```none
Find-ContainerImage

Name                 Version          Source           Summary
----                 -------          ------           -------
NanoServer           10.0.14300.1010  ContainerImag... Container OS Image of Windows Server 2016 Technical...
WindowsServerCore    10.0.14300.1000  ContainerImag... Container OS Image of Windows Server 2016 Technical...
```

Pour télécharger et installer l’image du système d’exploitation de base Nano Server, exécutez la commande suivante. Le paramètre `-version` est facultatif. Sans version d’image de système d’exploitation de base spécifiée, la dernière version est installée.

```none
Install-ContainerImage -Name NanoServer -Version 10.0.14300.1010
```

Cette commande permet de télécharger et d’installer l’image du système d’exploitation de base Windows Server Core. Le paramètre `-version` est facultatif. Sans version d’image de système d’exploitation de base spécifiée, la dernière version est installée.

```none
Install-ContainerImage -Name WindowsServerCore -Version 10.0.14300.1000
```

Utilisez la commande `docker images` pour vérifier que les images ont été installées. 

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     40356b90dc80        2 weeks ago         793.3 MB
windowsservercore   10.0.14304.1000     7837d9445187        2 weeks ago         9.176 GB
```  

Une fois l’installation terminée, vous pouvez également marquer les images avec la balise « latest ». Ces instructions sont détaillées dans la section relative aux balises qui figure ci-dessous.

> Si l’image de système d’exploitation de base est téléchargée, mais ne s’affiche pas lors de l’exécution de `docker images`, redémarrez le service Docker à l’aide de l’applet Panneau de configuration Services ou de la commande 'sc stop docker' suivie de la commande 'sc start docker'

### Marquer des images

Quand vous référencez une image de conteneur par son nom, le moteur Docker recherche la dernière version de l’image. Si la version la plus récente ne peut pas être déterminée, l’erreur suivante est déclenchée.

```none
docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

Une fois que les images de système d’exploitation de base Windows Server Core ou Nano Server sont installées, vous devez marquer leur version à l’aide de l’indicateur 'latest'. Pour ce faire, utilisez la commande `docker tag`. 

Pour plus d’informations sur `docker tag`, voir l’article relatif au [marquage et aux transmissions de type Push et Pull des images sur docker.com](https://docs.docker.com/mac/step_six/). 

```none
docker tag <image id> windowsservercore:latest
```

Quand elle est marquée, la sortie de `docker images` affiche deux versions de la même image, l’une avec la balise de la version de l’image et l’autre avec la balise « latest ». L’image peut désormais être référencée par son nom.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14300.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### Installation hors connexion

Les images de système d’exploitation de base peuvent également être installées sans connexion Internet. Pour ce faire, téléchargez l’image sur un ordinateur avec une connexion Internet, copiez-la sur le système cible, puis importez-la à l’aide de la commande `Install-ContainerOSImages`.

Avant de télécharger l’image de système d’exploitation de base, préparez le système **connecté à Internet** à l’aide du fournisseur d’images de conteneur en exécutant la commande suivante.

```none
Install-PackageProvider ContainerImage -Force
```

Retournez une liste d’images à partir du gestionnaire de package PowerShell OneGet :

```none
Find-ContainerImage
```

Sortie :

```none
Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.14300.1010         Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.14300.1000         Container OS Image of Windows Server 2016 Techn...
```

Pour télécharger une image, utilisez la commande `Save-ContainerImage`.

```none
Save-ContainerImage -Name NanoServer -Path c:\container-image
```

L’image de conteneur téléchargée peut maintenant être copiée sur l’**hôte de conteneur hors connexion** et installée à l’aide de la commande `Install-ContainerOSImage`.

```none
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### Désinstaller une image de système d’exploitation

Les images de système d’exploitation de base peuvent être désinstallées à l’aide de la commande `Uninstall-ContainerOSImage`. L’exemple suivant désinstalle l’image de système d’exploitation de base NanoServer.

```none
Uninstall-ContainerOSImage -FullName CN=Microsoft_NanoServer_10.0.14304.1003
```

## Images de conteneur

### Répertorier des images

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.14300.1000     6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.14300.1010     8572198a60f1        2 weeks ago          0 B
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






<!--HONumber=Jun16_HO4-->



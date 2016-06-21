---
title: Démarrage rapide du déploiement de conteneurs - Images
description: Démarrage rapide du déploiement de conteneurs
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
---

# Images de conteneur sur Windows Server

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

Dans le démarrage rapide de Windows Server précédent, un conteneur Windows était créé à partir d’une image de conteneur préexistante. Cet exercice décrit en détail la création manuelle de vos propres images de conteneur et la création d’images à l’aide d’un fichier Dockerfile.

Ce démarrage rapide est spécifique aux conteneurs Windows Server sur Windows Server 2016. Une documentation de démarrage rapide supplémentaire est disponible dans la table des matières affichée à gauche dans cette page. 

**Conditions préalables :**

- Un système informatique (physique ou virtuel) exécutant [Windows Server 2016 Technical Preview 5](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview).
- Configurez ce système avec la fonctionnalité de conteneur Windows et Docker. Pour obtenir une procédure pas à pas décrivant ces étapes, voir [Conteneurs Windows sur Windows Server](./quick_start_windows_server.md).

## 1. Images de conteneur - Manuelle

Pour une expérience optimale, parcourez cet exercice à partir d’une interface de commande Windows (cmd.exe).

La première étape de la création manuelle d’une image de conteneur consiste à déployer un conteneur. Pour cet exemple, déployez un conteneur IIS à partir de l’image IIS précréée. Une fois le conteneur déployé, vous allez travailler dans une session d’interpréteur de commandes à partir du conteneur. La session interactive est initialisée avec l’indicateur `-it`. Pour obtenir plus de détails sur la commande Docker Run, voir [Docker Run Reference]( https://docs.docker.com/engine/reference/run/) sur Docker.com. 

```none
docker run -it -p 80:80 microsoft/iis:windowsservercore cmd
```

Ensuite, une modification sera apportée au conteneur. Exécutez la commande suivante pour supprimer l’écran de démarrage IIS.

```none
del C:\inetpub\wwwroot\iisstart.htm
```

Exécutez également la commande suivante pour remplacer le site IIS par défaut par un nouveau site statique.

```none
echo "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

À partir d’un autre système, accédez à l’adresse IP de l’hôte du conteneur. Vous voyez maintenant l’application « Hello World ».

![](media/hello.png)

De retour dans le conteneur, quittez la session de conteneur interactive.

```none
exit
```

Le conteneur modifié peut maintenant être capturé dans une nouvelle image de conteneur. Pour ce faire, vous avez besoin du nom du conteneur. Pour le trouver, utilisez la commande `docker ps -a`.

```none
docker ps -a

CONTAINER ID     IMAGE                             COMMAND   CREATED             STATUS   PORTS   NAMES
489b0b447949     microsoft/iis:windowsservercore   "cmd"     About an hour ago   Exited           pedantic_lichterman
```

Pour créer la nouvelle image du conteneur, utilisez la commande `docker commit`. La commande docker commit prend la forme « docker commit nom_conteneur nom_nouvelle_image ». Remarque : Remplacez le nom de conteneur indiqué dans cet exemple par le nom de conteneur réel.

```none
docker commit pedantic_lichterman modified-iis
```

Pour vérifier que la nouvelle image a été créée, utilisez la commande `docker images`.  

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
modified-iis        latest              3e4fdb6ed3bc        About a minute ago   10.17 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago          9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago          9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago          9.344 GB
```

Cette image peut maintenant être déployée. Le conteneur obtenu inclut toutes les modifications capturées.

## 2. Image de conteneur - Fichier Dockerfile

Dans l’exercice précédent, un conteneur a été manuellement créé, modifié, puis capturé dans une nouvelle image de conteneur. Docker inclut une méthode pour automatiser ce processus à l’aide de ce que l’on appelle un « fichier Dockerfile ». Cet exercice permet d’obtenir presque les mêmes résultats que le précédent, mais avec un processus automatisé.

Sur l’hôte de conteneur, créez un répertoire `c:\build` dans lequel vous créez un fichier nommé `Dockerfile`. Remarque : Le fichier ne doit pas avoir d’extension de fichier.

```none
powershell new-item c:\build\Dockerfile -Force
```

Ouvrez le fichier Dockerfile dans le Bloc-notes.

```none
notepad c:\build\Dockerfile
```

Copiez le texte suivant dans le fichier Dockerfile, puis enregistrez ce dernier. Ces commandes indiquent à Docker de créer une image en se servant de `microsoft/iis` comme base. Le fichier Dockerfile exécute ensuite les commandes spécifiées dans l’instruction `RUN`. Dans le cas présent, le fichier index.html est mis à jour avec le nouveau contenu. 

Pour plus d’informations sur les fichiers Dockerfile, voir [Fichiers Dockerfile sur Windows](../docker/manage_windows_dockerfile.md).

```none
FROM microsoft/iis:windowsservercore
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

La commande `docker build` démarre le processus de génération de l’image. Le paramètre `-t` indique au processus de génération de nommer la nouvelle image `iis-dockerfile`.

```none
docker build -t iis-dockerfile c:\Build
```

Une fois l’opération terminée, vous pouvez vérifier que l’image a été créée à l’aide de la commande `docker images`.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

Déployez maintenant un conteneur à l’aide de la commande suivante. 

```none
docker run -d -p 80:80 iis-dockerfile ping -t localhost
```

Une fois le conteneur créé, accédez à l’adresse IP de l’hôte de conteneur. Vous devez voir l’application hello world.

![](media/dockerfile2.png)

De retour sur l’hôte de conteneur, utilisez `docker ps` pour obtenir le nom du conteneur, puis `docker rm` pour supprimer le conteneur. Remarque : Remplacez le nom de conteneur indiqué dans cet exemple par le nom de conteneur réel.

Obtenez le nom du conteneur.

```none
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

Supprimez le conteneur.

```none
docker rm -f cranky_brown
```

## Étapes suivantes

[Conteneurs Windows sur Windows 10](./quick_start_windows_10.md)

<!--HONumber=May16_HO4-->



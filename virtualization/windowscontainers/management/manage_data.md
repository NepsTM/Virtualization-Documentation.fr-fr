---
title: Volumes de données de conteneur
description: Créez et gérez des volumes de données avec des conteneurs Windows.
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: f5998534-917b-453c-b873-2953e58535b1
---

# Volumes de données de conteneur

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

Lors de la création de conteneurs, vous devrez peut-être créer un répertoire de données, ou ajouter un répertoire existant au conteneur. Pour ce faire, vous pouvez ajouter des volumes de données. Les volumes de données sont visibles pour le conteneur et l’hôte de conteneur, et les données peuvent être partagées entre eux. Les volumes de données peuvent également être partagés entre plusieurs conteneurs sur le même hôte de conteneur. Ce document décrit en détail la création, l’examen et la suppression des volumes de données.

## Volumes de données

### Créer un volume de données

Créez un volume de données à l’aide du paramètre `-v` de la commande `docker run`. Par défaut, les nouveaux volumes de données sont stockés sur l’hôte sous « c:\ProgramData\Docker\volumes ».

Cet exemple crée un volume de données nommé « new-data-volume ». Ce volume de données est accessible dans le conteneur en cours d’exécution sur « c:\new-data-volume ».

```none
docker run -it -v c:\new-data-volume windowsservercore cmd
```

Pour plus d’informations sur la création de volumes, voir [Gérer les données dans les conteneurs sur docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#data-volumes).

### Montage d’un répertoire existant

Outre la création d’un volume de données, vous pouvez transmettre un répertoire existant à partir de l’hôte vers le conteneur. Pour ce faire, vous pouvez également utiliser le paramètre `-v` de la commande `docker run`. Tous les fichiers dans le répertoire hôte sont également disponibles dans le conteneur. Tous les fichiers créés par le conteneur dans le volume monté sont disponibles sur l’hôte. Le même répertoire peut être monté sur de nombreux conteneurs. Dans cette configuration, les données peuvent être partagées entre les conteneurs.

Dans cet exemple, le répertoire source, « c:\source », est monté dans un conteneur en tant que « c:\destination ».

```none
docker run -it -v c:\source:c:\destination windowsservercore cmd
```

Pour plus d’informations sur le montage de répertoires hôtes, voir [Gérer les données dans les conteneurs sur docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-directory-as-a-data-volume).

### Monter des fichiers individuels

Un seul fichier peut être monté dans un conteneur en spécifiant explicitement le nom de fichier. Dans cet exemple, le répertoire partagé comporte de nombreux fichiers, mais seul le fichier « config.ini » est disponible à l’intérieur du conteneur. 

```none
docker run -it -v c:\container-share\config.ini windowsservercore cmd
```

Dans le conteneur en cours d’exécution, seul le fichier config.ini est visible.

```none
c:\container-share>dir
 Volume in drive C has no label.
 Volume Serial Number is 7CD5-AC14

 Directory of c:\container-share

04/04/2016  12:53 PM    <DIR>          .
04/04/2016  12:53 PM    <DIR>          ..
04/04/2016  12:53 PM    <SYMLINKD>     config.ini
               0 File(s)              0 bytes
               3 Dir(s)  21,184,208,896 bytes free
```

Pour plus d’informations sur le montage de fichiers individuels, voir [Gérer les données dans les conteneurs sur docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-directory-as-a-data-volume).

### Conteneurs de volumes de données

Les volumes de données peuvent être hérités d’autres conteneurs en cours d’exécution à l’aide du paramètre `--volumes-from` de la commande `docker run`. Un conteneur peut être créé à l’aide de cet héritage, avec l’objectif explicite d’héberger des volumes de données pour des applications en conteneur. 

Cet exemple monte les volumes de données à partir du conteneur « cocky_bell » dans un nouveau conteneur. Une fois que le nouveau conteneur a été démarré, les données de ce volume sont disponibles pour les applications en cours d’exécution dans le conteneur.  

```none
docker run -it --volumes-from cocky_bell windowsservercore cmd
```

Pour plus d’informations sur les conteneurs de données, voir [Gérer les données dans les conteneurs sur docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-file-as-a-data-volume).

### Examiner le volume de données partagées

Les volumes montés peuvent être affichés à l’aide de la commande `docker inspect`.

```none
docker inspect backstabbing_kowalevski
```

Celle-ci retourne des informations sur le conteneur, y compris une section nommée « Mounts », qui contient des données sur les volumes montés comme le répertoire source et de destination.

```none
"Mounts": [
    {
        "Source": "c:\\container-share",
        "Destination": "c:\\data",
        "Mode": "",
        "RW": true,
        "Propagation": ""
}
```

Pour plus d’informations sur l’examen de volumes, voir [Gérer les données dans les conteneurs sur docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#locating-a-volume).



<!--HONumber=May16_HO4-->



---
title: "Volumes de données de conteneur"
description: "Créez et gérez des volumes de données avec des conteneurs Windows."
keywords: docker, conteneurs
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: f5998534-917b-453c-b873-2953e58535b1
translationtype: Human Translation
ms.sourcegitcommit: 08f893b646046d18def65602eb926bc0ea211804
ms.openlocfilehash: a175091c943cf596b2a810245b1b73b8baddb0c6

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

Un fichier individuel ne peut pas être monté dans un conteneur de Windows. L’exécution de la commande suivante n’échoue pas, mais le conteneur résultant ne contient pas le fichier. 

```none
docker run -it -v c:\config\config.ini microsoft/windowsservercore cmd
```

Une solution de contournement est que tout fichier à monter dans un conteneur doit l’être à partir d’un répertoire.

```none
docker run -it -v c:\config:c:\config microsoft/windowsservercore cmd
```

### Monter un lecteur entier

Un lecteur complet peut être monté en utilisant une commande semblable à celle-ci. Veillez à ne pas inclure de barre oblique inverse.

```none
docker run -it -v d: windowsservercore cmd
```

À ce stade, le montage d’une partie du second lecteur ne fonctionne pas. Par exemple, l’opération suivante n’est pas possible.

```none
docker run -it -v d:\source:d:\destination windowsservercore cmd
```

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




<!--HONumber=Sep16_HO1-->



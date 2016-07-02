---
author: neilpeterson
redirect_url: ../quick_start/manage_docker
translationtype: Human Translation
ms.sourcegitcommit: 2b85875eae1dcf1e50162e69c53dbf1ac7463450
ms.openlocfilehash: 8921cbd910bf657ddc4998e4214c1e9f9c3a01e9

---

# Gestion des conteneurs Windows Server

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

Le cycle de vie de conteneur inclut des actions, telles que le démarrage, l’arrêt et la suppression de conteneurs. Quand vous effectuez ces actions, il est possible que vous deviez également récupérer une liste d’images de conteneur, gérer la mise en réseau des conteneurs et limiter les ressources des conteneurs. Ce document décrit en détail les tâches de gestion de conteneurs de base à l’aide de Docker et incorpore également des articles détaillés. 

## Gestion des conteneurs

### Créer un conteneur

Pour créer un conteneur avec Docker, utilisez la commande `docker run`.

```none
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Pour plus d’informations sur la commande docker `run`, voir les [informations de référence sur docker run]( https://docs.docker.com/engine/reference/run/).

### Arrête un conteneur

Pour arrêter un conteneur avec Docker, utilisez la commande `docker stop`.

```none
PS C:\> docker stop tender_panini

tender_panini
```

Cet exemple arrête tous les conteneurs en cours d’exécution avec Docker.

```none
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### Supprimer un conteneur

Pour supprimer un conteneur avec Docker, utilisez la commande `docker rm`.

```none
PS C:\> docker rm prickly_pike

prickly_pike
``` 

Pour supprimer tous les conteneurs avec Docker

```none
PS C:\> docker rm $(docker ps -aq)

dc3e282c064d
2230b0433370
```

Pour plus d’informations sur la commande docker rm, consultez les [informations de référence sur docker rm](https://docs.docker.com/engine/reference/commandline/rm/).



<!--HONumber=Jun16_HO4-->



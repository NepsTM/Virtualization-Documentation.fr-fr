---
title: Conteneurs Windows et Linux sur Windows 10
description: Démarrage rapide du déploiement de conteneurs
keywords: dockers, conteneurs, LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 91031f9394cb3fcb1af6c4813f8805ad6f79bf8c
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681099"
---
# <a name="linux-containers-on-windows-10"></a>Conteneurs Linux sur Windows 10

> [!div class="op_single_selector"]
> - [Conteneurs Linux sur Windows](quick-start-windows-10-linux.md)
> - [Conteneurs Windows sous Windows](quick-start-windows-10.md)

Dans cet exercice, vous allez découvrir comment créer et exécuter des conteneurs Linux sur Windows 10.

Dans ce démarrage rapide, vous allez effectuer les opérations suivantes:

1. Installation de la version de bureau de l’amarrage
2. Exécution d’un conteneur Linux simple utilisant des conteneurs Linux sur Windows (LCOW)

Ce démarrage rapide est spécifique à Windows10. Pour plus d’informations, reportez-vous à la documentation de démarrage rapide supplémentaire disponible dans la table des matières sur le côté gauche de cette page.

## <a name="prerequisites"></a>Prérequis

Veuillez vérifier que vous remplissez les conditions suivantes: <<<<<<< HEAD
- Un système informatique physique exécutant Windows 10 professionnel ou entreprise avec la mise à jour de la version 1709) ou version ultérieure
- Vérifiez que [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) est activé.
=======
- Un système informatique physique exécutant Windows 10 professionnel, Windows 10 entreprise ou Windows Server 2019 version 1809 ou ultérieure
- Vérifiez que [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) est activé.
>>>>>>> origine/maître

***Isolement Hyper-V:*** Les conteneurs Linux sur Windows nécessitent l’isolement Hyper-V sur Windows 10 afin de fournir aux développeurs le noyau Linux approprié pour exécuter le conteneur. Pour plus d’informations sur l’isolement Hyper-V, consultez la page [à propos des conteneurs Windows](../about/index.md) .

## <a name="install-docker-desktop"></a>Installer le Bureau de l’ancrage

Téléchargez la version de bureau de l' [amarrage](https://store.docker.com/editions/community/docker-ce-desktop-windows) et exécutez le programme d’installation (vous devez vous connecter. Créez un compte si vous n’en avez pas encore. [Des instructions d’installation détaillées](https://docs.docker.com/docker-for-windows/install) sont disponibles dans la documentation de Docker.

> Si vous avez déjà installé la version d’Arrimateur, assurez-vous que vous disposez de la version 18,02 ou version ultérieure pour prendre en charge LCOW. Vérifiez en exécutant `docker -v` ou en vérifiant s’il s’agit de l' *ancrage*.

> L’option’fonctionnalités expérimentales’du *démon > des paramètres* de l’ancrage doit être activée pour exécuter des conteneurs LCOW.

## <a name="run-your-first-lcow-container"></a>Exécuter votre premier conteneur LCOW

Pour cet exemple, un conteneur BusyBox est déployé. Tout d’abord, essayez d’exécuter une image de BusyBox’Hello World'.

```console
docker run --rm busybox echo hello_world
```

Notez que cette opération renvoie une erreur lorsque l’ancrage tente de tirer l’image. Cela se produit parce que les Dockeurs requièrent `--platform` une directive via l’indicateur pour vérifier que le système d’exploitation de l’image et de l’hôte est satisfait de manière appropriée. Comme la plateforme par défaut dans le mode conteneur Windows est Windows, `--platform linux` ajoutez un indicateur pour extraire et exécuter le conteneur.

```console
docker run --rm --platform linux busybox echo hello_world
```

Une fois l’image récupérée avec la plateforme indiquée, l' `--platform` indicateur n’est plus nécessaire. Exécutez la commande sans la vérifier.

```console
docker run --rm busybox echo hello_world
```

Exécuter `docker images` pour renvoyer une liste d’images installées. Dans ce cas, les images Windows et Linux.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Bonus: consultez le [billet de blog](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) correspondant de votre Dock sur l’exécution d’LCOW.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez comment créer un exemple d’application](./building-sample-app.md)

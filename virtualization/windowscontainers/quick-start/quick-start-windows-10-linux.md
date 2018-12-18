---
title: Windows et des conteneurs Linux sur Windows 10
description: Démarrage rapide du déploiement de conteneurs
keywords: docker, conteneurs, LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 036e4f80eaa6e7ce2c151d7732e670c0492bc61f
ms.sourcegitcommit: 95cec99aa8e817d3e3cb2163bd62a32d9e8f7181
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2018
ms.locfileid: "8973812"
---
# <a name="linux-containers-on-windows-10"></a>Conteneurs Linux sur Windows 10

> [!div class="op_single_selector"]
> - [Conteneurs Linux sur Windows](quick-start-windows-10-linux.md)
> - [Conteneurs Windows sur Windows](quick-start-windows-10.md)

L’exercice guideront à travers la création et l’exécution des conteneurs Linux sur Windows 10.

Dans ce démarrage rapide vous allez accomplir:

1. Installé Docker pour Windows
2. Exécuter un conteneur Linux simple à l’aide de conteneurs Linux sur Windows (LCOW)

Ce démarrage rapide est spécifique à Windows10. Vous trouverez la documentation de démarrage rapide supplémentaire dans la table des matières sur le côté gauche de cette page.

## <a name="prerequisites"></a>Conditions préalables

Vérifiez que vous respectez les exigences suivantes:
- Un système d’ordinateur physique exécutant Windows 10 Professionnel ou entreprise avec Fall Creators Update (version 1709) ou version ultérieure
- Assurez-vous que [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) est activé.

***Isolation Hyper-V:*** Conteneurs Linux sur Windows requièrent l’isolation Hyper-V sur Windows 10 afin de fournir aux développeurs le noyau Linux approprié pour exécuter le conteneur. Plus à propos d’Hyper-V isolation sont accessibles sur la page à [propos des conteneurs Windows](../about/index.md) .

## <a name="install-docker-for-windows"></a>Installer Docker pour Windows

Téléchargez [Docker pour Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) et exécutez le programme d’installation (vous devrez ouvrir une session. Créez un compte si vous n’avez pas déjà). [Des instructions d’installation détaillées](https://docs.docker.com/docker-for-windows/install) sont disponibles dans la documentation de Docker.

> Si vous avez déjà installé de Docker, vérifiez que vous disposez 18.02 ou version ultérieure pour prendre en charge LCOW. Vérifier en exécutant `docker -v` ou la vérification *Sur Docker*.

> Les «fonctions expérimentales' option dans *Docker Paramètres > démon* doit être activé pour exécuter des conteneurs LCOW.

## <a name="run-your-first-lcow-container"></a>Exécuter votre premier conteneur LCOW

Pour cet exemple, un conteneur BusyBox sera déployé. Tout d’abord, essayez d’exécuter une image «Hello World» BusyBox.

```console
docker run --rm busybox echo hello_world
```

Notez que cela retourne une erreur lorsque Docker tente d’extraire l’image. Cela se produit car Dockers nécessite une directive via le `--platform` indicateur pour confirmer que le système d’exploitation hôte et les images sont correctement mis en correspondance. Dans la mesure où la plateforme par défaut en mode de conteneur Windows est Windows, ajoutez un `--platform linux` indicateur pour extraire et exécuter le conteneur.

```console
docker run --rm --platform linux busybox echo hello_world
```

Une fois que l’image a été retirée avec la plateforme indiquée, le `--platform` indicateur n’est plus nécessaire. Exécutez la commande sans ce dernier pour effectuer ce test.

```console
docker run --rm busybox echo hello_world
```

Exécutez `docker images` pour retourner une liste d’images installées. Dans ce cas, les images à la fois Windows et Linux.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Bonus: Consultez correspondante de Docker [billet de blog](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) sur LCOW en cours d’exécution.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez comment créer un exemple d’application](./building-sample-app.md)
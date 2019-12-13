---
title: Conteneurs Windows et Linux sur Windows 10
description: Démarrage rapide du déploiement de conteneurs
keywords: ancrage, conteneurs, LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909579"
---
# <a name="linux-containers-on-windows-10"></a>Conteneurs Linux sur Windows 10

L’exercice vous guide dans la création et l’exécution de conteneurs Linux sur Windows 10.

Dans ce démarrage rapide, vous allez effectuer les opérations suivantes :

1. Installation du Bureau de station d’accueil
2. Exécution d’un conteneur Linux simple à l’aide de conteneurs Linux sur Windows (LCOW)

Ce démarrage rapide est spécifique à Windows 10. Vous trouverez une documentation de démarrage rapide supplémentaire dans la table des matières sur le côté gauche de cette page.

## <a name="prerequisites"></a>Conditions préalables

Assurez-vous que vous remplissez les conditions suivantes :
- Un système d’ordinateur physique exécutant Windows 10 professionnel, Windows 10 entreprise ou Windows Server 2019 version 1809 ou ultérieure
- Vérifiez que [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) est activé.

***Isolation Hyper-V :*** Les conteneurs Linux sur Windows nécessitent l’isolation Hyper-V sur Windows 10 afin de fournir aux développeurs le noyau Linux approprié pour exécuter le conteneur. Pour plus d’informations sur l’isolation Hyper-V, consultez la page [à propos des conteneurs Windows](../about/index.md) .

## <a name="install-docker-desktop"></a>Installer le Bureau de l’ancrage

Téléchargez le Bureau de l' [ancrage](https://store.docker.com/editions/community/docker-ce-desktop-windows) et exécutez le programme d’installation (vous devez vous connecter. Créez un compte si vous n’en avez pas déjà un. [Des instructions d’installation détaillées](https://docs.docker.com/docker-for-windows/install) sont disponibles dans la documentation de Docker.

> Si vous avez déjà installé la station d’accueil, assurez-vous que vous disposez de la version 18,02 ou ultérieure pour prendre en charge LCOW. Vérifiez en exécutant `docker -v` ou en vérifiant l' *ancrage*.

> L’option « fonctionnalités expérimentales » dans les paramètres de l' *arrimeur > démon* doit être activée pour exécuter des conteneurs LCOW.

## <a name="run-your-first-lcow-container"></a>Exécuter votre premier conteneur LCOW

Pour cet exemple, un conteneur BusyBox sera déployé. Tout d’abord, essayez d’exécuter une image BusyBox’Hello World'.

```console
docker run --rm busybox echo hello_world
```

Notez que cette opération renvoie une erreur lorsque l’ancrage tente d’extraire l’image. Cela est dû au fait que les Ancreurs requièrent une directive via l’indicateur `--platform` pour vérifier que l’image et le système d’exploitation hôte sont correctement mis en correspondance. Étant donné que la plateforme par défaut en mode conteneur Windows est Windows, ajoutez un indicateur `--platform linux` pour extraire et exécuter le conteneur.

```console
docker run --rm --platform linux busybox echo hello_world
```

Une fois que l’image a été extraite avec la plateforme indiquée, l’indicateur `--platform` n’est plus nécessaire. Exécutez la commande sans la tester.

```console
docker run --rm busybox echo hello_world
```

Exécutez `docker images` pour retourner une liste d’images installées. Dans ce cas, les images Windows et Linux.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Bonus : consultez le billet de [blog](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) correspondant de l’arrimeur sur l’exécution de LCOW.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez comment créer un exemple d’application](./building-sample-app.md)

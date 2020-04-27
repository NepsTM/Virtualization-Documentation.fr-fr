---
title: Conteneurs Windows et Linux sur Windows 10
description: Démarrage rapide du déploiement de conteneurs
keywords: docker, conteneurs, LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/24/2020
ms.locfileid: "74909579"
---
# <a name="linux-containers-on-windows-10"></a>Conteneurs Linux sur Windows 10

Cet exercice vous permet de créer et d'exécuter des conteneurs Linux sur Windows 10.

Dans ce démarrage rapide, vous allez effectuer ce qui suit :

1. Installation de Docker Desktop
2. Exécution d’un conteneur Linux simple à l’aide de conteneurs Linux sur Windows (LCOW)

Ce démarrage rapide est spécifique à Windows 10. Une documentation supplémentaire du démarrage rapide est disponible dans la table des matières affichée sur la gauche de cette page.

## <a name="prerequisites"></a>Prérequis

Assurez-vous que vous remplissez les conditions suivantes :
- Un ordinateur physique exécutant Windows 10 Professionnel, Windows 10 Entreprise ou Windows Server 2019 version 1809 ou ultérieure
- Assurez-vous que [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) est activé.

***Isolation Hyper-V :*** Les conteneurs Linux sur Windows requièrent l’isolation Hyper-V sur Windows 10 afin de fournir aux développeurs le noyau Linux qui convient pour exécuter le conteneur. Pour plus d’informations sur l’isolation Hyper-V, consultez la page [À propos des conteneurs Windows](../about/index.md).

## <a name="install-docker-desktop"></a>Installer Docker Desktop

Téléchargez [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows), puis exécutez le programme d’installation. (Vous serez invité à vous connecter. Créez un compte si vous n'en avez pas déjà un). [Des instructions d’installation détaillées](https://docs.docker.com/docker-for-windows/install) sont disponibles dans la documentation de Docker.

> Si vous avez déjà installé Docker, assurez-vous que vous disposez de la version 18.02 ou ultérieure pour prendre en charge LCOW. Pour ce faire, exécutez `docker -v` ou consultez *À propos de Docker*.

> L’option « Fonctionnalités expérimentales » dans *Paramètres Docker >* Démon doit être activée pour exécuter des conteneurs LCOW.

## <a name="run-your-first-lcow-container"></a>Exécuter votre premier conteneur LCOW

Pour cet exemple, un conteneur BusyBox sera déployé. Dans un premier temps, essayez d’exécuter une image BusyBox « Hello World ».

```console
docker run --rm busybox echo hello_world
```

Notez que cette opération renvoie une erreur lorsque Docker tente d’extraire l’image. Cela est dû au fait que Docker requiert une directive via l’indicateur `--platform` afin de vérifier que l’image et le système d’exploitation hôte correspondent. La plateforme par défaut du mode de conteneur Windows étant Windows, ajoutez un indicateur `--platform linux` pour extraire et exécuter le conteneur.

```console
docker run --rm --platform linux busybox echo hello_world
```

Une fois l’image extraite avec la plateforme indiquée, l’indicateur `--platform` n’est plus nécessaire. Exécutez la commande sans cet indicateur pour le tester.

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
> Bonus : Consultez le [billet de blog](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) Docker correspondant sur l’exécution de LCOW.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Apprendre à créer un exemple d’application](./building-sample-app.md)

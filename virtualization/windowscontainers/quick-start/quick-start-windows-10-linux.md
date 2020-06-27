---
title: Conteneurs Windows et Linux sur Windows 10
description: Démarrage rapide du déploiement de conteneurs
keywords: docker, conteneurs, LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: tutorial
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: df02dada3e5cf759f003999b38270dd1bf3131fe
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192186"
---
# <a name="linux-containers-on-windows-10"></a>Conteneurs Linux sur Windows 10

Cet exercice vous permet de créer et d'exécuter des conteneurs Linux sur Windows 10.

Dans ce démarrage rapide, vous allez effectuer ce qui suit :

1. Installation de Docker Desktop
2. Exécution d’un conteneur Linux simple

Ce démarrage rapide est spécifique à Windows 10. Une documentation supplémentaire du démarrage rapide est disponible dans la table des matières affichée sur la gauche de cette page.

## <a name="prerequisites"></a>Prérequis

Assurez-vous que vous remplissez les conditions suivantes :
- Un ordinateur physique exécutant Windows 10 Professionnel, Windows 10 Entreprise ou Windows Server 2019 version 1809 ou ultérieure
- Assurez-vous que [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) est activé.

## <a name="install-docker-desktop"></a>Installer Docker Desktop

Téléchargez [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows), puis exécutez le programme d’installation. (Vous serez invité à vous connecter. Créez un compte si vous n'en avez pas déjà un). [Des instructions d’installation détaillées](https://docs.docker.com/docker-for-windows/install) sont disponibles dans la documentation de Docker.

## <a name="run-your-first-linux-container"></a>Exécuter votre premier conteneur Linux

Pour exécuter des conteneurs Linux, vous devez vous assurer que Docker cible le démon approprié. Vous pouvez activer cette option en sélectionnant `Switch to Linux Containers` dans le menu Action après avoir cliqué sur l’icône de la baleine Docker dans la barre d’état système. Si vous voyez `Switch to Windows Containers`, cela signifie que vous ciblez déjà le démon Linux.

![Menu de la barre d'état système de Docker affichant la commande « Switch to Windows containers ».](./media/switchDaemon.png)

Une fois que vous avez confirmé que vous ciblez le bon démon, exécutez le conteneur avec la commande suivante :

```console
docker run --rm busybox echo hello_world
```

Le conteneur doit s’exécuter, afficher "hello_world", puis se fermer.

Lorsque vous interrogez `docker images`, vous devez voir l’image conteneur Linux que vous venez juste de tirer (pull) et d’exécuter :

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Apprendre à créer un exemple d’application](./building-sample-app.md)

---
title: Utiliser des conteneurs avec le programme Windows Insider
description: Découvrez comment prendre en main les conteneurs Windows avec le programme Windows Insider
keywords: ancrage, conteneurs, Insider, Windows
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909889"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Utiliser des conteneurs avec le programme Windows Insider

Cet exercice va vous guider tout au long du déploiement et de l’utilisation de la fonctionnalité de conteneur Windows de la dernière build Insider de Windows Server du programme Windows Insider Preview. Au cours de cet exercice, vous allez installer le rôle de conteneur et déployer une version d’évaluation des images de système d’exploitation de base. Si vous voulez vous familiariser avec les conteneurs, vous trouverez des informations dans la rubrique [À propos des conteneurs](../about/index.md).

## <a name="join-the-windows-insider-program"></a>Rejoignez le Programme Windows Insider

Pour exécuter la version Insider des conteneurs Windows, vous devez disposer d’un hôte exécutant la dernière version de Windows Server à partir du programme Windows Insider et/ou de la dernière version de Windows 10 du programme Windows Insider. Rejoignez le [programme Windows Insider](https://insider.windows.com/GettingStarted) et passez en revue les conditions d’utilisation.

> [!IMPORTANT]
> Vous devez utiliser une version de Windows Server à partir du programme Windows Server Insider Preview ou une version de Windows 10 du programme Windows Insider Preview pour utiliser l’image de base décrite ci-dessous. Si vous n’utilisez pas l’une de ces builds, la création d’un conteneur échouera lors de l’utilisation de ces images de base.

## <a name="install-docker"></a>Installer Docker

Si vous n’avez pas déjà installé vous-même, suivez le Guide de [prise en main](../quick-start/set-up-environment.md) pour installer la station d’accueil.

## <a name="pull-an-insider-container-image"></a>Extraire une image de conteneur Insider

En faisant partie du programme Windows Insider, vous pouvez utiliser nos dernières builds pour les images de base.

Pour extraire l’image de base Insider Nano Server, exécutez la commande suivante :

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Exécutez la commande suivante pour extraire l’image de base Insider Windows Server Core :

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

Les images de base « Windows » et « IoTCore » disposent également d’une version d’Insider qui est disponible pour l’extraction. Vous pouvez en savoir plus sur les images de base disponibles dans le document [images de base du conteneur](../manage-containers/container-base-images.md) .

> [!IMPORTANT]
> Veuillez lire le [CLUF](../images-eula.md ) de l’image du système d’exploitation des conteneurs Windows et les [conditions d’utilisation du](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)programme Windows Insider.

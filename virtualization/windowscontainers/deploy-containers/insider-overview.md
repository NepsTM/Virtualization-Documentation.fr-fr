---
title: Utiliser des conteneurs avec le programme Windows Insider
description: Découvrez comment commencer à utiliser les conteneurs Windows avec le programme Windows Insider
keywords: arrimeur, conteneurs, Insider, fenêtres
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 6080b2c5053720490d374f6fb0daa870d5ddd4e8
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/25/2019
ms.locfileid: "10257793"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Utiliser des conteneurs avec le programme Windows Insider

Cet exercice va vous guider tout au long du déploiement et de l’utilisation de la fonctionnalité de conteneur Windows de la dernière build Insider de WindowsServer du programme WindowsInsiderPreview. Au cours de cet exercice, vous allez installer le rôle de conteneur et déployer une version d’évaluation des images de système d’exploitation de base. Si vous voulez vous familiariser avec les conteneurs, vous trouverez des informations dans la rubrique [À propos des conteneurs](../about/index.md).

## <a name="join-the-windows-insider-program"></a>Rejoindre le Programme WindowsInsider

Pour exécuter la version Insider des conteneurs Windows, vous devez disposer d’un hôte exécutant la dernière version de Windows Server à partir du programme Windows Insider et/ou de la dernière version de Windows 10 à partir du programme Windows Insider. Rejoignez le [programme Windows Insider](https://insider.windows.com/GettingStarted) et consultez les conditions d’utilisation.

> [!IMPORTANT]
> Vous devez utiliser une version de Windows Server du programme Windows Server Insider Preview ou une version de Windows 10 à partir du programme Windows Insider Preview pour utiliser l’image de base décrite ci-dessous. Si vous n’utilisez pas l’une de ces builds, la création d’un conteneur échouera lors de l’utilisation de ces images de base.

## <a name="install-docker"></a>Installer Docker

Si vous n’avez pas encore installé l’ancrage, suivez le Guide de mise en [route](../quick-start/set-up-environment.md) pour installer l’ancrage.

## <a name="pull-an-insider-container-image"></a>Extraire une image de conteneur Insider

En faisant partie du programme Windows Insider, vous pouvez utiliser nos dernières builds pour les images de base.

Pour extraire l’image de base Insider NanoServer, exécutez la commande suivante:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Exécutez la commande suivante pour extraire l’image de base Insider WindowsServerCore:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

Les images de base «Windows» et «IoTCore» possèdent également une version Insider disponible pour l’extraction. Vous pouvez en savoir plus sur les images de base Insider disponibles dans le document des [images de base du conteneur](../manage-containers/container-base-images.md) .

> [!IMPORTANT]
> Prenez connaissance de [l’image du](../images-eula.md ) système d’exploitation conteneurs Windows et des [conditions d’utilisation du](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)programme Windows Insider.

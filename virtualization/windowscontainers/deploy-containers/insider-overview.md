---
title: Utiliser des conteneurs avec le programme Windows Insider
description: Découvrez comment commencer à utiliser les conteneurs Windows avec le programme Windows Insider
keywords: arrimeur, conteneurs, Insider, fenêtres
author: cwilhit
ms.openlocfilehash: 137209a66c3d0b907003498fe78a04a57a140130
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254391"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Utiliser des conteneurs avec le programme Windows Insider

Cet exercice va vous guider tout au long du déploiement et de l’utilisation de la fonctionnalité de conteneur Windows de la dernière build Insider de WindowsServer du programme WindowsInsiderPreview. Au cours de cet exercice, vous allez installer le rôle de conteneur et déployer une version d’évaluation des images de système d’exploitation de base. Si vous voulez vous familiariser avec les conteneurs, vous trouverez des informations dans la rubrique [À propos des conteneurs](../about/index.md).

> [!NOTE]
> Ce contenu est spécifique aux conteneurs Windows Server du programme Windows Server Insider preview. Pour obtenir des instructions sur l’utilisation des conteneurs Windows, consultez le Guide de mise en [route](../quick-start/set-up-environment.md) .

## <a name="join-the-windows-insider-program"></a>Rejoindre le Programme WindowsInsider

Pour exécuter la version Insider des conteneurs Windows, vous devez disposer d’un hôte exécutant la dernière version de Windows Server à partir du programme Windows Insider et/ou de la dernière version de Windows 10 à partir du programme Windows Insider. Rejoignez le [programme Windows Insider](https://insider.windows.com/GettingStarted) et consultez les conditions d’utilisation.

> [!IMPORTANT]
> Vous devez utiliser une version de Windows Server du programme Windows Server Insider Preview ou une version de Windows 10 à partir du programme Windows Insider Preview pour utiliser l’image de base décrite ci-dessous. Si vous n’utilisez pas l’une de ces builds, la création d’un conteneur échouera lors de l’utilisation de ces images de base.

## <a name="install-docker"></a>Installer Docker

<!-- start tab view -->
# [<a name="windows-server-insider"></a>Windows Server Insider](#tab/Windows-Server-Insider)

Pour installer Docker EE, nous allons utiliser le module PowerShell de fournisseur OneGet. Le fournisseur active la fonctionnalité de conteneurs sur votre ordinateur et installe Docker EE. Cette opération nécessite un redémarrage. Ouvrez une session PowerShell avec élévation de privilèges, puis exécutez les commandes suivantes.

> [!NOTE]
> L’installation de docker EE avec les builds Windows Server Insider nécessite un fournisseur de services de OneGet différent de celui utilisé pour les builds non-Insider. Si Docker EE et le fournisseur OneGet DockerMsftProvider sont déjà installés, supprimez-les avant de continuer.

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

Installez le module OneGet PowerShell pour une utilisation avec les builds Windows Insider.

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

Utilisez OneGet pour installer la dernière version de Docker EE Preview.

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

Une fois l’installation terminée, redémarrez l’ordinateur.

```powershell
Restart-Computer -Force
```

# [<a name="windows-10-insider"></a>Windows 10 Insider](#tab/Windows-10-Insider)

Sur Windows 10 Insider, le bord de l’amarrage est installé via le même programme d’installation que le Bureau de l’ordinateur de bureau. Téléchargez la version de bureau de l' [amarrage](https://store.docker.com/editions/community/docker-ce-desktop-windows) et exécutez le programme d’installation. Vous serez invité à vous connecter. Créez un compte si vous n’en avez pas encore. Des instructions d’installation plus détaillées sont disponibles dans la documentation de l' [ancrage](https://docs.docker.com/docker-for-windows/install).

Après l’installation, ouvrez les paramètres de l’amarrage et basculez vers le canal «Edge».

![](./media/docker-edge-instruction.png)

---
<!-- stop tab view -->

## <a name="pull-an-insider-container-image"></a>Extraire une image de conteneur Insider

Avant d’utiliser des conteneurs Windows, une image de base doit être installée. En faisant partie du programme Windows Insider, vous pouvez utiliser nos dernières builds pour les images de base. Vous pouvez en savoir plus sur les images de base disponibles dans le document [images de base du conteneur](../manage-containers/container-base-images.md) .

Pour extraire l’image de base Insider NanoServer, exécutez la commande suivante:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Exécutez la commande suivante pour extraire l’image de base Insider WindowsServerCore:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> Prenez connaissance de [l’image du](../images-eula.md ) système d’exploitation conteneurs Windows et des [conditions d’utilisation du](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)programme Windows Insider.

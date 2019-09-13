---
title: Conteneurs Windows et Linux sur Windows 10
description: Démarrage rapide du déploiement de conteneurs
keywords: dockers, conteneurs, LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 5f0922a1ee2588b6e5a06091fe34e07ceadf89cb
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129364"
---
# <a name="get-started-configure-your-environment-for-containers"></a>Commencer: configurer votre environnement pour les conteneurs

Ce démarrage rapide montre comment:

> [!div class="checklist"]
> * Configurer votre environnement pour les conteneurs
> * Exécuter votre première image de conteneur
> * Conteneur d’une application simple .NET Core

## <a name="prerequisites"></a>Prérequis

<!-- start tab view -->
# [<a name="windows-server"></a>WindowsServer](#tab/Windows-Server)

Veuillez vérifier que vous remplissez les conditions suivantes:

- Un système informatique (physique ou virtuel) exécutant Windows Server 2016 ou une version ultérieure.

> [!NOTE]
> Si vous utilisez Windows Server 2019 Insider Preview, effectuez une mise à jour vers la version d’évaluation de Windows [server 2019](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ).

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 professionnel et entreprise](#tab/Windows-10-Client)

Veuillez vérifier que vous remplissez les conditions suivantes:

- Un système informatique physique exécutant Windows 10 professionnel ou entreprise avec la mise à jour anniversaire (version 1607) ou une version ultérieure.
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) doit être activé.

> [!NOTE]
>  À compter de la mise à jour de Windows 10 octobre 2018, nous n’avons plus de empêche les utilisateurs d’exécuter un conteneur Windows en mode d’isolation de processus sur Windows 10 entreprise ou professionnel à des fins de développement et de test. Pour en savoir plus, voir le [Forum aux questions](../about/faq.md) . 
> 
> Les conteneurs Windows Server utilisent l’isolation Hyper-V par défaut sur Windows 10 afin de fournir aux développeurs la même version de noyau et la même configuration qui seront utilisées en production. Pour plus d’informations sur l’isolement Hyper-V, voir la section [concepts](../manage-containers/hyperv-container.md) de nos documents.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Installer Docker

Le docker est le chaîne native définitif pour l’utilisation de conteneurs Windows. Le dockeur propose une interface de l’interface utilisateur pour gérer les conteneurs sur un hôte donné, créer des conteneurs, supprimer des conteneurs, etc. Pour en savoir plus, consultez la rubrique [concepts](../manage-containers/configure-docker-daemon.md) de nos documents.

<!-- start tab view -->
# [<a name="windows-server"></a>WindowsServer](#tab/Windows-Server)

Sur Windows Server, docker est installé par le biais d’un [module PowerShell du fournisseur OneGet](https://github.com/oneget/oneget) publié par Microsoft appelé [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider). Ce fournisseur:

- active la fonctionnalité conteneurs sur votre ordinateur
- installe le moteur et le client de l’ancrage sur votre ordinateur.

Pour installer l’ancrage, ouvrez une session PowerShell avec élévation de privilèges et installez le fournisseur de services d’amarrage-Microsoft PackageManagement à partir de la [Galerie PowerShell](https://www.powershellgallery.com/packages/DockerMsftProvider).

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Ensuite, utilisez le module de PowerShell PackageManagement pour installer la version la plus récente de dock.

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Quand PowerShell vous demande si la source du package «DockerDefault» doit être approuvée, tapez`A` pour poursuivre l’installation. Une fois l’installation terminée, vous devez redémarrer l’ordinateur.

```powershell
Restart-Computer -Force
```

> [!TIP]
> Si vous voulez mettre à jour l’ancrage ultérieurement:
>  - Vérifiez la version installée avec `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Trouvez la version actuelle avec `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Lorsque vous êtes prêt, procédez à la mise à niveau `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, suivie de `Start-Service Docker`

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 professionnel et entreprise](#tab/Windows-10-Client)

Sur Windows 10 professionnel et entreprise, l’amarrage est installé par le biais d’un programme d’installation classique. Téléchargez la version de bureau de l' [amarrage](https://store.docker.com/editions/community/docker-ce-desktop-windows) et exécutez le programme d’installation. Vous serez invité à vous connecter. Créez un compte si vous n’en avez pas encore. Des instructions d’installation plus détaillées sont disponibles dans la documentation de l' [ancrage](https://docs.docker.com/docker-for-windows/install).

Après l’installation, l’option Bureau d’Arrimement utilise par défaut des conteneurs Linux. Basculez vers les conteneurs Windows à l’aide du menu de la barre d’état système ou en exécutant la commande suivante dans une invite PowerShell:

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>Étapes suivantes

À présent que votre environnement est correctement configuré, suivez le lien ci-dessous pour découvrir comment extraire et exécuter un conteneur.

> [!div class="nextstepaction"]
> [Exécuter votre premier conteneur](./run-your-first-container.md)

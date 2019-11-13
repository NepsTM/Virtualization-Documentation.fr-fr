---
title: Conteneurs Windows et Linux sur Windows 10
description: Configurez Windows 10 ou Windows Server pour les conteneurs, puis passez à l’exécution de votre première image de conteneur.
keywords: dockers, conteneurs, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288117"
---
# <a name="get-started-prep-windows-for-containers"></a>Commencer: préparer Windows pour les conteneurs

Ce didacticiel décrit comment:

- Configurer Windows 10 ou Windows Server pour les conteneurs
- Exécuter votre première image de conteneur
- Conteneur d’une application simple .NET Core

## <a name="prerequisites"></a>Prérequis

<!-- start tab view -->
# [<a name="windows-server"></a>WindowsServer](#tab/Windows-Server)

Pour exécuter des conteneurs sur Windows Server, vous avez besoin d’un serveur physique ou d’une machine virtuelle exécutant Windows Server (canal semi-annuel), Windows Server 2019 ou Windows Server 2016.

Pour le test, vous pouvez télécharger une copie de la version d’évaluation de Windows [server 2019](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ) ou [Windows Server Insider Preview](https://insider.windows.com/for-business-getting-started-server/).

# [<a name="windows-10"></a>Windows10](#tab/Windows-10-Client)

Pour exécuter des conteneurs sur Windows 10, vous devez disposer des éléments suivants:

- Un système informatique physique exécutant Windows 10 professionnel ou entreprise avec la mise à jour anniversaire (version 1607) ou une version ultérieure.
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) doit être activé.

> [!NOTE]
>  À compter de la mise à jour de Windows 10 octobre 2018, nous n’avons plus de empêche les utilisateurs d’exécuter un conteneur Windows en mode d’isolation de processus sur Windows 10 entreprise ou professionnel à des fins de développement et de test. Pour en savoir plus, voir le [Forum aux questions](../about/faq.md) . 
> 
> Les conteneurs Windows Server utilisent l’isolation Hyper-V par défaut sur Windows 10 afin de fournir aux développeurs la même version de noyau et la même configuration qui seront utilisées en production. Pour plus d’informations sur l’isolement Hyper-V, voir la section [concepts](../manage-containers/hyperv-container.md) de nos documents.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Installer Docker

La première étape consiste à installer docker, qui est requis pour utiliser les conteneurs Windows. Le docker fournit un environnement d’exécution standard pour les conteneurs, avec une API courante et une interface de ligne de commande.

Pour plus d’informations sur la configuration, voir [moteur de l’ancrage sur Windows](../manage-docker/configure-docker-daemon.md).

<!-- start tab view -->
# [<a name="windows-server"></a>WindowsServer](#tab/Windows-Server)

Pour installer l’ancrage sur Windows Server, vous pouvez utiliser un [module PowerShell du fournisseur OneGet](https://github.com/oneget/oneget) publié par Microsoft appelé [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider). Ce fournisseur active la fonctionnalité conteneurs dans Windows et installe le moteur et le client de l’ancrage. Voici comment procéder:

1. Ouvrez une session PowerShell avec élévation de privilèges et installez le fournisseur de service d’amarrage-Microsoft PackageManagement à partir de la [Galerie PowerShell](https://www.powershellgallery.com/packages/DockerMsftProvider).

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   Si vous êtes invité à installer le fournisseur NuGet, tapez `Y` -le également pour l’installer.

2. Utilisez le module pour PowerShell PackageManagement pour installer la version la plus récente de dock.

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   Quand PowerShell vous demande si la source du package «DockerDefault» doit être approuvée, tapez`A` pour poursuivre l’installation.
3. Une fois l’installation terminée, redémarrez l’ordinateur.

   ```powershell
   Restart-Computer -Force
   ```

Si vous voulez mettre à jour l’ancrage ultérieurement:

- Vérifiez la version installée avec `Get-Package -Name Docker -ProviderName DockerMsftProvider`
- Trouvez la version actuelle avec `Find-Package -Name Docker -ProviderName DockerMsftProvider`
- Lorsque vous êtes prêt, procédez à la mise à niveau `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, suivie de `Start-Service Docker`

# [<a name="windows-10"></a>Windows10](#tab/Windows-10-Client)

Vous pouvez installer l’ancrage sur les éditions Windows 10 Professional et Enterprise en procédant comme suit. 

1. Téléchargez et installez la version de bureau de la [station d’accueil](https://store.docker.com/editions/community/docker-ce-desktop-windows)et créez un compte d’ancrage gratuit si ce n’est déjà fait. Pour plus d’informations, consultez la documentation de l' [ancrage](https://docs.docker.com/docker-for-windows/install).

2. Lors de l’installation, définissez le type de conteneur par défaut sur conteneurs Windows. Pour basculer une fois l’installation terminée, vous pouvez utiliser l’élément de l’ancrage dans la barre d’état système de Windows (comme illustré ci-dessous) ou la commande suivante dans une invite PowerShell:

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![Menu de la barre d’état système d’amarrage montrant la commande «basculer vers les conteneurs Windows».](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>Étapes suivantes

À présent que votre environnement est correctement configuré, suivez le lien ci-dessous pour découvrir comment exécuter un conteneur.

> [!div class="nextstepaction"]
> [Exécuter votre premier conteneur](./run-your-first-container.md)

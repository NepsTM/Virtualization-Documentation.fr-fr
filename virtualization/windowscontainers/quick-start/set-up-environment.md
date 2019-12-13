---
title: Conteneurs Windows et Linux sur Windows 10
description: Configurez Windows 10 ou Windows Server pour les conteneurs, puis passez à l’exécution de votre première image de conteneur.
keywords: ancrage, conteneurs, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909559"
---
# <a name="get-started-prep-windows-for-containers"></a>Prise en main : préparer Windows pour les conteneurs

Ce didacticiel explique comment :

- Configurer Windows 10 ou Windows Server pour les conteneurs
- Exécuter votre première image de conteneur
- Conteneur d’une application .NET Core simple

## <a name="prerequisites"></a>Conditions préalables

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

Pour exécuter des conteneurs sur Windows Server, vous avez besoin d’un serveur physique ou d’une machine virtuelle exécutant Windows Server (canal semi-annuel), Windows Server 2019 ou Windows Server 2016.

Pour le test, vous pouvez télécharger une copie de Windows [server 2019 Evaluation](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ) ou une version [préliminaire de Windows Server Insider](https://insider.windows.com/for-business-getting-started-server/).

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

Pour exécuter des conteneurs sur Windows 10, vous avez besoin des éléments suivants :

- Un système d’ordinateur physique exécutant Windows 10 professionnel ou entreprise avec une mise à jour anniversaire (version 1607) ou ultérieure.
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) doit être activé.

> [!NOTE]
>  À compter de la mise à jour 2018 de Windows 10 octobre, nous n’interdisent plus les utilisateurs d’exécuter un conteneur Windows en mode d’isolation de processus sur Windows 10 entreprise ou professionnel à des fins de développement et de test. Pour en savoir plus, consultez le [Forum aux questions](../about/faq.md) . 
> 
> Les conteneurs Windows Server utilisent l’isolation Hyper-V par défaut sur Windows 10 afin de fournir aux développeurs la même version de noyau et la même configuration que celle qui sera utilisée en production. En savoir plus sur l’isolation Hyper-V dans la zone [concepts](../manage-containers/hyperv-container.md) de nos documents.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Installer Docker

La première étape consiste à installer l’arrimeur, qui est requis pour utiliser des conteneurs Windows. Dockr fournit un environnement d’exécution standard pour les conteneurs, avec une API commune et une interface de ligne de commande (CLI).

Pour plus d’informations sur la configuration, consultez [le moteur de l’amarrage sur Windows](../manage-docker/configure-docker-daemon.md).

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

Pour installer Dockr sur Windows Server, vous pouvez utiliser un [module PowerShell Provider OneGet](https://github.com/oneget/oneget) publié par Microsoft, appelé [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider). Ce fournisseur active la fonctionnalité conteneurs dans Windows et installe le moteur et le client de l’outil d’ancrage. Voici comment procéder :

1. Ouvrez une session PowerShell avec élévation de privilèges et installez le fournisseur Dockr-Microsoft PackageManagement à partir du [PowerShell Gallery](https://www.powershellgallery.com/packages/DockerMsftProvider).

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   Si vous êtes invité à installer le fournisseur NuGet, tapez `Y` pour l’installer également.

2. Utilisez le module PowerShell PackageManagement pour installer la dernière version de la station d’accueil.

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   Quand PowerShell vous demande si la source du package « DockerDefault » doit être approuvée, tapez `A` pour poursuivre l’installation.
3. Une fois l’installation terminée, redémarrez l’ordinateur.

   ```powershell
   Restart-Computer -Force
   ```

Si vous souhaitez mettre à jour docker ultérieurement :

- Vérifiez la version installée avec `Get-Package -Name Docker -ProviderName DockerMsftProvider`
- Rechercher la version actuelle avec `Find-Package -Name Docker -ProviderName DockerMsftProvider`
- Quand vous êtes prêt, effectuez une mise à niveau avec `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, suivi de `Start-Service Docker`

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

Vous pouvez installer Dockr sur les éditions Windows 10 professionnel et entreprise en procédant comme suit. 

1. Téléchargez et installez le Bureau de la [station](https://store.docker.com/editions/community/docker-ce-desktop-windows)d’accueil, en créant un compte d’ancrage gratuit si vous n’en avez pas déjà un. Pour plus d’informations, consultez la documentation de l' [ancrage](https://docs.docker.com/docker-for-windows/install).

2. Pendant l’installation, définissez le type de conteneur par défaut sur conteneurs Windows. Pour basculer une fois l’installation terminée, vous pouvez utiliser l’élément docker dans la barre d’état système Windows (comme indiqué ci-dessous), ou la commande suivante dans une invite de commandes PowerShell :

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![Menu de la barre d’état système de l’ordinateur d’amarrage qui affiche la commande « basculer vers les conteneurs Windows ».](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>Étapes suivantes

Maintenant que votre environnement a été correctement configuré, suivez le lien pour savoir comment exécuter un conteneur.

> [!div class="nextstepaction"]
> [Exécuter votre premier conteneur](./run-your-first-container.md)

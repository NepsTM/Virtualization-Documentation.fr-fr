---
title: Déployer des conteneurs Windows sur Windows Server
description: Déployer des conteneurs Windows sur Windows Server
keywords: docker, conteneurs
author: taylorb-microsoft
ms.date: 09/09/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 6e3996af36b4a710f9a12b3a1371138b053a43d8
ms.sourcegitcommit: f3b6b470dd9cde8e8cac7b13e7e7d8bf2a39aa34
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/10/2019
ms.locfileid: "10077500"
---
# <a name="container-host-deployment-windows-server"></a>Déploiement d’hébergement de conteneur: Windows Server

Les étapes du déploiement d’un hôte de conteneur Windows sont différentes selon le système d’exploitation et le type de système hôte (physique ou virtuel). Ce document décrit en détail le déploiement d’un hôte de conteneur Windows vers Windows Server2016 ou Windows Server Core2016, sur un système physique ou virtuel.

## <a name="install-docker"></a>Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. L’arrimeur se compose du moteur de l’ancrage et du client de l’ancrage.

Pour installer docker, nous utilisons le [module PowerShell du fournisseur OneGet](https://github.com/OneGet/MicrosoftDockerProvider). Le fournisseur active la fonctionnalité conteneurs sur votre ordinateur et installateur, ce qui nécessite un redémarrage.

Ouvrez une session PowerShell avec élévation de privilèges et exécutez les applets de commande suivantes.

Installez le module PowerShell OneGet.

```PowerShell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Utilisez OneGet pour installer la dernière version de Docker.

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Une fois l’installation terminée, redémarrez l’ordinateur.

```PowerShell
Restart-Computer -Force
```

## <a name="install-a-specific-version-of-docker"></a>Installer une version spécifique de docker

Deux canaux sont actuellement disponibles pour le docker EE pour Windows Server:

* `17.06` -Utilisez cette version si vous utilisez Dockr Enterprise Edition (moteur de l’amarrage, UCP, DTR). `17.06` est la valeur par défaut.
* `18.03` -Utilisez cette version si vous exécutez le moteur Dockr EE uniquement.

Pour installer une version spécifique, utilisez l' `RequiredVersion` indicateur:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

Pour installer des versions spécifiques de docker EE, il est possible que vous ayez besoin d’une mise à jour des modules DockerMsftProvider précédemment installés. Pour mettre à jour:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Mettre à jour l’ancrage

Si vous avez besoin de mettre à jour le moteur docker EE d’un canal antérieur vers un canal ultérieur `-Update` , `-RequiredVersion` utilisez les indicateurs et:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>Installer les images du conteneur de base

Avant d’utiliser des conteneurs Windows, une image de base doit être installée. Les images de base sont disponibles avec Windows Server Core ou Nano Server comme système d’exploitation de conteneur. Pour plus d’informations sur les images de conteneur Docker, consultez [Créer vos propres images sur docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

Avec la version de Windows Server 2019, les images de conteneur de sources Microsoft sont déplacées vers un nouveau registre appelé Registre de conteneurs Microsoft. Les images de conteneur publiées par Microsoft doivent rester découvertes par le biais du Hub de l’amarrage. Pour les nouvelles images de conteneur publiées avec Windows Server 2019 et les versions ultérieures, vous devez chercher à les récupérer à partir de la fonction de multifonction. Pour les images de conteneur plus anciennes publiées avant Windows Server 2019, vous devez continuer à les extraire du registre de l’ancrage.

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 et versions ultérieures

Pour installer l’image de base «Windows Server Core», exécutez la commande suivante:

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

Pour installer l’image de base «nano Server», exécutez la commande suivante:

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (version 1607-1803)

Exécutez la commande suivante pour installer l’image de base Windows Server Core :

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:1607
```

Pour installer l’image de base Nano Server, exécutez la commande suivante:

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1803
```

> Pour plus d’informations, consultez [le CLUF sur](../images-eula.md)l’image du système d’exploitation Windows Container.

## <a name="hyper-v-isolation-host"></a>Hôte d’isolation Hyper-V

Vous devez avoir le rôle Hyper-V pour exécuter l’isolation Hyper-V. Si l’hôte de conteneur Windows est lui-même une machine virtuelle Hyper-V, la virtualisation imbriquée doit être activée avant d’installer le rôle Hyper-V. Pour plus d’informations sur la virtualisation imbriquée, voir [Virtualisation imbriquée](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization).

### <a name="nested-virtualization"></a>Virtualisation imbriquée

Le script suivant configure la virtualisation imbriquée pour l’hôte de conteneur. Ce script est exécuté sur l’ordinateur Hyper-V parent. Vérifiez que la machine virtuelle de l’hôte de conteneur est désactivée lors de l’exécution de ce script.

```PowerShell
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory -VMName $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="enable-the-hyper-v-role"></a>Activer le rôle Hyper-V

Pour activer la fonctionnalité Hyper-V à l’aide de PowerShell, exécutez l’applet de commande suivante dans une session PowerShell avec élévation de privilèges.

```PowerShell
Install-WindowsFeature hyper-v
```

---
title: Déployer des conteneurs Windows sur Windows Server
description: Déployer des conteneurs Windows sur Windows Server
keywords: docker, conteneurs
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 0d982996a1aabd434df04551f30725a21b31d500
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2018
ms.locfileid: "6948008"
---
# <a name="container-host-deployment---windows-server"></a>Déploiement d’un hôte de conteneurs–Windows Server

Les étapes du déploiement d’un hôte de conteneur Windows sont différentes selon le système d’exploitation et le type de système hôte (physique ou virtuel). Ce document décrit en détail le déploiement d’un hôte de conteneur Windows vers Windows Server2016 ou Windows Server Core2016, sur un système physique ou virtuel.

## <a name="install-docker"></a>Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. 

Pour installer Docker, nous allons utiliser le [module PowerShell de fournisseur OneGet](https://github.com/OneGet/MicrosoftDockerProvider). Le fournisseur active la fonctionnalité de conteneurs sur votre ordinateur et installe Docker. Cette opération nécessite un redémarrage. 

Ouvrez une session PowerShell avec élévation de privilèges, puis exécutez les commandes suivantes.

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

## <a name="install-a-specific-version-of-docker"></a>Installer une Version spécifique de Docker

Deux canaux sont actuellement disponibles pour Docker EE pour Windows Server:

* `17.06` : Utilisez cette version si vous utilisez Docker Enterprise Edition (moteur Docker, UCP, DTR). `17.06` est la valeur par défaut.
* `18.03` : Utilisez cette version si vous exécutez Docker EE moteur seul.

Pour installer une version spécifique, utilisez la `RequiredVersion` indicateur:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

L’installation des versions spécifiques de Docker EE peut nécessiter une mise à jour des modules DockerMsftProvider précédemment installés. Pour mettre à jour:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Mise à jour Docker

Si vous avez besoin mettre à jour du moteur de Docker EE à partir d’un canal antérieur à un canal ultérieure, utilisez à la fois le `-Update` et `-RequiredVersion` indicateurs:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>Installer les images de conteneur de base

Avant de travailler avec des conteneurs Windows, une image de base doit être installée. Les images de base sont disponibles avec Windows Server Core ou Nano Server comme système d’exploitation de conteneur. Pour plus d’informations sur les images de conteneur Docker, consultez [Créer vos propres images sur docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

Avec la version de Windows Server 2019, les images de conteneur proviennent de Microsoft migrent vers un Registre appelé le Registre de conteneur de Microsoft. Publié par Microsoft des images de conteneur doivent continuer à être détectés via Docker Hub. De nouvelles images de conteneur publiés avec Windows Server 2019 et versions ultérieures, vous doivent se présenter pour extraire les règles d’ordinateurs gérés. Pour les images de conteneur publiés avant Windows Server 2019 plus anciens, vous devez continuer à les collecter à partir du Registre de Docker.

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 et versions ultérieures

Pour installer l’image de base 'Windows Server Core' exécutez la commande suivante:

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

Pour installer l’image de base 'Nano Server' exécutez la commande suivante:

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (versions 1607-1803)

Exécutez la commande suivante pour installer l’image de base Windows Server Core :

```PowerShell
docker pull microsoft/windowsservercore
```

Pour installer l’image de base Nano Server, exécutez la commande suivante:

```PowerShell
docker pull microsoft/nanoserver
```

> Veuillez lire les termes du contrat de licence applicables à l’image de système d’exploitation des conteneurs Windows, disponibles [ici](../images-eula.md).

## <a name="hyper-v-container-host"></a>Hôte de conteneurs Hyper-V

Pour exécuter des conteneurs Hyper-V, le rôle Hyper-V est nécessaire. Si l’hôte de conteneur Windows est lui-même une machine virtuelle Hyper-V, la virtualisation imbriquée doit être activée avant d’installer le rôle Hyper-V. Pour plus d’informations sur la virtualisation imbriquée, voir [Virtualisation imbriquée]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

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

Pour activer la fonctionnalité Hyper-V à l’aide de PowerShell, exécutez la commande suivante dans une session PowerShell avec élévation de privilèges.

```PowerShell
Install-WindowsFeature hyper-v
```

---
title: "Déployer des conteneurs Windows sur Windows Server"
description: "Déployer des conteneurs Windows sur Windows Server"
keywords: docker, conteneurs
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 12c7c713468618a9fedc82ec5a1c488f57edcfd7
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/21/2017
---
# Déploiement d’un hôte de conteneurs–Windows Server

Les étapes du déploiement d’un hôte de conteneur Windows sont différentes selon le système d’exploitation et le type de système hôte (physique ou virtuel). Ce document décrit en détail le déploiement d’un hôte de conteneur Windows vers Windows Server2016 ou Windows Server Core2016, sur un système physique ou virtuel.

## Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. 

Pour installer Docker, nous allons utiliser le [module PowerShell de fournisseur OneGet](https://github.com/OneGet/MicrosoftDockerProvider). Le fournisseur active la fonctionnalité de conteneurs sur votre ordinateur et installe Docker. Cette opération nécessite un redémarrage. 

Ouvrez une session PowerShell avec élévation de privilèges, puis exécutez les commandes suivantes.

Installez le module PowerShell OneGet.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Utilisez OneGet pour installer la dernière version de Docker.

```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Une fois l’installation terminée, redémarrez l’ordinateur.

```none
Restart-Computer -Force
```

## Installer les images de conteneur de base

Avant de travailler avec des conteneurs Windows, une image de base doit être installée. Les images de base sont disponibles avec Windows Server Core ou Nano Server comme système d’exploitation de conteneur. Pour plus d’informations sur les images de conteneur Docker, consultez [Créer vos propres images sur docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

Exécutez la commande suivante pour installer l’image de base Windows Server Core :

```none
docker pull microsoft/windowsservercore
```

Pour installer l’image de base Nano Server, exécutez la commande suivante:

```none
docker pull microsoft/nanoserver
```

> Veuillez lire les termes du contrat de licence applicables à l’image de système d’exploitation des conteneurs Windows, disponibles [ici](../images-eula.md).

## Hôte de conteneurs Hyper-V

Pour exécuter des conteneurs Hyper-V, le rôle Hyper-V est nécessaire. Si l’hôte de conteneur Windows est lui-même une machine virtuelle Hyper-V, la virtualisation imbriquée doit être activée avant d’installer le rôle Hyper-V. Pour plus d’informations sur la virtualisation imbriquée, voir [Virtualisation imbriquée]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

### Virtualisation imbriquée

Le script suivant configure la virtualisation imbriquée pour l’hôte de conteneur. Ce script est exécuté sur l’ordinateur Hyper-V parent. Vérifiez que la machine virtuelle de l’hôte de conteneur est désactivée lors de l’exécution de ce script.

```none
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### Activer le rôle Hyper-V

Pour activer la fonctionnalité Hyper-V à l’aide de PowerShell, exécutez la commande suivante dans une session PowerShell avec élévation de privilèges.

```none
Install-WindowsFeature hyper-v
```

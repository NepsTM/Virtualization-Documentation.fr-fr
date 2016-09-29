---
title: "Déployer des conteneurs Windows sur Windows Server"
description: "Déployer des conteneurs Windows sur Windows Server"
keywords: docker, conteneurs
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
translationtype: Human Translation
ms.sourcegitcommit: f721639b1b10ad97cc469df413d457dbf8d13bbe
ms.openlocfilehash: 4d7e8fb1fcbb7e9680b7d5bd143ef6d59e45035e

---

# Déploiement d’un hôte de conteneurs – Windows Server

Les étapes du déploiement d’un hôte de conteneur Windows sont différentes selon le système d’exploitation et le type de système hôte (physique ou virtuel). Ce document décrit en détail le déploiement d’un hôte de conteneur Windows vers Windows Server 2016 ou Windows Server Core 2016, sur un système physique ou virtuel.

## Installer la fonctionnalité de conteneur

La fonctionnalité de conteneur doit être activée avant d’utiliser des conteneurs Windows. Pour ce faire, exécutez la commande suivante dans une session PowerShell avec élévation de privilèges.

```none
Install-WindowsFeature containers
```

Une fois l’installation de la fonctionnalité terminée, redémarrez l’ordinateur.

```none
Restart-Computer -Force
```

## Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. Pour cet exercice, les deux seront installés.

Téléchargez le moteur Docker et le client au format d’archive zip.

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Développez l’archive zip dans Program Files, le contenu de l’archive est déjà dans le répertoire de docker.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Exécutez les deux commandes suivantes pour ajouter le répertoire Docker au chemin d’accès système.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Pour installer Docker comme service Windows, exécutez la commande suivante.

```none
dockerd --register-service
```

Une fois installé, le service peut être démarré.

```none
Start-Service Docker
```

## Installer les images de conteneur de base

Avant de travailler avec des conteneurs Windows, une image de base doit être installée. Les images de base sont disponibles avec Windows Server Core ou Nano Server comme système d’exploitation sous-jacent. Pour plus d’informations sur les images de conteneur Docker, consultez [Build your own images (Créer vos propres images) sur docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

Exécutez la commande suivante pour installer l’image de base Windows Server Core :

```none
docker pull microsoft/windowsservercore
```

Pour installer l’image de base Nano Server, exécutez la commande suivante :

```none
docker pull microsoft/nanoserver
```

> Veuillez lire les termes du contrat de licence applicables à l’image de système d’exploitation des conteneurs Windows, disponibles [ici](../Images_EULA.md).

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



<!--HONumber=Sep16_HO4-->



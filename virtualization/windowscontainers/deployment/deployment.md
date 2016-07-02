---
title: "Déployer des conteneurs Windows sur Windows Server"
description: "Déployer des conteneurs Windows sur Windows Server"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
translationtype: Human Translation
ms.sourcegitcommit: eae45c2c81c7edc94d963da69dcdee2b6f08f37d
ms.openlocfilehash: cbbff2bf4a68ee348bcc33979ef4469daf54a8a7

---

# Déploiement d’un hôte de conteneurs – Windows Server

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.**

Les étapes du déploiement d’un hôte de conteneur Windows sont différentes selon le système d’exploitation et le type de système hôte (physique ou virtuel). Ce document décrit en détail le déploiement d’un hôte de conteneur Windows vers Windows Server 2016 ou Windows Server Core 2016, sur un système physique ou virtuel.

## Image Azure 

Une image Windows Server entièrement configurée est disponible dans Azure. Pour utiliser cette image, déployez une machine virtuelle en cliquant sur le bouton ci-dessous. Si vous déployez un système de conteneur Windows vers Azure à l’aide de ce modèle, le reste de ce document peut être ignoré.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

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

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. Pour cet exercice, les deux sont installés.

Créez un dossier pour les exécutables Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Téléchargez le démon Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Téléchargez le client Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

Ajoutez le répertoire Docker au chemin du système.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Redémarrez la session PowerShell pour que le chemin modifié soit reconnu.

Pour installer Docker comme service Windows, exécutez la commande suivante.

```none
dockerd --register-service
```

Une fois installé, le service peut être démarré.

```none
Start-Service Docker
```

## Installer les images de conteneur de base

Avant de pouvoir déployer un conteneur, une image de système d’exploitation de base de conteneur doit être téléchargée. L’exemple suivant télécharge l’image de système d’exploitation de base Windows Server Core. Cette même procédure peut être suivie pour installer l’image de base Nano Server. Pour plus d’informations sur les images de conteneur Windows, voir la rubrique relative à la [gestion des images de conteneur](../management/manage_images.md).

Tout d’abord, installez le fournisseur de package d’images de conteneur.

```none
Install-PackageProvider ContainerImage -Force
```

Ensuite, installez l’image Windows Server Core. Ce processus peut prendre du temps. Faites une pause, puis reprenez une fois le téléchargement terminé.

```none
Install-ContainerImage -Name WindowsServerCore    
```

Une fois l’image de base installée, le service Docker doit être redémarré.

```none
Restart-Service docker
```

Enfin, l’image doit être marquée avec une version de la balise « latest ». Pour ce faire, exécutez la commande suivante.

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

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



<!--HONumber=Jun16_HO5-->



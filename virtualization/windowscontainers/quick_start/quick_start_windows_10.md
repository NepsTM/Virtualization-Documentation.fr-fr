---
title: "Conteneur Windows sur Windows 10"
description: "Démarrage rapide du déploiement de conteneurs"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 07/13/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: edf2c2597e57909a553eb5e6fcc75cdb820fce68
ms.openlocfilehash: b37d402f2e6c950db061f5de0c86f0e9aace62b4

---

# Conteneurs Windows sur Windows 10

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

L’exercice vous guide tout au long du déploiement et de l’utilisation de base de la fonctionnalité de conteneur Windows 10 (builds Insider 14372 et ultérieures). Une fois terminé, vous aurez installé le rôle de conteneur et déployé un conteneur Hyper-V simple. Avant de commencer ce démarrage rapide, familiarisez-vous avec la terminologie et les concepts de base des conteneurs. Ces informations figurent dans la [Présentation du démarrage rapide](./quick_start.md). 

Ce démarrage rapide est spécifique aux conteneurs Hyper-V sur Windows 10. Une documentation de démarrage rapide supplémentaire est disponible dans la table des matières affichée à gauche dans cette page.

**Conditions préalables :**

- Un système informatique physique exécutant une [version de Windows 10 Insiders](https://insider.windows.com/).   
- Ce démarrage rapide peut être exécuté sur une machine virtuelle Windows 10. Toutefois, la virtualisation imbriquée doit être activée. Pour plus d’informations, voir le [Guide de la virtualisation imbriquée](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

## 1. Installer la fonctionnalité de conteneur

La fonctionnalité de conteneur doit être activée avant d’utiliser des conteneurs Windows. Pour ce faire, exécutez la commande suivante dans une session PowerShell avec élévation de privilèges. 

```none
Enable-WindowsOptionalFeature -Online -FeatureName containers -All
```

Étant donné que Windows 10 prend uniquement en charge les conteneurs Hyper-V, la fonctionnalité Hyper-V doit également être activée. Pour activer la fonctionnalité Hyper-V à l’aide de PowerShell, exécutez la commande suivante dans une session PowerShell avec élévation de privilèges.

```none
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

Une fois l’installation terminée, redémarrez l’ordinateur.

```none
Restart-Computer -Force
```

Une fois la sauvegarde effectuée, exécutez la commande suivante pour résoudre un problème connu lié à la version d’évaluation technique des conteneurs Windows.  

 ```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

## 2. Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. Pour cet exercice, les deux seront installés. Pour ce faire, exécutez la commande suivante. 

Créez un dossier pour les exécutables Docker.

```none
New-Item -Type Directory -Path $env:ProgramFiles\docker\
```

Téléchargez le démon Docker.

```none
Invoke-WebRequest https://master.dockerproject.org/windows/amd64/dockerd.exe -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Téléchargez le client Docker.

```none
Invoke-WebRequest https://master.dockerproject.org/windows/amd64/docker.exe -OutFile $env:ProgramFiles\docker\docker.exe
```

Ajoutez le répertoire Docker au chemin du système.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:ProgramFiles\docker\", [EnvironmentVariableTarget]::Machine)
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

## 3. Installer les images de conteneur de base

Les conteneurs Windows sont déployés à partir de modèles ou d’images. Avant de pouvoir déployer un conteneur, une image de système d’exploitation de base de conteneur doit être téléchargée. Les commandes suivantes téléchargent l’image de base Nano Server.
    
> Cette procédure, qui s’applique aux builds Windows Insider ultérieures à 14372, est temporaire tant que la commande « pull docker » n’est pas fonctionnelle.

Téléchargez l’image de base Nano Server. 

```none
Start-BitsTransfer https://aka.ms/tp5/6b/docker/nanoserver -Destination nanoserver.tar.gz
```

Installez l’image de base.

```none  
docker load -i nanoserver.tar.gz
```

À ce stade, l’exécution de la commande `docker images` retourne une liste d’images installées ; dans le cas présent, l’image Nano Server.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1030     3f5112ddd185        3 weeks ago         810.2 MB
```

Avant de poursuivre, cette image doit être marquée de la balise « latest ». Pour ce faire, exécutez la commande suivante.

```none
docker tag microsoft/nanoserver:10.0.14300.1030 nanoserver:latest
```

Pour obtenir des informations détaillées sur les images de conteneur Windows, voir la rubrique relative à la [gestion des images de conteneur](../management/manage_images.md).

## 4. Déployer votre premier conteneur

Pour ce simple exemple, une image de conteneur « Hello World » est créée et déployée. Pour une expérience optimale, exécutez ces commandes dans une interface de commande Windows élevée.

Commencez par démarrer un conteneur avec une session interactive à partir de l’image `nanoserver`. Une fois que le conteneur a démarré, une interface de commande s’affiche à partir du conteneur.  

```none
docker run -it nanoserver cmd
```

À l’intérieur du conteneur, nous allons créer un script « Hello World » simple.

```none
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

Quand vous avez terminé, quittez le conteneur.

```none
exit
```

Vous allez maintenant créer une image de conteneur à partir du conteneur modifié. Pour afficher une liste de conteneurs, exécutez la commande suivante et notez l’ID de conteneur.

```none
docker ps -a
```

Exécutez la commande suivante pour créer l’image « Hello World ». Remplacez <containerid> par l’ID de votre conteneur.

```none
docker commit <containerid> helloworld
```

Quand vous avez terminé, vous disposez d’une image personnalisée qui contient le script « Hello World ». Pour l’afficher, utilisez la commande suivante.

```none
docker images
```

Enfin, pour exécuter le conteneur, utilisez la commande `docker run`.

```none
docker run --rm helloworld powershell c:\helloworld.ps1
```

Le résultat de la commande `docker run` est qu’un conteneur Hyper-V a été créé à partir de l’image « Hello World », un exemple de script « Hello World » a été exécuté (sortie répercutée sur l’interface de commande), puis le conteneur s’est arrêté et a été supprimé. Les démarrages rapides suivants de Windows 10 et des conteneurs exploreront la création et le déploiement d’applications dans des conteneurs sur Windows 10.

## Étapes suivantes

[Conteneurs Windows sur Windows Server](./quick_start_windows_server.md)





<!--HONumber=Jul16_HO2-->



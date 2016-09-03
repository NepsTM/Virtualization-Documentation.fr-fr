---
title: "Conteneur Windows sur Windows 10"
description: "Démarrage rapide du déploiement de conteneurs"
keywords: docker, conteneurs
author: neilpeterson
manager: timlt
ms.date: 08/17/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 2319649d1dd39677e59a9431fbefaf82982492c6
ms.openlocfilehash: 8d3c8263819688d1a47893726619458ee44be59b

---

# Conteneurs Windows sur Windows 10

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

L’exercice vous guide tout au long du déploiement et de l’utilisation de base de la fonctionnalité de conteneur Windows 10 Professionnel ou Entreprise (Édition anniversaire). Une fois terminé, vous aurez installé le rôle de conteneur et déployé un conteneur Hyper-V simple. Avant de commencer ce démarrage rapide, familiarisez-vous avec la terminologie et les concepts de base des conteneurs. Ces informations figurent dans la [Présentation du démarrage rapide](./quick_start.md). 

Ce démarrage rapide est spécifique aux conteneurs Hyper-V sur Windows 10. Une documentation de démarrage rapide supplémentaire est disponible dans la table des matières affichée à gauche dans cette page.

**Conditions préalables :**

- Un système d’ordinateur physique exécutant Windows 10 Édition anniversaire (Professionnel ou Entreprise).   
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

> Dans les versions actuelles, vous devez désactiver OpLocks pour utiliser des conteneurs Hyper-V de façon fiable. Pour réactiver OpLocks, utilisez la commande suivante :  `Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 0 -Force`

## 2. Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. Pour cet exercice, les deux seront installés. Pour ce faire, exécutez la commande suivante. 

Téléchargez le moteur Docker et le client au format d’archive zip.

```none
Invoke-WebRequest "https://master.dockerproject.org/windows/amd64/docker-1.13.0-dev.zip" -OutFile "$env:TEMP\docker-1.13.0-dev.zip" -UseBasicParsing
```

Développez l’archive zip dans Program Files, le contenu de l’archive est déjà dans le répertoire de docker.

```none
Expand-Archive -Path "$env:TEMP\docker-1.13.0-dev.zip" -DestinationPath $env:ProgramFiles
```

Ajoutez le répertoire Docker au chemin du système.

```none
# for quick use, does not require shell to be restarted
$env:path += ";c:\program files\docker"

# for persistent use, will apply even after a reboot 
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

## 3. Installer les images de conteneur de base

Les conteneurs Windows sont déployés à partir de modèles ou d’images. Avant de pouvoir déployer un conteneur, une image de système d’exploitation de base de conteneur doit être téléchargée. Les commandes suivantes téléchargent l’image de base Nano Server.

Extrayez l’image de base Nano Server. 

```none
docker pull microsoft/nanoserver
```

Une fois l’image extraite, l’exécution de la commande `docker images` retourne une liste d’images installées ; dans le cas présent, l’image Nano Server.

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              3a703c6e97a2        7 weeks ago         969.8 MB
```

Pour obtenir des informations détaillées sur les images de conteneur Windows, voir la rubrique relative à la [gestion des images de conteneur](../management/manage_images.md).

## 4. Déployer votre premier conteneur

Pour ce simple exemple, une image de conteneur « Hello World » est créée et déployée. Pour une expérience optimale, exécutez ces commandes dans une interface de commande Windows élevée.

Commencez par démarrer un conteneur avec une session interactive à partir de l’image `nanoserver`. Une fois que le conteneur a démarré, une interface de commande s’affiche à partir du conteneur.  

```none
docker run -it microsoft/nanoserver cmd
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





<!--HONumber=Aug16_HO4-->



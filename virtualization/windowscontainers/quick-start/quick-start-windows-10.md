---
title: "Conteneur Windows sur Windows 10"
description: "Démarrage rapide du déploiement de conteneurs"
keywords: docker, conteneurs
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 996d3b1a8f7c8325ac66d331e1d62208c0cf6b53
ms.openlocfilehash: 091a3570291624a3be40e3aabb9f99a482cb6470
ms.lasthandoff: 02/27/2017

---

# Conteneurs Windows sur Windows 10

L’exercice vous guide tout au long du déploiement et de l’utilisation de base de la fonctionnalité de conteneur Windows 10 Professionnel ou Entreprise (Édition anniversaire). Une fois terminé, vous aurez installé le rôle de conteneur et déployé un conteneur Hyper-V simple. Avant de commencer ce démarrage rapide, familiarisez-vous avec la terminologie et les concepts de base des conteneurs. Ces informations figurent dans la [Présentation du démarrage rapide](./index.md).

Ce démarrage rapide est spécifique aux conteneurs Hyper-V sur Windows 10. Une documentation de démarrage rapide supplémentaire est disponible dans la table des matières affichée à gauche dans cette page.

**Conditions préalables :**

- Un système d’ordinateur physique exécutant Windows 10 Édition anniversaire (Professionnel ou Entreprise).   
- Ce démarrage rapide peut être exécuté sur une machine virtuelle Windows 10. Toutefois, la virtualisation imbriquée doit être activée. Pour plus d’informations, voir le [Guide de la virtualisation imbriquée](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

> Vous devez installer les mises à jour critiques pour que les conteneurs Windows fonctionnent. 
> Pour vérifier la version de votre système d’exploitation, exécutez `winver.exe` et comparez la version indiquée dans [l’historique des mises à jour Windows 10](https://support.microsoft.com/en-us/help/12387/windows-10-update-history). 
> Assurez-vous d’avoir la version 14393.222 ou ultérieure avant de continuer.

## 1. Installer la fonctionnalité de conteneur

La fonctionnalité de conteneur doit être activée avant d’utiliser des conteneurs Windows. Pour ce faire, exécutez la commande suivante dans une session PowerShell **avec élévation de privilèges**.

Si vous recevez un message d’erreur indiquant que `Enable-WindowsOptionalFeature` n’existe pas, vérifiez que vous exécutez PowerShell comme administrateur.

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

> Si vous utilisiez précédemment des conteneurs Hyper-V sur des images de base du conteneur Windows 10 Technical Preview 5, vous devez réactiver OpLocks. Exécutez la commande suivante :  `Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 0 -Force`

## 2. Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. Pour cet exercice, les deux seront installés. Pour ce faire, exécutez la commande suivante.

Téléchargez le moteur Docker et le client au format d’archive zip.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-17.03.0-ce.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Développez l’archive zip dans Program Files, le contenu de l’archive est déjà dans le répertoire de docker.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Ajoutez le répertoire Docker au chemin d'accès système.

```none
# Add path to this PowerShell session immediately
$env:path += ";$env:ProgramFiles\Docker"

# For persistent use after a reboot
$existingMachinePath = [Environment]::GetEnvironmentVariable("Path",[System.EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("Path", $existingMachinePath + ";$env:ProgramFiles\Docker", [EnvironmentVariableTarget]::Machine)
```

Pour installer Docker en tant que service Windows, exécutez la commande suivante.

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
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> Veuillez lire les termes du contrat de licence applicables à l’image de système d’exploitation des conteneurs Windows, disponibles [ici](../images-eula.md).

## 4. Déployer votre premier conteneur

Pour ce simple exemple, une image de conteneur « Hello World » est créée et déployée. Pour une expérience optimale, exécutez ces commandes dans une interface de commande Windows ou PowerShell élevée.

> Windows PowerShell ISE ne fonctionne pas pour les sessions interactives avec des conteneurs. Même si le conteneur est en cours d’exécution, il apparaît comme étant bloqué.

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

Le résultat de la commande `docker run` est qu’un conteneur Hyper-V a été créé à partir de l’image « Hello World », un exemple de script « Hello World » a été exécuté (sortie répercutée sur l’interface de commande), puis le conteneur s’est arrêté et a été supprimé.
Les démarrages rapides suivants de Windows 10 et des conteneurs exploreront la création et le déploiement d’applications dans des conteneurs sur Windows 10.

## Étapes suivantes

[Conteneurs Windows sur Windows Server](./quick-start-windows-server.md)


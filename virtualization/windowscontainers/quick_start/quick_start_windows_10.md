---
title: "Conteneur Windows sur Windows 10"
description: "Démarrage rapide du déploiement de conteneurs"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/28/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 5980babe886024de93f6d6c5f04eaed47407209d
ms.openlocfilehash: 188c85a9e6f5d1c334e51853efd8fa3ca461837c

---

# Conteneurs Windows sur Windows 10

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

L’exercice vous guide lors du déploiement et de l’utilisation de base de la fonctionnalité de conteneur Windows 10 (Insiders build 14352 et ultérieures). Une fois terminé, vous aurez installé le rôle de conteneur et déployé un conteneur Hyper-V simple. Avant de commencer ce démarrage rapide, familiarisez-vous avec la terminologie et les concepts de base des conteneurs. Ces informations figurent dans la [Présentation du démarrage rapide](./quick_start.md). 

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

## 2. Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. Pour cet exercice, les deux seront installés. Pour ce faire, exécutez la commande suivante. 

Créez un dossier pour les exécutables Docker.

```none
New-Item -Type Directory -Path $env:ProgramFiles\docker\
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
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:ProgramFiles\docker\\Docker", [EnvironmentVariableTarget]::Machine)
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
    
Définissez la stratégie d’exécution PowerShell pour le processus PowerShell actuel. Cela affecte uniquement les scripts exécutés dans la session PowerShell actuelle, mais il convient d’être prudent quand vous changez de stratégie d’exécution.

```none
Set-ExecutionPolicy Bypass -scope Process
```

Installez le fournisseur de package d’images de conteneur.

```none  
Install-PackageProvider ContainerImage -Force
```

Installez ensuite l’image Nano Server.

```none
Install-ContainerImage -Name NanoServer
```

Une fois l’image de base installée, le service Docker doit être redémarré.

```none
Restart-Service docker
```

À ce stade, l’exécution de la commande `docker images` retourne une liste d’images installées ; dans le cas présent, l’image Nano Server.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1016     3f5112ddd185        3 weeks ago         810.2 MB
```

Avant de poursuivre, cette image doit être marquée de la balise « latest ». Pour ce faire, exécutez la commande suivante.

```none
docker tag nanoserver:10.0.14300.1016 nanoserver:latest
```

Pour obtenir des informations détaillées sur les images de conteneur Windows, voir la rubrique relative à la [gestion des images de conteneur](../management/manage_images.md).

## 4. Déployer votre premier conteneur

Pour cet exemple simple, une image .NET Core a été précréée. Téléchargez cette image à l’aide de la commande `docker pull`.

Une fois cette commande exécutée, un conteneur est démarré, l’application .NET Core simple s’exécute, puis le conteneur se ferme. 

```none
docker pull microsoft/sample-dotnet
```

Pour le vérifier, exécutez la commande `docker images`.

```none
docker images

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
microsoft/sample-dotnet  latest              28da49c3bff4        41 hours ago        918.3 MB
nanoserver               10.0.14300.1016     3f5112ddd185        3 weeks ago         810.2 MB
nanoserver               latest              3f5112ddd185        3 weeks ago         810.2 MB
```

Exécutez le conteneur à l’aide de la commande `docker run`. L’exemple suivant spécifie le paramètre `--rm`. Cela indique au moteur Docker de supprimer le conteneur une fois qu’il n’est plus en cours d’exécution. 

Pour obtenir des informations détaillées sur la commande Docker Run, voir [Docker Run Reference]( https://docs.docker.com/engine/reference/run/) sur Docker.com.

```none
docker run --isolation=hyperv --rm microsoft/sample-dotnet
```

**Remarque** : Si une erreur indiquant un événement de dépassement de délai d’attente s’affiche, exécutez le script PowerShell suivant et recommencez l’opération.

```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

Le résultat de la commande `docker run` est qu’un conteneur Hyper-V a été créé à partir de l’image sample-dotnet, un exemple d’application a été exécuté (sortie répercutée sur l’interpréteur de commandes), puis le conteneur s’est arrêté et a été supprimé. Les démarrages rapides suivants de Windows 10 et des conteneurs exploreront la création et le déploiement d’applications dans des conteneurs sur Windows 10.

## Étapes suivantes

[Conteneurs Windows sur Windows Server](./quick_start_windows_server.md)





<!--HONumber=Jul16_HO1-->



---
title: Conteneurs Windows sur Windows Server
description: "Démarrage rapide du déploiement de conteneurs"
keywords: docker, conteneurs
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 6c7ce9f1767c6c6391cc6d33a553216bd815ff72
ms.openlocfilehash: 4e7123dcff2564bd264c91f228941e86a0723674

---

# Conteneurs Windows sur Windows Server

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.**

Cet exercice vous guide lors du déploiement et l’utilisation de base de la fonctionnalité de conteneur Windows sur Windows Server. Une fois terminé, vous aurez installé le rôle de conteneur et déployé un conteneur Windows Server simple. Avant de commencer ce démarrage rapide, familiarisez-vous avec la terminologie et les concepts de base des conteneurs. Ces informations figurent dans la [Présentation du démarrage rapide](./quick_start.md).

Ce démarrage rapide est spécifique aux conteneurs Windows Server sur Windows Server 2016. Une documentation de démarrage rapide supplémentaire est disponible dans la table des matières affichée à gauche dans cette page.

**Conditions préalables :**

Un système informatique (physique ou virtuel) exécutant [Windows Server 2016 Technical Preview 5](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview).

Une image Windows Server entièrement configurée est disponible dans Azure. Pour utiliser cette image, déployez une machine virtuelle en cliquant sur le bouton ci-dessous. Le déploiement prendra environ 10 minutes. Une fois qu’il est terminé, connectez-vous à la machine virtuelle Azure et passez à l’étape 4 de ce didacticiel. 

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## 1. Installer la fonctionnalité de conteneur

La fonctionnalité de conteneur doit être activée avant d’utiliser des conteneurs Windows. Pour ce faire, exécutez la commande suivante dans une session PowerShell avec élévation de privilèges.

```none
Install-WindowsFeature containers
```

Une fois l’installation de la fonctionnalité terminée, redémarrez l’ordinateur.

```none
Restart-Computer -Force
```

## 2. Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. Pour cet exercice, les deux seront installés.

Téléchargez le moteur Docker et le client au format d’archive zip.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Développez l’archive zip dans Program Files, le contenu de l’archive est déjà dans le répertoire de docker.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Ajoutez le répertoire Docker au chemin du système. Quand vous avez terminé, redémarrez la session PowerShell pour que le chemin d’accès modifié soit reconnu.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Pour installer Docker en tant que service Windows, exécutez la commande suivante.

```none
& $env:ProgramFiles\docker\dockerd.exe --register-service
```

Une fois installé, le service peut être démarré.

```none
Start-Service docker
```

## 3. Installer les images de conteneur de base

Les conteneurs Windows sont déployés à partir de modèles ou d’images. Avant de pouvoir déployer un conteneur, une image de système d’exploitation de base doit être téléchargée. Les commandes suivantes téléchargent l’image de base Windows Server Core.

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

À ce stade, l’exécution de la commande `docker images` retourne une liste d’images installées ; dans le cas présent, l’image Windows Server Core.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
windowsservercore   10.0.14300.1000     dbfee88ee9fd        7 weeks ago         9.344 GB
```

Avant de poursuivre, cette image doit être marquée de la balise « latest ». Pour ce faire, exécutez la commande suivante.

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

Pour obtenir des informations détaillées sur les images de conteneur Windows, voir la rubrique relative à la [gestion des images de conteneur](../management/manage_images.md).

## 4. Déployer votre premier conteneur

Dans cet exercice, vous allez télécharger une image IIS précréée à partir du Registre du hub Docker et déployer un conteneur simple qui exécute IIS.  

Pour rechercher des images de conteneur Windows dans le hub Docker, exécutez la commande `docker search Microsoft`.  

```none
docker search microsoft

NAME                                         DESCRIPTION                                     
microsoft/sample-django:windowsservercore    Django installed in a Windows Server Core ...   
microsoft/dotnet35:windowsservercore         .NET 3.5 Runtime installed in a Windows Se...   
microsoft/sample-golang:windowsservercore    Go Programming Language installed in a Win...   
microsoft/sample-httpd:windowsservercore     Apache httpd installed in a Windows Server...   
microsoft/iis:windowsservercore              Internet Information Services (IIS) instal...   
microsoft/sample-mongodb:windowsservercore   MongoDB installed in a Windows Server Core...   
microsoft/sample-mysql:windowsservercore     MySQL installed in a Windows Server Core b...   
microsoft/sample-nginx:windowsservercore     Nginx installed in a Windows Server Core b...  
microsoft/sample-python:windowsservercore    Python installed in a Windows Server Core ...   
microsoft/sample-rails:windowsservercore     Ruby on Rails installed in a Windows Serve...  
microsoft/sample-redis:windowsservercore     Redis installed in a Windows Server Core b...   
microsoft/sample-ruby:windowsservercore      Ruby installed in a Windows Server Core ba...   
microsoft/sample-sqlite:windowsservercore    SQLite installed in a Windows Server Core ...  
```

Téléchargez l’image IIS à l’aide de la commande `docker pull`.  

```none
docker pull microsoft/iis:windowsservercore
```

Le téléchargement de l’image peut être vérifié avec la commande `docker images`. Notez que vous verrez à la fois l’image de base (windowsservercore) et l’image IIS.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

Utilisez la commande `docker run` pour déployer le conteneur IIS.

```none
docker run -d -p 80:80 microsoft/iis:windowsservercore ping -t localhost
```

Cette commande exécute l’image IIS en tant que service d’arrière-plan (-d) et configure la mise en réseau de façon à ce que le port 80 de l’hôte du conteneur soit mappé au port 80 du conteneur.
Pour obtenir des informations détaillées sur la commande Docker Run, voir [Docker Run Reference]( https://docs.docker.com/engine/reference/run/) sur Docker.com.


Les conteneurs en cours d’exécution peuvent être consultés à l’aide de la commande `docker ps`. Notez le nom du conteneur ; il sera utilisé dans une étape ultérieure.

```none
docker ps

CONTAINER ID    IMAGE                             COMMAND               CREATED              STATUS   PORTS                NAMES
9cad3ea5b7bc    microsoft/iis:windowsservercore   "ping -t localhost"   About a minute ago   Up       0.0.0.0:80->80/tcp   grave_jang
```

À partir d’un autre ordinateur, ouvrez un navigateur web et entrez l’adresse IP de l’hôte du conteneur. Si tout a été configuré correctement, vous devez voir l’écran de démarrage d’IIS. Cela s’effectue à partir de l’instance IIS hébergée dans le conteneur Windows.

**Remarque :** Si vous travaillez dans Azure, l’adresse IP externe de la machine virtuelle et une sécurité réseau configurée seront nécessaires. Pour plus d’informations, voir [Créer une règle dans un groupe de sécurité réseau]( https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg).

![](media/iis1.png)

De retour sur l’hôte de conteneur, utilisez la commande `docker rm` pour supprimer le conteneur. Remarque : Remplacez le nom de conteneur indiqué dans cet exemple par le nom de conteneur réel.

```none
docker rm -f grave_jang
```
## Étapes suivantes

[Images de conteneur sur Windows Server](./quick_start_images.md)

[Conteneurs Windows sur Windows 10](./quick_start_windows_10.md)



<!--HONumber=Aug16_HO1-->



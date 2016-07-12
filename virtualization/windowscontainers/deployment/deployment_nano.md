---
title: "Déployer des conteneurs Windows sur Nano Server"
description: "Déployer des conteneurs Windows sur Nano Server"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 07/06/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: e035a45e22eee04263861d935b338089d8009e92
ms.openlocfilehash: 876ffb4f4da32495fb77b735391203c33c78cff3

---

# Déploiement d’un hôte de conteneur – Nano Server

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

Ce document présente les étapes d’un déploiement Nano Server très simple avec la fonctionnalité de conteneur Windows. Il s’agit d’une rubrique avancée qui suppose des connaissances globales de Windows et des conteneurs Windows. Pour obtenir une présentation des conteneurs Windows, voir [Démarrage rapide des conteneurs Windows](../quick_start/quick_start.md).

## Préparer Nano Server

La section suivante décrit en détail le déploiement d’une configuration Nano Server très simple. Pour obtenir une description plus complète des options de déploiement et de configuration pour Nano Server, voir [Prise en main de Nano Server] (https://technet.microsoft.com/en-us/library/mt126167.aspx).

### Créer une machine virtuelle Nano Server

Tout d’abord, téléchargez le disque dur virtuel d’évaluation de Nano Server à partir de [cet emplacement](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula). Créez une machine virtuelle à partir de ce disque dur virtuel, démarrez la machine virtuelle et connectez-vous à celle-ci à l’aide de l’option de connexion Hyper-V, ou d’une option équivalente basée sur la plateforme de virtualisation utilisée.

Vous devez ensuite définir le mot de passe d’administration. Pour ce faire, appuyez sur `F11` sur la console de récupération Nano Server. La boîte de dialogue de modification du mot de passe s’affiche.

### Créer une session PowerShell distante

Comme Nano Server ne dispose pas de fonctionnalités de connexion interactives, toute la gestion s’effectue à partir d’une session PowerShell distante. Pour créer la session distante, obtenez l’adresse IP du système à l’aide de la section de mise en réseau de la console de récupération Nano Server, puis exécutez les commandes suivantes sur l’hôte distant. Remplacez IPADDRESS par l’adresse IP réelle du système Nano Server.

Ajoutez le système Nano Server aux hôtes approuvés.

```none
set-item WSMan:\localhost\Client\TrustedHosts IPADDRESS -Force
```

Créez la session PowerShell distante.

```none
Enter-PSSession -ComputerName IPADDRESS -Credential ~\Administrator
```

Une fois ces étapes terminées, une session PowerShell distante est établie avec le système Nano Server. Le reste de ce document, sauf indication contraire, se déroule à partir de la session distante.


## Installer la fonctionnalité de conteneur

Le fournisseur de gestion des packages Nano Server permet l’installation des rôles et fonctionnalités sur Nano Server. Installez le fournisseur à l’aide de cette commande.

```none
Install-PackageProvider NanoServerPackage
```

Une fois le fournisseur des packages installé, installez la fonctionnalité de conteneur.

```none
Install-NanoServerPackage -Name Microsoft-NanoServer-Containers-Package
```

L’hôte Nano Server doit être redémarré après l’installation des fonctionnalités de conteneur. 

```none
Restart-Computer
```

Une fois la sauvegarde effectuée, rétablissez la connexion PowerShell distante.

## Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. Installez le client et le démon Docker en suivant ces étapes.

Créez un dossier sur l’hôte Nano Server pour les exécutables Docker.

```none
New-Item -Type Directory -Path $env:ProgramFiles'\docker\'
```

Téléchargez le client et le démon Docker, puis copiez-les dans 'C:\Program Files\docker\' sur l’hôte de conteneur. 

**Remarque :** Nano Server ne prenant pas en charge `Invoke-WebRequest`, les téléchargements doivent être effectués à partir d’un système distant, puis copiés sur l’hôte Nano Server.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Téléchargez le client Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile .\docker.exe
```

Une fois que le client et le démon Docker ont été téléchargés, copiez-les dans le dossier C:\Program Files\docker\' dans l’hôte de conteneur Nano Server. Vous devez configurer le pare-feu Nano Server pour autoriser les connexions SMB entrantes. Pour cela, utilisez PowerShell ou la console de récupération Nano Server. 

```none
Set-NetFirewallRule -Name FPS-SMB-In-TCP -Enabled True
```

Les fichiers peuvent maintenant être copiés à l’aide des méthodes de copie de fichiers SMB standard.

Le fichier dockerd.exe étant copié sur l’hôte, exécutez cette commande pour installer Docker en tant que service Windows.

```none
& $env:ProgramFiles'\docker\dockerd.exe' --register-service
```

Démarrez le service Docker.

```none
Start-Service Docker
```

## Installer les images de conteneur de base

Les images de système d’exploitation de base sont utilisées comme base pour un conteneur Windows Server ou Hyper-V. Les images de système d’exploitation de base sont disponibles avec Windows Server Core et Nano Server comme système d’exploitation sous-jacent et peuvent être installées à l’aide du fournisseur d’images de conteneur. Pour plus d’informations sur les images de conteneur Windows, voir la rubrique relative à la [gestion des images de conteneur](../management/manage_images.md).

La commande suivante peut être utilisée pour installer le fournisseur d’images de conteneur.

```none
Install-PackageProvider ContainerImage -Force
```

Pour télécharger et installer l’image de base Nano Server, exécutez la commande suivante :

```none
Install-ContainerImage -Name NanoServer
```

**Remarque** : Pour l’instant, seule l’image de base Nano Server est compatible avec un hôte de conteneur Nano Server.

Redémarrez le service Docker.

```none
Restart-Service Docker
```

Marquez l’image de base Nano Server comme étant la dernière.

```none
& $env:ProgramFiles'\docker\docker.exe' tag nanoserver:10.0.14300.1016 nanoserver:latest
```

## Gérer Docker sur Nano Server

Pour une expérience optimale et une bonne pratique, gérez Docker sur Nano Server à partir d’un système distant. Pour ce faire, vous devez effectuer les opérations suivantes.

### Préparer l’hôte de conteneur

Créez une règle de pare-feu sur l’hôte de conteneur pour la connexion Docker. Il s’agit du port `2375` pour une connexion non sécurisée ou du port `2376` pour une connexion sécurisée.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

Configurez le démon Docker pour accepter une connexion entrante via TCP.

Tout d’abord, créez un fichier `daemon.json` dans `c:\ProgramData\docker\config\daemon.json` sur l’hôte Nano Server.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Exécutez ensuite la commande suivante pour ajouter la configuration de connexion au fichier `daemon.json`. Vous configurez ainsi le démon Docker pour accepter les connexions entrantes sur le port TCP 2375. Cette connexion n’est pas sécurisée et est déconseillée, mais peut être utilisée pour un test isolé. Pour plus d’informations sur la sécurisation de cette connexion, voir [Protéger le démon Docker sur Docker.com](https://docs.docker.com/engine/security/https/).

```none
Add-Content 'c:\programdata\docker\config\daemon.json' '{ "hosts": ["tcp://0.0.0.0:2375", "npipe://"] }'
```

Redémarrez le service Docker.

```none
Restart-Service docker
```

### Préparer le client distant

Sur le système distant sur lequel vous allez travailler, créez un répertoire pour le client Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Téléchargez le client Docker dans ce répertoire.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile "$env:ProgramFiles\docker\docker.exe"
```

Ajoutez le répertoire Docker au chemin du système.

```none
$env:Path += ";$env:ProgramFiles\Docker"
```

Redémarrez la session de commande ou PowerShell pour que le chemin modifié soit reconnu.

Quand vous avez terminé, l’hôte Docker distant est accessible avec le paramètre `docker -H`.

```none
docker -H tcp://<IPADDRESS>:2375 run -it nanoserver cmd
```

Une variable d’environnement `DOCKER_HOST` peut être créée pour supprimer la nécessité du paramètre `-H`. Vous pouvez pour cela exécuter la commande PowerShell suivante.

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

Quand cette variable est définie, la commande ressemble maintenant à ceci.

```none
docker run -it nanoserver cmd
```

## Hôte de conteneurs Hyper-V

Pour déployer des conteneurs Hyper-V, le rôle Hyper-V est nécessaire sur l’hôte de conteneur. Pour plus d’informations sur les conteneurs Hyper-V, voir [Conteneurs Hyper-V](../management/hyperv_container.md).

Si l’hôte de conteneur Windows est lui-même une machine virtuelle Hyper-V, la virtualisation imbriquée doit être activée. Pour plus d’informations sur la virtualisation imbriquée, voir [Virtualisation imbriquée](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).


Installez le rôle Hyper-V sur l’hôte de conteneur Nano Server.

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

L’hôte Nano Server doit être redémarré après l’installation du rôle Hyper-V.

```none
Restart-Computer
```


<!--HONumber=Jul16_HO1-->



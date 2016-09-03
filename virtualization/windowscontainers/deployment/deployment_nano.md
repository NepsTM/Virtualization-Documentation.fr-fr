---
title: "Déployer des conteneurs Windows sur Nano Server"
description: "Déployer des conteneurs Windows sur Nano Server"
keywords: docker, conteneurs
author: neilpeterson
manager: timlt
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: 2319649d1dd39677e59a9431fbefaf82982492c6
ms.openlocfilehash: a79469987879656117812ff6f9563046584172c0

---

# Déploiement d’un hôte de conteneur – Nano Server

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

Ce document présente les étapes d’un déploiement Nano Server très simple avec la fonctionnalité de conteneur Windows. Il s’agit d’une rubrique avancée qui suppose des connaissances globales de Windows et des conteneurs Windows. Pour obtenir une présentation des conteneurs Windows, voir [Démarrage rapide des conteneurs Windows](../quick_start/quick_start.md).

## Préparer Nano Server

La section suivante décrit en détail le déploiement d’une configuration Nano Server très simple. Pour obtenir une description plus complète des options de déploiement et de configuration pour Nano Server, voir [Prise en main de Nano Server] (https://technet.microsoft.com/en-us/library/mt126167.aspx).

### Créer une machine virtuelle Nano Server

Tout d’abord, téléchargez le disque dur virtuel d’évaluation de Nano Server à partir de [cet emplacement](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula). Créez une machine virtuelle à partir de ce disque dur virtuel, démarrez la machine virtuelle et connectez-vous à celle-ci à l’aide de l’option de connexion Hyper-V, ou d’une option équivalente basée sur la plateforme de virtualisation utilisée.

### Créer une session PowerShell distante

Comme Nano Server ne dispose pas de fonctionnalités de connexion interactives, toute la gestion s’effectue à partir d’un système distant utilisant PowerShell.

Ajoutez le système Nano Server aux hôtes approuvés du système distant. Remplacez l'adresse IP par l'adresse IP de Nano Server.

```none
Set-Item WSMan:\localhost\Client\TrustedHosts 192.168.1.50 -Force
```

Créez la session PowerShell distante.

```none
Enter-PSSession -ComputerName 192.168.1.50 -Credential ~\Administrator
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

Le moteur Docker est nécessaire pour utiliser les conteneurs Windows. Installez le moteur Docker en suivant les étapes suivantes.

Tout d’abord, assurez-vous que le pare-feu de Nano Server a été configuré pour SMB. Pour cela, exécutez la commande suivante sur l’hôte Nano Server.

```none
Set-NetFirewallRule -Name FPS-SMB-In-TCP -Enabled True
```

Créez un dossier sur l’hôte Nano Server pour les exécutables Docker.

```none
New-Item -Type Directory -Path $env:ProgramFiles'\docker\'
```

Téléchargez le client et le moteur Docker, puis copiez-les dans 'C:\Program Files\docker\' sur l’hôte de conteneur. 

> Nano Server ne prend actuellement pas en charge `Invoke-WebRequest`. Le téléchargement doit être effectué sur un système distant, et les fichiers copiés sur l’hôte Nano Server.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile .\docker-1.12.0.zip -UseBasicParsing
```

Effectuez l’extraction du package téléchargé. Une fois l’extraction terminée, vous disposerez d’un répertoire contenant **dockerd.exe** et **docker.exe**. Copiez ces deux fichiers dans le dossier **C:\Program Files\docker\** dans l’hôte du conteneur Nano Server. 

```none
Expand-Archive .\docker-1.12.0.zip
```

Ajoutez le répertoire Docker au chemin du système dans Nano Server.

> Veillez à revenir à la session Nano Server à distance.

```none
# for quick use, does not require shell to be restarted
$env:path += “;C:\program files\docker”

# for persistent use, will apply even after a reboot 
setx PATH $env:path /M
```

Installez Docker en tant que service Windows.

```none
dockerd --register-service
```

Démarrez le service Docker.

```none
Start-Service Docker
```

## Installer les images de conteneur de base

Les images de système d’exploitation de base sont utilisées comme base pour un conteneur Windows Server ou Hyper-V. Les images de système d’exploitation de base sont disponibles avec Windows Server Core et Nano Server comme système d’exploitation sous-jacent et peuvent être installées à l’aide de `docker pull`. Pour plus d’informations sur les images de conteneur Windows, voir la rubrique relative à la [gestion des images de conteneur](../management/manage_images.md).

Pour télécharger et installer l’image de base Nano Server, exécutez la commande suivante :

```none
docker pull microsoft/nanoserver
```

> Pour l’instant, seule l’image de base Nano Server est compatible avec un hôte de conteneur Nano Server.

## Gérer Docker sur Nano Server

Pour une expérience optimale et une bonne pratique, gérez Docker sur Nano Server à partir d’un système distant. Pour ce faire, vous devez effectuer les opérations suivantes.

### Préparer l’hôte de conteneur

Créez une règle de pare-feu sur l’hôte de conteneur pour la connexion Docker. Il s’agit du port `2375` pour une connexion non sécurisée ou du port `2376` pour une connexion sécurisée.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2375
```

Configurez le moteur Docker pour accepter une connexion entrante via TCP.

Tout d’abord, créez un fichier `daemon.json` dans `c:\ProgramData\docker\config\daemon.json` sur l’hôte Nano Server.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Exécutez ensuite la commande suivante pour ajouter la configuration de connexion au fichier `daemon.json`. Vous configurez ainsi le moteur Docker pour accepter les connexions entrantes sur le port TCP 2375. Cette connexion n’est pas sécurisée et est déconseillée, mais peut être utilisée pour un test isolé. Pour plus d’informations sur la sécurisation de cette connexion, voir [Protéger le démon Docker sur Docker.com](https://docs.docker.com/engine/security/https/).

```none
Add-Content 'c:\programdata\docker\config\daemon.json' '{ "hosts": ["tcp://0.0.0.0:2375", "npipe://"] }'
```

Redémarrez le service Docker.

```none
Restart-Service docker
```

### Préparer le client distant

Sur le système distant sur lequel vous allez travailler, téléchargez le client Docker.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Extrayez le package compressé.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Exécutez les deux commandes suivantes pour ajouter le répertoire Docker au chemin d’accès système.

```none
# for quick use, does not require shell to be restarted
$env:path += ";c:\program files\docker"

# for persistent use, will apply even after a reboot 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

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


<!--HONumber=Aug16_HO4-->



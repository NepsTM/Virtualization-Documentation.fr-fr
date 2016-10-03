---
title: "Déployer des conteneurs Windows sur Nano Server"
description: "Déployer des conteneurs Windows sur Nano Server"
keywords: docker, conteneurs
author: neilpeterson
manager: timlt
ms.date: 09/28/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: df9723e3a9d9ada778d01d43dcb36c99813dea8f
ms.openlocfilehash: 9af33e6bce21aa339109f060100b2c7ab3c1eb91

---

# Déploiement d’un hôte de conteneur – Nano Server

Ce document présente les étapes d’un déploiement Nano Server très simple avec la fonctionnalité de conteneur Windows. Il s’agit d’une rubrique avancée qui suppose des connaissances globales de Windows et des conteneurs Windows. Pour obtenir une présentation des conteneurs Windows, voir [Démarrage rapide des conteneurs Windows](../quick_start/quick_start.md).

## Préparer Nano Server

La section suivante décrit en détail le déploiement d’une configuration Nano Server très simple. Pour obtenir une description plus complète des options de déploiement et de configuration pour Nano Server, voir [Prise en main de Nano Server] (https://technet.microsoft.com/en-us/library/mt126167.aspx).

### Créer une machine virtuelle Nano Server

Tout d’abord, téléchargez le disque dur virtuel d’évaluation de Nano Server à partir de [cet emplacement](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016). Créez une machine virtuelle à partir de ce disque dur virtuel, démarrez la machine virtuelle et connectez-vous à celle-ci à l’aide de l’option de connexion Hyper-V, ou d’une option équivalente basée sur la plateforme de virtualisation utilisée.

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

### Installer les mises à jour Windows

Les mises à jour critiques sont nécessaires au fonctionnement de la fonctionnalité Conteneur Windows. Ces mises à jour peuvent être installées en exécutant les commandes suivantes.

```none
$sess = New-CimInstance -Namespace root/Microsoft/Windows/WindowsUpdate -ClassName MSFT_WUOperationsSession
Invoke-CimMethod -InputObject $sess -MethodName ApplyApplicableUpdates
```

Une fois les mises à jour appliquées, redémarrez le système.

```none
Restart-Computer
```

Une fois la sauvegarde effectuée, rétablissez la connexion PowerShell distante.

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

Téléchargez le moteur Docker et le client au format d’archive zip.

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Développez l’archive zip dans Program Files, le contenu de l’archive est déjà dans le répertoire de docker.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Ajoutez le répertoire Docker au chemin du système dans Nano Server.

```none
# For quick use, does not require shell to be restarted.
$env:path += “;C:\program files\docker”

# For persistent use, will apply even after a reboot.
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

Les images de système d’exploitation de base sont utilisées comme base pour un conteneur Windows Server ou Hyper-V. Les images de système d’exploitation de base sont disponibles avec Windows Server Core et Nano Server comme système d’exploitation sous-jacent et peuvent être installées à l’aide de `docker pull`. Pour plus d’informations sur les images de conteneur Docker, consultez [Build your own images (Créer vos propres images) sur docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

Pour télécharger et installer l’image de base Windows Server et Nano Server, exécutez les commandes suivantes.

```none
docker pull microsoft/nanoserver
```

```none
docker pull microsoft/windowsservercore
```

> Veuillez lire les termes du contrat de licence applicables à l’image de système d’exploitation des conteneurs Windows, disponibles [ici](../Images_EULA.md).

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
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Extrayez le package compressé.

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

Quand vous avez terminé, l’hôte Docker distant est accessible avec le paramètre `docker -H`.

```none
docker -H tcp://<IPADDRESS>:2375 run -it microsoft/nanoserver cmd
```

Une variable d’environnement `DOCKER_HOST` peut être créée pour supprimer la nécessité du paramètre `-H`. Vous pouvez pour cela exécuter la commande PowerShell suivante.

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

Quand cette variable est définie, la commande ressemble maintenant à ceci.

```none
docker run -it microsoft/nanoserver cmd
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



<!--HONumber=Sep16_HO5-->



---
title: "Déployer des conteneurs Windows sur Nano Server"
description: "Déployer des conteneurs Windows sur Nano Server"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/17/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: 1ba6af300d0a3eba3fc6d27598044f983a4c9168
ms.openlocfilehash: f2790186aa641378b1981a1f946665ca46fdbd73

---

# Déploiement d’un hôte de conteneur – Nano Server

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

Avant de commencer la configuration du conteneur Windows sur Nano Server, vous avez besoin d’un système exécutant Nano Server et également d’une connexion PowerShell à distance à ce système. Pour plus d’informations sur le déploiement et la connexion de Nano Server, voir [Prise en main de Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx).

Une copie d’évaluation de Nano Server est disponible [ici](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula).

## Installer la fonctionnalité de conteneur

Installez le fournisseur de gestion des packages Nano Server.

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

## Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. Installez le client et le démon Docker en suivant ces étapes.

Créez un dossier sur l’hôte Nano Server pour les exécutables Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Téléchargez le client et le démon Docker, puis copiez-les dans 'C:\Program Files\docker\' sur l’hôte de conteneur. 

**Remarque :** Nano Server ne prenant pas en charge `Invoke-WebRequest`, les téléchargements doivent être effectués à partir d’un système distant.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Téléchargez le client Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile .\docker.exe
```

Une fois que le client et le démon Docker ont été téléchargés et copiés dans l’hôte de conteneur Nano Server, exécutez cette commande sur l’hôte pour installer Docker comme un service Windows.

```none
& 'C:\Program Files\docker\dockerd.exe' --register-service
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
& 'C:\Program Files\docker\docker.exe' tag nanoserver:10.0.14300.1016 nanoserver:latest
```

## Gérer Docker sur Nano Server

Pour une expérience optimale et une bonne pratique, gérez Docker sur Nano Server à partir d’un système distant. Pour ce faire, vous devez effectuer les opérations suivantes.

**Préparez le démon Docker :**

Créez une règle de pare-feu sur l’hôte de conteneur pour la connexion Docker. Il s’agit du port `2375` pour une connexion non sécurisée ou du port `2376` pour une connexion sécurisée.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

Configurez le démon Docker pour accepter une connexion entrante via TCP.

Créez d’abord un fichier `daemon.json` sur `c:\ProgramData\docker\config\daemon.json`.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Ensuite, copiez ce JSON dans le fichier de configuration. Vous configurez ainsi le démon Docker pour accepter les connexions entrantes sur le port TCP 2375. Cette connexion n’est pas sécurisée et est déconseillée, mais peut être utilisée pour un test isolé.

```none
{
    "hosts": ["tcp://0.0.0.0:2375", "npipe://"]
}
```

L’exemple suivant configure une connexion à distance sécurisée. Les certificats TLS doivent être créés et copiés aux emplacements appropriés. Pour plus d’informations, voir [Démon Docker sur Windows](./docker_windows.md).

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

Redémarrez le service Docker.

```none
Restart-Service docker
```

**Préparez le client Docker :**

Créez un répertoire pour y placer le client Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Téléchargez le client Docker sur le système de gestion à distance.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile "C:\Program Files\docker\docker.exe"
```

Ajoutez le répertoire Docker au chemin du système.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Redémarrez la session de commande ou PowerShell pour que le chemin modifié soit reconnu.

Quand vous avez terminé, l’hôte Docker distant est accessible avec le paramètre `docker -H`.

```none
docker -H tcp://10.0.0.5:2375 run -it nanoserver cmd
```

Une variable d’environnement `DOCKER_HOST` peut être créée pour supprimer la nécessité du paramètre `-H`. Vous pouvez pour cela exécuter la commande PowerShell suivante.

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2375"
```

Quand cette variable est définie, la commande ressemble maintenant à ceci.

```none
docker run -it nanoserver cmd
```

## Hôte de conteneurs Hyper-V

Pour déployer des conteneurs Hyper-V, le rôle Hyper-V est nécessaire. Pour plus d’informations sur les conteneurs Hyper-V, voir [Conteneurs Hyper-V](../management/hyperv_container.md).

Si l’hôte de conteneur Windows est lui-même une machine virtuelle Hyper-V, la virtualisation imbriquée doit être activée. Pour plus d’informations sur la virtualisation imbriquée, voir [Virtualisation imbriquée](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).


Installez le rôle Hyper-V :

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

L’hôte Nano Server doit être redémarré après l’installation du rôle Hyper-V.

```none
Restart-Computer
```






<!--HONumber=Jun16_HO4-->



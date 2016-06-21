---
title: Déployer des conteneurs Windows sur Nano Server
description: Déployer des conteneurs Windows sur Nano Server
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
---

# Déploiement d’un hôte de conteneur – Nano Server

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

Avant de commencer la configuration du conteneur Windows sur Nano Server, vous avez besoin d’un système exécutant Nano Server et également d’une connexion PowerShell à distance à ce système.

Pour plus d’informations sur le déploiement et la connexion de Nano Server, voir [Prise en main de Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx).

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

L’hôte Nano Server doit être redémarré après l’installation de ces fonctionnalités.

## Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. Installez le démon Docker en suivant ces étapes.

Téléchargez le démon Docker et copiez-le dans `$env:SystemRoot\system32\` de l’hôte de conteneur. Nano Server ne prenant pas en charge `Invoke-Webrequest`, cette opération doit être effectuée à partir d’un système distant.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Installez Docker en tant que service Windows.

```none
dockerd.exe --register-service
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

Enfin, l’image doit être marquée avec une version de la balise « latest ». Pour ce faire, exécutez la commande suivante.

```none
docker tag nanoserver:10.0.14300.1010 nanoserver:latest
```

## Hôte de conteneurs Hyper-V

Pour déployer des conteneurs Hyper-V, le rôle Hyper-V est nécessaire. Si l’hôte de conteneur Windows est lui-même une machine virtuelle Hyper-V, la virtualisation imbriquée doit être activée avant d’installer le rôle Hyper-V. Pour plus d’informations sur la virtualisation imbriquée, voir Virtualisation imbriquée.

### Virtualisation imbriquée

Le script suivant configure la virtualisation imbriquée pour l’hôte de conteneur. Ce script est exécuté sur l’ordinateur Hyper-V qui héberge la machine virtuelle de l’hôte de conteneur. Vérifiez que la machine virtuelle de l’hôte de conteneur est désactivée lors de l’exécution de ce script.

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

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

## Gérer Docker sur Nano Server

**Préparez le démon Docker :**

Pour une expérience optimale, gérez Docker sur Nano Server à partir d’un système distant. Pour ce faire, vous devez effectuer les opérations suivantes.

Créez une règle de pare-feu sur l’hôte de conteneur pour la connexion Docker. Il s’agit du port `2375` pour une connexion non sécurisée ou du port `2376` pour une connexion sécurisée.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

Configurez le démon Docker pour accepter une connexion entrante via TCP.

Créez d’abord un fichier `daemon.json` sur `c:\ProgramData\docker\config\daemon.json`.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Ensuite, copiez ce JSON dans le fichier. Vous configurez ainsi le démon Docker pour accepter les connexions entrantes sur le port TCP 2375. Cette connexion n’est pas sécurisée et est déconseillée, mais peut être utilisée pour un test isolé.

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

Téléchargez le client Docker sur le système de gestion à distance.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:SystemRoot\system32\docker.exe
```

Une fois le téléchargement terminé, le démon Docker est accessible avec le paramètre `Docker -H`.

```none
docker -H tcp://10.0.0.5:2376 run -it nanoserver cmd
```

Une variable d’environnement `DOCKER_HOST` peut être créée pour supprimer la nécessité du paramètre `-H`. Vous pouvez pour cela exécuter la commande PowerShell suivante.

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2376"
```

Quand cette variable est définie, la commande ressemble maintenant à ceci.

```none
docker run -it nanoserver cmd
```

<!--HONumber=May16_HO5-->



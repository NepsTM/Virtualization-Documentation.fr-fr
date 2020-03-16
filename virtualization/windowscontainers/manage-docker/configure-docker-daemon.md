---
title: Configurer Docker dans Windows
description: Configurer Docker dans Windows
keywords: docker, conteneurs
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: c84a6652b5918238ee8ef6e1fa7a9b2aa596aefd
ms.sourcegitcommit: 8eedfdc1fda9d0abb36e28dc2b5fb39891777364
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/15/2020
ms.locfileid: "79402880"
---
# <a name="docker-engine-on-windows"></a>Moteur Docker sur Windows

Le moteur et le client Dockr ne sont pas inclus avec Windows et doivent être installés et configurés individuellement. De plus, le moteur Docker accepte de nombreuses configurations personnalisées. Certains exemples incluent la configuration de la façon dont le démon accepte les requêtes entrantes, les options de mise en réseau par défaut et les paramètres de débogage/du journal. Sur Windows, ces configurations peuvent être spécifiées dans un fichier de configuration ou à l’aide du Gestionnaire de contrôle des services Windows. Ce document décrit en détail comment installer et configurer le moteur de l’ancrage et fournit également quelques exemples de configurations couramment utilisées.

## <a name="install-docker"></a>Installer Docker

Pour utiliser des conteneurs Windows, vous devez disposer d’un ancrage. Docker comprend le moteur Docker (dockerd.exe) et le client Docker (docker.exe). Le moyen le plus simple d’obtenir tout ce qui est installé est dans le Guide de démarrage rapide, qui vous permet d’obtenir tout ce que vous allez configurer et d’exécuter votre premier conteneur.

- [Installer Docker](../quick-start/set-up-environment.md)

Pour les installations à l’aide de scripts, consultez [utiliser un script pour installer dockr EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Avant de pouvoir utiliser l’ancrage, vous devez installer les images de conteneur. Pour plus d’informations, consultez [docs pour les images de base de votre conteneur](../manage-containers/container-base-images.md).

## <a name="configure-docker-with-a-configuration-file"></a>Configurer l’arrimeur avec un fichier de configuration

La méthode privilégiée pour configurer le moteur Docker sur Windows consiste à utiliser un fichier de configuration. Ce fichier de configuration se trouve dans C:\ProgramData\Docker\config\daemon.json. Vous pouvez créer ce fichier s’il n’existe pas déjà.

>[!NOTE]
>Toutes les options de configuration de l’arrimeur disponibles ne s’appliquent pas à l’ancrage sur Windows. L’exemple suivant montre les options de configuration qui s’appliquent. Pour plus d’informations sur la configuration du moteur de l’ancrage, consultez le [fichier de configuration du démon](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)de l’ancrage.

```json
{
    "authorization-plugins": [],
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "storage-driver": "",
    "storage-opts": [],
    "labels": [],
    "log-driver": "", 
    "mtu": 0,
    "pidfile": "",
    "data-root": "",
    "cluster-store": "",
    "cluster-advertise": "",
    "debug": true,
    "hosts": [],
    "log-level": "",
    "tlsverify": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "group": "",
    "default-ulimits": {},
    "bridge": "",
    "fixed-cidr": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "insecure-registries": [],
    "disable-legacy-registry": false
}
```

Il vous suffit d’ajouter les modifications de configuration souhaitées au fichier de configuration. Par exemple, l’exemple suivant configure le moteur de l’ancrage pour qu’il accepte les connexions entrantes sur le port 2375. Toutes les autres options de configuration utiliseront les valeurs par défaut.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

De même, l’exemple suivant configure le démon de la station d’accueil pour conserver les images et les conteneurs dans un autre chemin d’accès. S’il n’est pas spécifié, la valeur par défaut est `c:\programdata\docker`.

```json
{    
    "data-root": "d:\\docker"
}
```

L’exemple suivant configure le démon de l’arrimeur pour qu’il n’accepte que les connexions sécurisées sur le port 2376.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Configurer l’arrimeur sur le service d’ancrage

Le moteur de l’ancrage peut également être configuré en modifiant le service d’ancrage avec `sc config`. Avec cette méthode, les indicateurs du moteur Docker sont définis directement sur le service de Docker. Exécutez la commande suivante dans une invite de commandes (cmd.exe, pas PowerShell) :

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>Vous n’avez pas besoin d’exécuter cette commande si votre fichier daemon. JSON contient déjà l’entrée `"hosts": ["tcp://0.0.0.0:2375"]`.

## <a name="common-configuration"></a>Configuration commune

Les exemples de fichiers de configuration suivants présentent des configurations courantes de Docker. Elles peuvent être combinées en un seul fichier de configuration.

### <a name="default-network-creation"></a>Création de réseau par défaut

Pour configurer le moteur de l’ancrage afin qu’il ne crée pas de réseau NAT par défaut, utilisez la configuration suivante.

```json
{
    "bridge" : "none"
}
```

Pour plus d’informations, voir la rubrique indiquant comment [gérer les réseaux Docker](../container-networking/network-drivers-topologies.md).

### <a name="set-docker-security-group"></a>Définir le groupe de sécurité de l’arrimeur

Lorsque vous vous êtes connecté à l’hôte de l’ordinateur de la station d’accueil et que vous exécutez des commandes de l’arrimeur localement, ces commandes sont exécutées via un canal nommé. Par défaut, seuls les membres du groupe Administrateurs peuvent accéder au moteur Docker via le canal nommé. Pour spécifier un groupe de sécurité bénéficiant de cet accès, utilisez l’indicateur `group`.

```json
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>Configuration du proxy

Pour définir des informations de proxy pour `docker search` et `docker pull`, créez une variable d’environnement Windows nommée `HTTP_PROXY` ou `HTTPS_PROXY`, et une valeur des informations de proxy. Vous pouvez effectuer cette opération dans PowerShell en utilisant une commande semblable à celle-ci :

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

Une fois que la variable a été définie, redémarrez le service Docker.

```powershell
Restart-Service docker
```

Pour plus d’informations, consultez [fichier de configuration Windows sur docker.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="how-to-uninstall-docker"></a>Comment désinstaller l’ancrage

Cette section vous indique comment désinstaller Dockr et effectuer un nettoyage complet des composants système de l’ordinateur d’amarrage à partir de votre système Windows 10 ou Windows Server 2016.

>[!NOTE]
>Vous devez exécuter toutes les commandes de ces instructions à partir d’une session PowerShell avec élévation de privilèges.

### <a name="prepare-your-system-for-dockers-removal"></a>Préparer votre système pour la suppression de l’ancrage

Avant de désinstaller Dockr, assurez-vous qu’aucun conteneur n’est en cours d’exécution sur votre système.

Exécutez les applets de commande suivantes pour vérifier les conteneurs en cours d’exécution :

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

Il est également conseillé de supprimer tous les conteneurs, les images de conteneur, les réseaux et les volumes de votre système avant de supprimer Dockr. Pour ce faire, exécutez l’applet de commande suivante :

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Désinstallation de Docker

Ensuite, vous devez désinstaller la station d’accueil.

Pour désinstaller l’amarrage sur Windows 10

- Accédez à **paramètres** > **applications** sur votre ordinateur Windows 10.
- Sous **applications & fonctionnalités**, recherchez **docker pour Windows**
- Accédez à **Docker pour Windows** > **désinstaller**

Pour désinstaller l’amarrage sur Windows Server 2016 :

À partir d’une session PowerShell avec élévation de privilèges, utilisez les applets de commande **Uninstall-package** et **Uninstall-Module** pour supprimer le module d’ancrage et son fournisseur de Package Management correspondant de votre système, comme illustré dans l’exemple suivant :

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Vous pouvez trouver le fournisseur de package que vous avez utilisé pour installer l’amarrage avec `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Nettoyer les données et les composants système de l’ancrage

Une fois que vous avez désinstallé la station d’accueil, vous devez supprimer les réseaux par défaut de l’arrimeur afin que leur configuration ne reste pas sur votre système une fois que l’ordinateur a disparu. Pour ce faire, exécutez l’applet de commande suivante :

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Exécutez l’applet de commande suivante pour supprimer les données de programme de l’ancrage de votre système :

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Vous pouvez également supprimer les fonctionnalités Windows facultatives associées à l’ancrage/aux conteneurs sur Windows.

Cela comprend la fonctionnalité « conteneurs », qui est automatiquement activée sur n’importe quel ordinateur Windows 10 ou Windows Server 2016 quand l’outil de connexion est installé. Cela peut également inclure la fonctionnalité « Hyper-V », qui est automatiquement activée sur Windows 10 lorsque Docker est installé, mais qui doit être activée explicitement sur Windows Server 2016.

>[!IMPORTANT]
>[La fonctionnalité Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/) est une fonctionnalité de virtualisation générale qui active bien plus qu’un simple conteneur. Avant de désactiver la fonctionnalité Hyper-V, assurez-vous qu’il n’y a pas d’autres composants virtualisés sur votre système qui nécessitent Hyper-V.

Pour supprimer les fonctionnalités Windows sur Windows 10 :

- Accédez au **panneau de configuration** > **programmes** > **programmes et fonctionnalités** > **activer ou désactiver des fonctionnalités Windows**.
- Recherchez le nom de la fonctionnalité ou des fonctionnalités que vous souhaitez désactiver, dans ce cas, les **conteneurs** et (éventuellement) **Hyper-V**.
- Désactivez la case à cocher en regard du nom de la fonctionnalité que vous souhaitez désactiver.
- Sélectionnez **« OK »** .

Pour supprimer les fonctionnalités Windows sur Windows Server 2016 :

À partir d’une session PowerShell avec élévation de privilèges, exécutez les applets de commande suivantes pour désactiver les **conteneurs** et (éventuellement) les fonctionnalités **Hyper-V** de votre système :

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>Redémarrez votre système

Pour terminer la désinstallation et le nettoyage, exécutez l’applet de commande suivante à partir d’une session PowerShell avec élévation de privilèges pour redémarrer votre système :

```powershell
Restart-Computer -Force
```

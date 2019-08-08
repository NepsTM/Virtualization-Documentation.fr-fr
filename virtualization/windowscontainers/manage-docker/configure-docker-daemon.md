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
ms.openlocfilehash: 953dfaf71170de656f4e6ba5e91d524708d5a12a
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998216"
---
# <a name="docker-engine-on-windows"></a>Moteur Docker sur Windows

Le moteur et le client d’ancrage ne sont pas inclus dans Windows et doivent être installés et configurés individuellement. De plus, le moteur Docker accepte de nombreuses configurations personnalisées. Certains exemples incluent la configuration de la façon dont le démon accepte les requêtes entrantes, les options de mise en réseau par défaut et les paramètres de débogage/du journal. Sur Windows, ces configurations peuvent être spécifiées dans un fichier de configuration ou à l’aide du Gestionnaire de contrôle des services Windows. Ce document fournit des détails sur l’installation et la configuration du moteur de l’ancrage, ainsi que des exemples de configurations couramment utilisées.

## <a name="install-docker"></a>Installer Docker

Pour utiliser les conteneurs Windows, vous devez disposer de l’arrimeur. Docker comprend le moteur Docker (dockerd.exe) et le client Docker (docker.exe). Le moyen le plus simple d’obtenir tous les éléments installés figure dans les guides de démarrage rapide, qui vous aideront à configurer tous les éléments et à exécuter votre premier conteneur.

- [Conteneurs Windows sur Windows Server 2019](../quick-start/quick-start-windows-server.md)
- [Conteneurs Windows sur Windows 10](../quick-start/quick-start-windows-10.md)

Pour les installations par script, reportez-vous [à la rubrique utiliser un script pour installer le docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Pour pouvoir utiliser la station d’accueil, vous devez installer les images du conteneur. Pour plus d’informations, consultez [le Guide de démarrage rapide sur l’utilisation d’images](../quick-start/quick-start-images.md).

## <a name="configure-docker-with-a-configuration-file"></a>Configurer l’ancrage avec un fichier de configuration

La méthode privilégiée pour configurer le moteur Docker sur Windows consiste à utiliser un fichier de configuration. Ce fichier de configuration se trouve dans C:\ProgramData\Docker\config\daemon.json. Vous pouvez créer ce fichier s’il n’existe pas déjà.

>[!NOTE]
>Toutes les options de configuration de l’ancrage disponibles ne s’appliquent pas à l’ancrage sur Windows. L’exemple ci-après illustre les options de configuration qui s’appliquent. Pour plus d’informations sur la configuration du moteur de l’ancrage, voir [fichier de configuration du démon d’amarrage](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

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

Il vous suffit d’ajouter les modifications de configuration souhaitées au fichier de configuration. Par exemple, l’exemple suivant configure le moteur de l’amarrage pour accepter les connexions entrantes sur le port 2375. Toutes les autres options de configuration utiliseront les valeurs par défaut.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

De même, l’exemple suivant configure le démon de l’ancrage pour conserver les images et les conteneurs dans un autre chemin. Si cette valeur n’est pas spécifiée `c:\programdata\docker`, la valeur par défaut est.

```json
{    
    "data-root": "d:\\docker"
}
```

L’exemple suivant configure le démon d’ancrage de manière à ce qu’il accepte uniquement les connexions sécurisées sur le port 2376.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Configurer l’ancrage sur le service d’ancrage

Le moteur de l’ancrage peut également être configuré en modifiant le service d' `sc config`amarrage. Avec cette méthode, les indicateurs du moteur Docker sont définis directement sur le service de Docker. Exécutez la commande suivante dans une invite de commandes (cmd.exe, pas PowerShell):

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>Vous n’avez pas besoin d’exécuter cette commande si votre fichier daemon. JSON contient `"hosts": ["tcp://0.0.0.0:2375"]` déjà l’entrée.

## <a name="common-configuration"></a>Configuration courante

Les exemples de fichiers de configuration suivants présentent des configurations courantes de Docker. Elles peuvent être combinées en un seul fichier de configuration.

### <a name="default-network-creation"></a>Création de réseau par défaut

Pour configurer le moteur de l’ancrage de sorte qu’il ne crée pas un réseau NAT par défaut, utilisez la configuration suivante.

```json
{
    "bridge" : "none"
}
```

Pour plus d’informations, voir la rubrique indiquant comment [gérer les réseaux Docker](../container-networking/network-drivers-topologies.md).

### <a name="set-docker-security-group"></a>Définir le groupe de sécurité de l’ancrage

Lorsque vous vous êtes connecté à l’hôte de l’ordinateur de Dock et que vous exécutez des commandes de l’amarrage en local, ces commandes sont exécutées par le biais d’un canal nommé. Par défaut, seuls les membres du groupe Administrateurs peuvent accéder au moteur Docker via le canal nommé. Pour spécifier un groupe de sécurité bénéficiant de cet accès, utilisez l’indicateur `group`.

```json
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>Configuration du proxy

Pour définir des informations de proxy pour `docker search` et `docker pull`, créez une variable d’environnement Windows nommée `HTTP_PROXY` ou `HTTPS_PROXY`, et une valeur des informations de proxy. Vous pouvez effectuer cette opération dans PowerShell en utilisant une commande semblable à celle-ci:

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

Une fois que la variable a été définie, redémarrez le service Docker.

```powershell
Restart-Service docker
```

Pour plus d’informations, consultez [le fichier de configuration Windows sur docker.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="how-to-uninstall-docker"></a>Comment désinstaller l’ancrage

Cette section vous explique comment désinstaller l’ancrage et effectuer un nettoyage complet des composants du système d’amarrage à partir de votre système Windows 10 ou Windows Server 2016.

>[!NOTE]
>Vous devez exécuter toutes les commandes dans ces instructions à partir d’une session PowerShell avec élévation de privilèges.

### <a name="prepare-your-system-for-dockers-removal"></a>Préparer votre système pour la suppression de l’Arrimateur

Avant de désinstaller l’ancrage, assurez-vous qu’aucun conteneur n’est en cours d’exécution sur votre système.

Pour vérifier les conteneurs en cours d’exécution, exécutez les applets de commande suivantes:

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

Il est également conseillé de supprimer tous les conteneurs, images de conteneurs, réseaux et volumes de votre système avant d’enlever le Dock. Pour cela, exécutez l’applet de commande suivante:

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Désinstallation de Docker

Ensuite, vous devez désinstaller la station d’accueil.

Pour désinstaller l’ancrage sur Windows 10

- Accédez à **paramètres** > des**applications** sur votre ordinateur Windows 10.
- Sous **applications & fonctionnalités**, recherchez **docker pour Windows** .
- Accéder à la **fenêtre** > **** de désinstallation de l’amarrage

Pour désinstaller docker sur Windows Server 2016:

À partir d’une session PowerShell avec élévation de **** privilèges, vous pouvez utiliser les applets de commande de désinstallation et de désinstallation pour supprimer le module d’amarrage et son fournisseur de gestion de packages correspondant de votre système, comme le montre l’exemple suivant: ****

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Vous pouvez trouver le fournisseur de package que vous avez utilisé pour installer l’ancrage `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Nettoyer les données et composants système de l’ancrage

Une fois que vous avez désinstallé docker, vous devez supprimer les réseaux par défaut de l’ancrage pour que la configuration ne reste pas sur votre système après le retrait du Dock. Pour cela, exécutez l’applet de commande suivante:

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Exécutez l’applet de commande suivante pour supprimer les données du programme de l’ancrage de votre système:

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Vous pouvez également supprimer les fonctionnalités facultatives Windows associés aux conteneurs/Docker sur Windows.

Cela inclut la fonctionnalité «conteneurs», qui est activée automatiquement sur n’importe quel Windows 10 ou Windows Server 2016 lors de l’installation de l’outil de connexion. Cela peut également inclure la fonctionnalité «Hyper-V», qui est automatiquement activée sur Windows10 lorsque Docker est installé, mais qui doit être activée explicitement sur Windows Server2016.

>[!IMPORTANT]
>[La fonctionnalité Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/) est une fonctionnalité de virtualisation générale qui permet bien plus que des conteneurs. Avant de désactiver la fonctionnalité Hyper-V, assurez-vous qu’il n’y a pas d’autres composants virtualisés sur votre système qui requièrent Hyper-V.

Pour supprimer des fonctionnalités Windows sur Windows 10:

- Accédez à**programmes** >  **du panneau de configuration** > **, programmes et fonctionnalités** > **activez ou désactivez les fonctionnalités Windows**.
- Recherchez le nom de la ou des fonctionnalités que vous souhaitez désactiver (dans ce cas, **conteneurs** et (facultatif) **Hyper-V**).
- Décochez la case en regard du nom de la fonctionnalité que vous souhaitez désactiver.
- Sélectionner **"OK"**

Pour supprimer des fonctionnalités Windows sur Windows Server 2016:

À partir d’une session PowerShell avec élévation de privilèges, exécutez les applets de commande suivantes pour désactiver les **conteneurs** et (éventuellement) des fonctionnalités **Hyper-V** de votre système:

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>Redémarrer votre système

Pour terminer la désinstallation et le nettoyage, exécutez l’applet de commande suivante à partir d’une session PowerShell avec élévation de privilèges pour redémarrer votre système:

```powershell
Restart-Computer -Force
```

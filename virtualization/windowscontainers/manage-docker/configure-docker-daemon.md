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
ms.openlocfilehash: a04d356415e7bed84980747edc927cc1eaa1e7c1
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621087"
---
# <a name="docker-engine-on-windows"></a>Moteur Docker sur Windows

Le moteur Docker et le client ne sont pas inclus avec Windows et doivent être installés et configurés individuellement. De plus, le moteur Docker accepte de nombreuses configurations personnalisées. Certains exemples incluent la configuration de la façon dont le démon accepte les requêtes entrantes, les options de mise en réseau par défaut et les paramètres de débogage/du journal. Sur Windows, ces configurations peuvent être spécifiées dans un fichier de configuration ou à l’aide du Gestionnaire de contrôle des services Windows. Ce document explique en détail comment installer et configurer le moteur Docker et fournit également des exemples de configurations fréquemment utilisées.

## <a name="install-docker"></a>Installer Docker

Vous devez Docker pour utiliser les conteneurs Windows. Docker comprend le moteur Docker (dockerd.exe) et le client Docker (docker.exe). Pour obtenir tous les éléments installés, le plus simple consiste dans le démarrage rapide guides, ce qui vous aideront à tout configurer et exécutent votre premier conteneur.

- [Conteneurs Windows sur Windows Server 2019](../quick-start/quick-start-windows-server.md)
- [Conteneurs Windows sur Windows 10](../quick-start/quick-start-windows-10.md)

Pour les installations par script, voir [utilisation d’un script pour installer Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Avant de pouvoir utiliser Docker, vous devez installer les images de conteneur. Pour plus d’informations, voir [le guide de démarrage rapide pour l’utilisation d’images](../quick-start/quick-start-images.md).

## <a name="configure-docker-with-a-configuration-file"></a>Configurer Docker avec un fichier de configuration

La méthode privilégiée pour configurer le moteur Docker sur Windows consiste à utiliser un fichier de configuration. Ce fichier de configuration se trouve dans C:\ProgramData\Docker\config\daemon.json. Vous pouvez créer ce fichier s’il n’existe déjà.

>[!NOTE]
>Les options de configuration Docker sont pas toutes disponibles s’applique à Docker sur Windows. L’exemple suivant montre les options de configuration qui s’appliquent. Pour plus d’informations sur la configuration du moteur Docker, consultez le [fichier de configuration du démon Docker](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

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

Vous devez uniquement ajouter les modifications de configuration souhaitées dans le fichier de configuration. Par exemple, l’exemple suivant configure le moteur Docker pour accepter les connexions entrantes sur le port 2375. Toutes les autres options de configuration utiliseront les valeurs par défaut.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

De même, l’exemple suivant configure le démon Docker pour conserver les images et des conteneurs dans un autre chemin. Si vous n’est spécifié, la valeur par défaut est `c:\programdata\docker`.

```json
{    
    "data-root": "d:\\docker"
}
```

L’exemple suivant configure le démon Docker pour accepter uniquement les connexions sécurisées sur le port 2376.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Configurer Docker sur le service Docker

Le moteur Docker peut également être configuré en modifiant le service Docker avec `sc config`. Avec cette méthode, les indicateurs du moteur Docker sont définis directement sur le service de Docker. Exécutez la commande suivante dans une invite de commandes (cmd.exe, pas PowerShell):

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>Vous n’avez pas besoin d’exécuter cette commande si votre fichier daemon.json contient déjà le `"hosts": ["tcp://0.0.0.0:2375"]` entrée.

## <a name="common-configuration"></a>Configuration commune

Les exemples de fichiers de configuration suivants présentent des configurations courantes de Docker. Elles peuvent être combinées en un seul fichier de configuration.

### <a name="default-network-creation"></a>Création du réseau par défaut

Pour configurer le moteur Docker de sorte qu’il n’a pas de créer un réseau NAT par défaut, utilisez la configuration suivante.

```json
{
    "bridge" : "none"
}
```

Pour plus d’informations, voir la rubrique indiquant comment [gérer les réseaux Docker](../container-networking/network-drivers-topologies.md).

### <a name="set-docker-security-group"></a>Définir le groupe de sécurité Docker

Lorsque vous avez connecté à l’hôte Docker et exécutez les commandes Docker localement, ces commandes sont exécutées par le biais d’un canal nommé. Par défaut, seuls les membres du groupe Administrateurs peuvent accéder au moteur Docker via le canal nommé. Pour spécifier un groupe de sécurité bénéficiant de cet accès, utilisez l’indicateur `group`.

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

Pour plus d’informations, consultez [Le fichier de Configuration Windows sur Docker.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="how-to-uninstall-docker"></a>Procédure pour désinstaller Docker

Cette section vous indique comment désinstaller Docker et effectuer un nettoyage complet des composants système Docker de votre système Windows 10 ou Windows Server 2016.

>[!NOTE]
>Vous devez exécuter toutes les commandes dans les instructions ci-après à partir d’une session PowerShell avec élévation de privilèges.

### <a name="prepare-your-system-for-dockers-removal"></a>Préparer votre système pour la suppression de Docker

Avant de désinstaller Docker, assurez-vous qu’aucun conteneur n’est en cours d’exécution sur votre système.

Exécutez les applets de commande suivantes pour rechercher les conteneurs en cours d’exécution:

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

Il est également recommandé de supprimer tous les conteneurs, les images de conteneur, les réseaux et les volumes de votre système avant la suppression de Docker. Vous pouvez le faire en exécutant l’applet de commande suivante:

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Désinstallation de Docker

Ensuite, vous devez réellement désinstallation de Docker.

Pour désinstaller Docker sur Windows 10

- Accédez à **paramètres** > **applications** sur votre ordinateur Windows 10
- Sous **les applications & fonctionnalités**, recherchez **Docker pour Windows**
- Accédez à **Docker pour Windows** > **désinstaller**

Pour désinstaller Docker sur Windows Server 2016:

À partir d’une session PowerShell avec élévation de privilèges, utilisez les applets de commande de **Désinstallation-Package** et **Module de désinstallation** pour supprimer le module Docker et le fournisseur de gestion de packages correspondant à partir de votre système, comme illustré dans l’exemple suivant:

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Vous pouvez trouver le fournisseur du Package que vous avez utilisé pour installer Docker `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Nettoyer les composants système et de données de Docker

Après la désinstallation de Docker, vous devez supprimer les réseaux par défaut de Docker pour que leur configuration ne reste sur votre système une fois que Docker a disparu. Vous pouvez le faire en exécutant l’applet de commande suivante:

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Exécutez l’applet de commande suivante pour supprimer les données du programme de Docker de votre système:

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Vous pouvez également supprimer les fonctionnalités facultatives Windows associés aux conteneurs/Docker sur Windows.

Cela inclut la fonctionnalité «Conteneurs», qui est automatiquement activée sur Windows 10 ou Windows Server 2016 lorsque Docker est installé. Cela peut également inclure la fonctionnalité «Hyper-V», qui est automatiquement activée sur Windows10 lorsque Docker est installé, mais qui doit être activée explicitement sur Windows Server2016.

>[!IMPORTANT]
>[Fonctionnalité l’Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/) est une fonctionnalité de virtualisation générale bien plus que les conteneurs. Avant de désactiver la fonctionnalité Hyper-V, assurez-vous qu’il n’existe aucun autre composant virtualisé sur votre système qui nécessitent Hyper-V.

Pour supprimer les fonctionnalités de Windows sur Windows 10:

- Accédez à **Panneau** > **programmes** > **programmes et fonctionnalités** > **Windows activer ou désactiver des fonctionnalités**.
- Recherchez le nom de l’ou les fonctionnalités que vous souhaitez désactiver, en l’occurrence, **les conteneurs** et (éventuellement) **Hyper-V**.
- Décochez la case en regard du nom de la fonctionnalité que vous souhaitez désactiver.
- Sélectionnez **«OK»**

Pour supprimer les fonctionnalités de Windows sur Windows Server 2016:

À partir d’une session PowerShell avec élévation de privilèges, exécutez les applets de commande suivante pour désactiver les fonctionnalités **Hyper-V** à partir de votre système de **conteneurs** et (éventuellement):

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>Redémarrez votre système

Pour terminer la désinstallation et le nettoyage, exécutez l’applet de commande suivante à partir d’une session PowerShell avec élévation de privilèges à redémarrer votre système:

```powershell
Restart-Computer -Force
```

---
title: Configurer Docker dans Windows
description: Configurer Docker dans Windows
keywords: docker, conteneurs
author: PatrickLang
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: bbc405fc2a490cfe5082be112fde724707e24785
ms.sourcegitcommit: 21d93e5febd9b1b47ae1aa59d08086e6ec1691e0
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/28/2019
ms.locfileid: "9121051"
---
# <a name="docker-engine-on-windows"></a>Moteur Docker sur Windows

Le moteur Docker et le client ne sont pas inclus avec Windows et doivent être installés et configurés individuellement. De plus, le moteur Docker accepte de nombreuses configurations personnalisées. Certains exemples incluent la configuration de la façon dont le démon accepte les requêtes entrantes, les options de mise en réseau par défaut et les paramètres de débogage/du journal. Sur Windows, ces configurations peuvent être spécifiées dans un fichier de configuration ou à l’aide du Gestionnaire de contrôle des services Windows. Ce document explique en détail comment installer et configurer le moteur Docker et fournit également des exemples de configurations fréquemment utilisées.


## <a name="install-docker"></a>Installer Docker
Vous devez installer Docker pour utiliser les conteneurs Windows. Docker comprend le moteur Docker (dockerd.exe) et le client Docker (docker.exe). Vous trouverez le moyen le plus simple de tout installer dans les guides de démarrage rapide. Elles vous aident à rendre tout configurent et exécutent votre premier conteneur. 

* [Conteneurs Windows sur Windows Server 2019](../quick-start/quick-start-windows-server.md)
* [Conteneurs Windows sur Windows10](../quick-start/quick-start-windows-10.md)

Pour les installations par script, consultez [Utiliser un script pour installer Docker EE ](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Avant de pouvoir utiliser Docker images de conteneur doivent être installés. Pour plus d’informations, voir le [guide de démarrage rapide consacré à l’utilisation d’images](../quick-start/quick-start-images.md).

## <a name="configure-docker-with-configuration-file"></a>Configurer Docker avec un fichier de configuration

La méthode privilégiée pour configurer le moteur Docker sur Windows consiste à utiliser un fichier de configuration. Ce fichier de configuration se trouve dans C:\ProgramData\Docker\config\daemon.json. Si ce fichier n’existe pas encore, il peut être créé.

Remarque: Toutes les options de configuration de Docker disponibles ne s’appliquent pas à Docker sur Windows. L’exemple ci-dessous montre celles qui le sont. Pour obtenir une documentation complète sur la configuration du moteur Docker, voir la rubrique relative au [fichier de configuration du démon Docker](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

```
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

Seules les modifications de configuration souhaitées doivent être ajoutées au fichier de configuration. Ainsi, le présent exemple configure le moteur Docker pour accepter les connexions entrantes sur le port2375. Toutes les autres options de configuration utiliseront les valeurs par défaut.

```
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

De même, cet exemple configure le démon Docker pour conserver les images et les conteneurs dans un autre chemin. S’il n’est pas spécifié, la valeur par défaut est c:\programdata\docker.

```
{    
    "data-root": "d:\\docker"
}
```

De même, cet exemple configure le démon Docker pour accepter uniquement les connexions sécurisées sur le port2376.

```
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Configurer Docker sur le service Docker

Le moteur Docker peut également être configuré en modifiant le service de Docker avec `sc config`. Avec cette méthode, les indicateurs du moteur Docker sont définis directement sur le service de Docker. Exécutez la commande suivante dans une invite de commandes (cmd.exe, pas PowerShell):


```
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

Remarque: Vous n’avez pas besoin d’exécuter cette commande si votre fichier daemon.json contient déjà l’entrée `"hosts": ["tcp://0.0.0.0:2375"]`.

## <a name="common-configuration"></a>Configuration commune

Les exemples de fichiers de configuration suivants présentent des configurations courantes de Docker. Elles peuvent être combinées en un seul fichier de configuration.

### <a name="default-network-creation"></a>Création de réseau par défaut 

Pour configurer le moteur Docker de sorte qu’un réseau NAT par défaut ne soit pas créé, utilisez le code suivant. Pour plus d’informations, voir la rubrique indiquant comment [gérer les réseaux Docker](../container-networking/network-drivers-topologies.md).

```
{
    "bridge" : "none"
}
```

### <a name="set-docker-security-group"></a>Définir un groupe de sécurité Docker

Quand vous êtes connecté à l’hôte Docker et que vous exécutez des commandes Docker localement, ces commandes sont exécutées via un canal nommé. Par défaut, seuls les membres du groupe Administrateurs peuvent accéder au moteur Docker via le canal nommé. Pour spécifier un groupe de sécurité bénéficiant de cet accès, utilisez l’indicateur `group`.

```
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

Pour plus d’informations, consultez [Fichier de configuration Windows sur Docker.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="uninstall-docker"></a>Désinstallation de Docker
*Suivez les étapes de cette section pour désinstaller Docker et effectuer un nettoyage complet des composants système Docker de votre système Windows10 ou Windows Server2016.*

> Remarque: toutes les commandes dans les étapes ci-dessous doivent être exécutées à partir d’une session PowerShell **avec élévation de privilèges**.

### <a name="step-1-prepare-your-system-for-dockers-removal"></a>ÉTAPE1: Préparez votre système à la suppression de Docker 
Si vous ne l’avez pas déjà fait, il convient de vérifier qu’aucun conteneur n’est en cours d’exécution sur votre système avant la suppression de Docker. Voici quelques commandes utiles pour y parvenir:
```
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```
Il est également recommandé de supprimer tous les conteneurs, les images de conteneur, les réseaux et les volumes de votre système avant la suppression de Docker:
```
docker system prune --volumes --all
```

### <a name="step-2-uninstall-docker"></a>ÉTAPE2: Désinstallez Docker 

#### ***<a name="steps-to-uninstall-docker-on-windows-10"></a>Étapes de désinstallation de Docker sur Windows10:10:***
- Accédez à **"Paramètres" > "Applications"** sur votre ordinateur Windows10
- Sous **"Applications et fonctionnalités"**, recherchez **"Docker pour Windows"**
- Cliquez sur **«Docker pour Windows» > «Désinstaller»**

#### ***<a name="steps-to-uninstall-docker-on-windows-server-2016"></a>Étapes de désinstallation de Docker sur Windows Server2016:16:***
À partir d’une session PowerShell avec élévation de privilèges, utilisez les applets de commande `Uninstall-Package` et `Uninstall-Module` pour supprimer de votre système le module Docker et le fournisseur de gestion de packages correspondant. 
> Conseil: Vous pouvez chercher le fournisseur de packages que vous avez utilisé pour installer Docker `PS C:\> Get-PackageProvider -Name *Docker*`

*Par exemple*:
```
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

### <a name="step-3-cleanup-docker-data-and-system-components"></a>ÉTAPE3: Nettoyez les données et les composants système Docker
Supprimez les *réseaux par défaut* afin que leur configuration ne demeure pas sur votre système une fois Docker supprimé:
```
Get-HNSNetwork | Remove-HNSNetwork
```
Supprimez les *données programme* Docker de votre système:
```
Remove-Item "C:\ProgramData\Docker" -Recurse
```
Vous pouvez également supprimer les *fonctionnalités facultatives Windows* associés aux conteneurs/Docker sur Windows. 

Au minimum, cela inclut la fonctionnalité «Conteneurs», qui est automatiquement activée sur Windows10 ou Windows Server2016 lorsque Docker est installé. Cela peut également inclure la fonctionnalité «Hyper-V», qui est automatiquement activée sur Windows10 lorsque Docker est installé, mais qui doit être activée explicitement sur Windows Server2016.

> **REMARQUE IMPORTANT CONCERNANT LA DÉSACTIVATION D’HYPER-V:**[ la fonctionnalité Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/) est une fonctionnalité de virtualisation générale bien plus puissante que les conteneurs! Avant de désactiver la fonctionnalité Hyper-V, assurez-vous qu’il n’existe pas d’autre composant virtualisé sur votre système qui en a besoin.

#### ***<a name="steps-to-remove-windows-features-on-windows-10"></a>Étapes pour supprimer les fonctionnalités Windows sur Windows10:10:***
- Accédez au **«Panneau de configuration» > «Programmes» > «Programmes et fonctionnalités» > «Activer ou désactiver des fonctionnalités Windows»** sur votre machine Windows10
- Recherchez le nom de la ou des fonctionnalités que vous souhaitez désactiver, en l’occurrence **«Conteneurs»** et (éventuellement) **«Hyper-V»**
- **Décochez** la case en regard du nom de la fonctionnalité que vous souhaitez désactiver
- Cliquez sur **«OK»**.

#### ***<a name="steps-to-remove-windows-features-on-windows-server-2016"></a>Étapes pour supprimer les fonctionnalités Windows sur Windows Server2016:16:***
À partir d’une session PowerShell avec élévation de privilèges, utilisez les commandes suivantes pour désactiver les fonctionnalités **«Conteneurs»** et (éventuellement) **«Hyper-V»** de votre système:
```
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V 
```

### <a name="step-4-reboot-your-system"></a>ÉTAPE4: Redémarrez votre système
Pour effectuer ces étapes de nettoyage/de désinstallation, à partir d’une session PowerShell avec élévation de privilèges, exécutez:
```
Restart-Computer -Force
```

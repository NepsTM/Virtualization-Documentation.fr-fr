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
ms.openlocfilehash: ccc45d47fc9f17c10b149bc647463824e1ecbc9e
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2017
---
# <a name="docker-engine-on-windows"></a>Moteur Docker sur Windows

Le moteur et le client Docker ne sont pas inclus avec Windows et doivent être installés et configurés individuellement. De plus, le moteur Docker accepte de nombreuses configurations personnalisées. Certains exemples incluent la configuration de la façon dont le démon accepte les requêtes entrantes, les options de mise en réseau par défaut et les paramètres de débogage/du journal. Sur Windows, ces configurations peuvent être spécifiées dans un fichier de configuration ou à l’aide du Gestionnaire de contrôle des services Windows. Ce document décrit en détail comment installer et configurer le moteur Docker, et fournit également quelques exemples de configurations fréquemment utilisées.


## <a name="install-docker"></a>Installer Docker
Vous devez installer Docker pour utiliser les conteneurs Windows. Docker comprend le moteur Docker (dockerd.exe) et le client Docker (docker.exe). Vous trouverez le moyen le plus simple de tout installer dans les guides de démarrage rapide. Ces guides vous aideront à tout configurer et à exécuter votre premier conteneur. 

* [Conteneurs Windows sur Windows Server2016](../quick-start/quick-start-windows-server.md)
* [Conteneurs Windows sur Windows10](../quick-start/quick-start-windows-10.md)


### <a name="manual-installation"></a>Installation manuelle
Si vous souhaitez plutôt utiliser une version en développement du client et du moteur Docker, vous pouvez effectuer les étapes qui suivent. Elles vous permettront d’installer à la fois le client et le moteur Docker. Si, en tant que développeur, vous testez de nouvelles fonctionnalités ou utilisez une build de Windows Insider, vous devrez peut-être utiliser une version en développement de Docker. Autrement, suivez les étapes décrites dans la section «Installer Docker» ci-dessus pour obtenir les dernières versions officielles.

> Si vous avez installé Docker pour Windows, veillez à le supprimer avant de suivre ces étapes d’installation manuelles. 

Télécharger le moteur Docker

Vous trouverez la dernière version à l’adresse https://master.dockerproject.org. Cet exemple utilise la dernière version provenant de la branche maître. 

```powershell
$version = (Invoke-WebRequest -UseBasicParsing https://raw.githubusercontent.com/docker/docker/master/VERSION).Content.Trim()
Invoke-WebRequest "https://master.dockerproject.org/windows/x86_64/docker-$($version).zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Développez l’archive zip dans Program Files.

```powershell
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Ajoutez le répertoire Docker au chemin d’accès système. Quand vous avez terminé, redémarrez la session PowerShell pour que le chemin d’accès modifié soit reconnu.

```powershell
# Add path to this PowerShell session immediately
$env:path += ";$env:ProgramFiles\Docker"

# For persistent use after a reboot
$existingMachinePath = [Environment]::GetEnvironmentVariable("Path",[System.EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("Path", $existingMachinePath + ";$env:ProgramFiles\Docker", [EnvironmentVariableTarget]::Machine)
```

Pour installer Docker en tant que service Windows, exécutez la commande suivante.

```
dockerd --register-service
```

Une fois installé, le service peut être démarré.

```powershell
Start-Service Docker
```

Pour que Docker puisse être utilisé, des images de conteneur doivent au préalable être installées. Pour plus d’informations, voir le [guide de démarrage rapide consacré à l’utilisation d’images](../quick-start/quick-start-images.md).

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
    "graph": "",
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
    "graph": "d:\\docker"
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

Pour configurer le moteur Docker de sorte qu’un réseau NAT par défaut ne soit pas créé, utilisez le code suivant. Pour plus d’informations, voir la rubrique indiquant comment [gérer les réseaux Docker](../manage-containers/container-networking.md).

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


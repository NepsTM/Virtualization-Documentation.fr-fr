---
title: Configurer Docker dans Windows
description: Configurer Docker dans Windows
keywords: docker, conteneurs
author: neilpeterson
manager: timlt
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
translationtype: Human Translation
ms.sourcegitcommit: 38d9f06af87cf1d69529d28e30cab60f16e0982b
ms.openlocfilehash: 185831094b63a1b7fb1931db7fb82a6c59c2b060

---

# Moteur Docker sur Windows

Le moteur et le client Docker ne sont pas inclus avec Windows et doivent être installés et configurés individuellement. De plus, le moteur Docker accepte de nombreuses configurations personnalisées. Certains exemples incluent la configuration de la façon dont le démon accepte les requêtes entrantes, les options de mise en réseau par défaut et les paramètres de débogage/du journal. Sur Windows, ces configurations peuvent être spécifiées dans un fichier de configuration ou à l’aide du Gestionnaire de contrôle des services Windows. Ce document décrit en détail comment installer et configurer le moteur Docker, et fournit également quelques exemples de configurations fréquemment utilisées.


## Installer Docker
Vous devez installer Docker pour utiliser les conteneurs Windows. Docker comprend le moteur Docker (dockerd.exe) et le client Docker (docker.exe). Vous trouverez le moyen le plus simple de tout installer dans les guides de démarrage rapide. Ces guides vous aideront à tout configurer et à exécuter votre premier conteneur. 

* [Conteneurs Windows sur Windows Server 2016](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/quick_start/quick_start_windows_server)
* [Conteneurs Windows sur Windows 10](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/quick_start/quick_start_windows_10)


### Installation manuelle
Si vous souhaitez plutôt utiliser une version en développement du client et du moteur Docker, vous pouvez effectuer les étapes qui suivent. Elles vous permettront d’installer à la fois le client et le moteur Docker. Dans le cas contraire, passez à la section suivante.

> Si vous avez installé Docker pour Windows, veillez à le supprimer avant de suivre ces étapes d’installation manuelles. 

Télécharger le moteur Docker

Vous trouverez la dernière version à l’adresse https://master.dockerproject.org. Cet exemple utilise la toute dernière version de la branche de développement v1.13. 

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Développez l’archive zip dans Program Files.

```
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Ajoutez le répertoire Docker au chemin du système. Quand vous avez terminé, redémarrez la session PowerShell pour que le chemin d’accès modifié soit reconnu.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Pour installer Docker en tant que service Windows, exécutez la commande suivante.

```none
dockerd --register-service
```

Une fois installé, le service peut être démarré.

```none
Start-Service Docker
```

Pour que Docker puissent être utilisé, des images de conteneur doivent au préalable être installées. Pour plus d’informations, voir [Gérer des images de conteneur](../management/manage_images.md).

## Configurer Docker avec un fichier de configuration

La méthode privilégiée pour configurer le moteur Docker sur Windows consiste à utiliser un fichier de configuration. Ce fichier de configuration se trouve dans C:\ProgramData\docker\config\daemon.json. Si ce fichier n’existe pas encore, il peut être créé.

Remarque : Toutes les options de configuration de Docker disponibles ne s’appliquent pas à Docker sur Windows. L’exemple ci-dessous montre celles qui le sont. Pour obtenir une documentation complète sur la configuration du moteur Docker, voir la rubrique relative au [fichier de configuration du démon Docker](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

```none
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

Seules les modifications de configuration souhaitées doivent être ajoutées au fichier de configuration. Ainsi, le présent exemple configure le moteur Docker pour accepter les connexions entrantes sur le port 2375. Toutes les autres options de configuration utiliseront les valeurs par défaut.

```none
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

De même, cet exemple configure le démon Docker pour accepter uniquement les connexions sécurisées sur le port 2376.

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## Configurer Docker sur le service Docker

Le moteur Docker peut également être configuré en modifiant le service de Docker avec `sc config`. Avec cette méthode, les indicateurs du moteur Docker sont définis directement sur le service de Docker. Exécutez la commande suivante dans une invite de commandes (cmd.exe, pas Powershell) :


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

Remarque : Vous n’avez pas besoin d’exécuter cette commande si votre fichier daemon.json contient déjà l’entrée `"hosts": ["tcp://0.0.0.0:2375"]`.

## Configuration commune

Les exemples de fichiers de configuration suivants présentent des configurations courantes de Docker. Elles peuvent être combinées en un seul fichier de configuration.

### Création de réseau par défaut 

Pour configurer le moteur Docker de sorte qu’un réseau NAT par défaut ne soit pas créé, utilisez le code suivant. Pour plus d’informations, voir la rubrique indiquant comment [gérer les réseaux Docker](../management/container_networking.md).

```none
{
    "bridge" : "none"
}
```

### Définir un groupe de sécurité Docker

Quand vous êtes connecté à l’hôte Docker et que vous exécutez des commandes Docker localement, ces commandes sont exécutées via un canal nommé. Par défaut, seuls les membres du groupe Administrateurs peuvent accéder au moteur Docker via le canal nommé. Pour spécifier un groupe de sécurité bénéficiant de cet accès, utilisez l’indicateur `group`.

```none
{
    "group" : "docker"
}
```

## Configuration du proxy

Pour définir des informations de proxy pour `docker search` et `docker pull`, créez une variable d’environnement Windows nommée `HTTP_PROXY` ou `HTTPS_PROXY`, et une valeur des informations de proxy. Vous pouvez effectuer cette opération dans PowerShell en utilisant une commande semblable à celle-ci :

```none
[Environment]::SetEnvironmentVariable("HTTP_PROXY”, “http://username:password@proxy:port/”, [EnvironmentVariableTarget]::Machine)
```

Une fois que la variable a été définie, redémarrez le service Docker.

```none
restart-service docker
```

Pour plus d’informations, voir [Options de socket du démon sur Docker.com](https://docs.docker.com/v1.10/engine/reference/commandline/daemon/#daemon-socket-option).



<!--HONumber=Oct16_HO3-->



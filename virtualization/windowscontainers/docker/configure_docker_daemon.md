---
title: Configurer Docker dans Windows
description: Configurer Docker dans Windows
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
translationtype: Human Translation
ms.sourcegitcommit: 2d6f2c24624883457302c925c2bb47e6c867b730
ms.openlocfilehash: 533f3a3277e3d9654f0d425c9c0f442c93e2d24a

---

# Démon Docker sur Windows

Le moteur Docker n’est pas inclus avec Windows et doit être installé et configuré individuellement. De plus, le démon Docker peut accepter de nombreuses configurations personnalisées. Certains exemples incluent la configuration de la façon dont le démon accepte les requêtes entrantes, les options de mise en réseau par défaut et les paramètres de débogage/du journal. Sur Windows, ces configurations peuvent être spécifiées dans un fichier de configuration ou à l’aide du Gestionnaire de contrôle des services Windows. Ce document décrit en détail comment installer et configurer le démon Docker et fournit également quelques exemples de configurations fréquemment utilisées.

## Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. Pour cet exercice, les deux sont installés.

Créez un dossier pour les exécutables Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Téléchargez le démon Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Téléchargez le client Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

Ajoutez le répertoire Docker au chemin d’accès système. Quand vous avez terminé, redémarrez la session PowerShell pour que le chemin d’accès modifié soit reconnu.

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

## Fichier de configuration de Docker

La méthode privilégiée pour configurer le démon Docker sur Windows consiste à utiliser un fichier de configuration. Ce fichier de configuration se trouve dans C:\ProgramData\docker\config\daemon.json. Si ce fichier n’existe pas encore, il peut être créé.

Remarque : Toutes les options de configuration de Docker disponibles ne s’appliquent pas à Docker sur Windows. L’exemple ci-dessous montre celles qui le sont. Pour obtenir une documentation complète sur la configuration du démon Docker, notamment pour Linux, voir la rubrique relative au [Démon Docker]( https://docs.docker.com/v1.10/engine/reference/commandline/daemon/).

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

Seules les modifications de configuration souhaitées doivent être ajoutées au fichier de configuration. Ainsi, le présent exemple configure le démon Docker pour accepter les connexions entrantes sur le port 2375. Toutes les autres options de configuration utiliseront les valeurs par défaut.

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



## Gestionnaire de contrôle des services

Le démon Docker peut également être configuré en modifiant le service de Docker avec `sc config`. À l’aide de cette méthode, les indicateurs du démon Docker sont définis directement sur le service de Docker.


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

## Configuration commune

Les exemples de fichiers de configuration suivants présentent des configurations courantes de Docker. Elles peuvent être combinées en un seul fichier de configuration.

### Création de réseau par défaut 

Pour configurer le démon Docker de sorte qu’un réseau NAT par défaut ne soit pas créé, utilisez le code suivant. Pour plus d’informations, voir la rubrique indiquant comment [gérer les réseaux Docker](../management/container_networking.md).

```none
{
    "bridge" : "none"
}
```

### Définir un groupe de sécurité Docker

Quand vous êtes connecté à l’hôte Docker et que vous exécutez des commandes Docker localement, ces commandes sont exécutées via un canal nommé. Par défaut, seuls les membres du groupe Administrateurs peuvent accéder au démon Docker via le canal nommé. Pour spécifier un groupe de sécurité bénéficiant de cet accès, utilisez l’indicateur `group`.

```none
{
    "group" : "docker"
}
```


## Collecte de journaux
Les journaux du démon Docker sont consignés dans le journal des événements des applications Windows, plutôt que dans un fichier. Ces journaux peuvent facilement être lus, triés et filtrés à l’aide de Windows PowerShell

Par exemple, cette commande affiche les journaux du démon Docker des 5 dernières minutes en commençant par le plus ancien.
```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

Ces journaux peuvent aussi être facilement redirigés dans un fichier CSV pour être lus par un autre outil ou une feuille de calcul.
```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.csv ```



<!--HONumber=Jul16_HO1-->



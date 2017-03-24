---
title: Prise en main du mode Swarm
description: "Initialisation d’un cluster Swarm, création d’un réseau de superposition et association d’un service au réseau."
keywords: docker, conteneurs, swarm, orchestration
author: kallie-b
ms.date: 02/9/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5ceb9626-7c48-4d42-81f8-9c936595ad85
translationtype: Human Translation
ms.sourcegitcommit: f615c6dd268932a2ff99ac12c4e9ffdcf2cc217e
ms.openlocfilehash: ee6053003b31f226d2cfba8566f274ccc19d97ec
ms.lasthandoff: 03/02/2017

---

# Prise en main du mode Swarm 

**Remarque importante :** *pour le moment, la prise en charge du mode Swarm et de la mise en réseau de superposition n’est disponible que les [Windows Insiders](https://insider.windows.com/) dans le cadre de la prochaine Windows 10 Creators Update. Prise en charge pour les autres plateformes Windows bientôt disponible.*

## Qu’est-ce que le « mode Swarm » ?
Le mode Swarm est une fonctionnalité de Docker qui fournit des fonctionnalités prédéfinies d’orchestration de conteneur, y compris le clustering natif des hôtes Docker et la planification des charges de travail de conteneur. Un groupe d’hôtes Docker constitue un cluster « Swarm » lorsque leurs moteurs Docker s’exécutent ensemble en « mode Swarm ». Pour un contexte supplémentaire sur le mode Swarm, reportez-vous au [site de documentation principale de Docker](https://docs.docker.com/engine/swarm/).

## Nœuds de gestionnaire et nœuds de travail
Un cluster Swarm est composé de deux types d’hôte de conteneur : *nœuds de gestionnaire* et *nœuds de travail*. Chaque Swarm est initialisé via un nœud de gestionnaire, et toutes les commandes de l’interface de ligne de commande Docker pour contrôler et analyser un cluster Swarm doivent être exécutées à partir de l’un de ses nœuds de gestionnaire. Les nœuds de gestionnaire peuvent être comparés aux « gardiens » de l’état Swarm : ensemble, ils forment un groupe consensuel qui gère la reconnaissance de l’état des services exécutés dans le cluster Swarm. Leur tâche consiste à s’assurer que l’état réel du cluster Swarm correspond à son état souhaité, défini par le développeur ou l’administrateur. 

>    **Remarque :** tout cluster Swarm peut disposer de plusieurs nœuds de gestionnaire, mais il doit toujours en posséder *au moins un*. 

Les nœuds de travail sont orchestrés par Docker Swarm via les nœuds de gestionnaire. Pour joindre un cluster Swarm, un nœud de travail doit utiliser un « jeton de jointure », qui aura été généré par le nœud de gestionnaire à l’initialisation du cluster Swarm. Les nœuds de travail reçoivent et exécutent simplement les tâches des nœuds de gestionnaire. En conséquence, ils ne nécessitent (et ne détiennent) pas de reconnaissance de l’état Swarm.

## Configuration requise du mode Swarm

Au moins un ordinateur physique ou virtuel (pour utiliser les fonctionnalités complètes de Swarm, au moins deux nœuds sont recommandés) exécutant **Windows 10 Creators Update** (disponible pour les membres du programme [Windows Insiders](https://insider.windows.com/)), configuré en tant qu’hôte de conteneur (voir la rubrique [Conteneurs Windows sur Windows 10](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10) pour en savoir plus sur la prise en main des conteneurs Docker sur Windows 10)

**Moteur Docker v1.13.0 ou ultérieur**

Ports ouverts : les ports suivants doivent être disponibles sur chaque hôte. Sur certains systèmes, ces ports sont ouverts par défaut.
- Port TCP 2377 pour les communications de gestion de cluster
- Port TCP et UDP 7946 pour la communication entre les nœuds
- Port TCP et UDP 4789 pour le trafic réseau de superposition

## Initialisation d’un cluster Swarm
Pour initialiser un cluster Swarm, exécutez simplement la commande suivante à partir d’un de vos hôtes de conteneur (en remplaçant \<HOSTIPADDRESS\> par l’adresse IPv4 locale de votre ordinateur hôte) :

```none
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```
Lorsque cette commande est exécutée à partir d’un hôte de conteneur donné, le moteur Docker de cet hôte commence à s’exécuter en mode Swarm en tant que nœud de gestionnaire.

## Ajout de nœuds à un cluster Swarm

> **Remarque :** plusieurs nœuds ne sont *pas* nécessaires pour tirer parti des fonctionnalités du mode Swarm et du mode Mise en réseau de superposition. Toutes les fonctionnalités Swarm/superposition peuvent être utilisées avec un seul et même hôte s’exécutant en mode Swarm (autrement dit, un nœud de gestionnaire placé en mode Swarm avec la commande `docker swarm init`).

### Ajout de nœuds de travail à un cluster Swarm
Une fois qu’un cluster Swarm a été initialisé à partir d’un nœud de gestionnaire, d’autres hôtes peuvent être ajoutés à ce cluster en tant que nœuds de travail à l’aide d’une simple autre commande :

```none
C:\> docker swarm join --token <WORKERJOINTOKEN> <MANAGERIPADDRESS>
```

Ici, \<MANAGERIPADDRESS\> est l’adresse IP locale d’un nœud de gestionnaire Swarm, et \<WORKERJOINTOKEN\> le jeton de jointure du nœud de travail fourni en tant que sortie par la commande `docker swarm init` exécutée à partir du nœud de gestionnaire. Le jeton de jointure peut également être obtenu en exécutant l’une des commandes suivantes à partir du nœud de gestionnaire après l’initialisation du cluster Swarm :

```none
# Get the full command required to join a worker node to the swarm
C:\> docker swarm join-token worker

# Get only the join-token needed to join a worker node to the swarm
C:\> docker swarm join-token worker -q
```

### Ajout de nœuds de gestionnaire à un cluster Swarm
D’autres nœuds de gestionnaire peuvent être ajoutés à un cluster Swarm à l’aide de la commande suivante :

```none
C:\> docker swarm join --token <MANAGERJOINTOKEN> <MANAGERIPADDRESS>
```

Là encore, \<MANAGERIPADDRESS\> est l’adresse IP locale d’un nœud de gestionnaire de cluster Swarm. Le jeton de jointure, \<MANAGERJOINTOKEN\>, est un jeton de jointure de *gestionnaire*, qui peut être obtenu en exécutant l’une des commandes suivantes à partir d’un nœud de gestionnaire existant :

```none
# Get the full command required to join a **manager** node to the swarm
C:\> docker swarm join-token manager

# Get only the join-token needed to join a **manager** node to the swarm
C:\> docker swarm join-token manager -q
```

## Création d’un réseau de superposition

Une fois qu’un cluster Swarm a été configuré, des réseaux de superposition peuvent être créés dans le cluster. Vous pouvez créer un réseau de superposition en exécutant la commande suivante à partir d’un nœud de gestionnaire du cluster Swarm :

```none
# Create an overlay network 
C:\> docker network create --driver=overlay <NETWORKNAME>
```

Ici, \<NETWORKNAME\> est le nom que vous souhaitez donner à votre réseau.

## Déploiement de services dans un cluster Swarm
Une fois qu’un réseau de superposition a été créé, des services peuvent être créés et associés au réseau. Un réseau est créé à l’aide de la syntaxe suivante :

```none
# Deploy a service to the swarm
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

Ici, \<SERVICENAME\> est le nom que vous souhaitez donner au service. Il s’agit du nom que vous utiliserez pour référencer le service via la découverte de service (qui utilise un serveur DNS natif de Docker). \<NETWORKNAME\> est le nom du réseau auquel vous souhaitez connecter ce service (par exemple, « monRéseauSuperpos »). \<CONTAINERIMAGE\> est le nom de l’image de conteneur qui est défini par le service.

> **Remarque :** le deuxième argument de cette commande, `--endpoint-mode dnsrr`, est nécessaire pour spécifier au moteur Docker que la stratégie de tourniquet DNS sera utilisée pour équilibrer le trafic réseau entre les points de terminaison du conteneur de service. Actuellement, le tourniquet DNS est la seule stratégie d’équilibrage de charge prise en charge sur Windows. Le [routage de maillage](https://docs.docker.com/engine/swarm/ingress/) pour les hôtes Windows Docker n’est pas encore pris en charge, mais il le sera prochainement. Les utilisateurs recherchant une autre stratégie d’équilibrage de charge dès aujourd'hui peuvent configurer un équilibreur de charge externe (par exemple, NGINX) et utiliser le [mode de publication de port](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) de Swarm pour exposer les ports d’hôte de conteneur sur lesquels équilibrer la charge.

## Mise à l’échelle d’un service
Une fois qu’un service est déployé dans un cluster Swarm, les instances de conteneur composant ce service sont déployées dans le cluster. Par défaut, le nombre d’instances de conteneur soutenant un service, autrement dit le nombre de « réplicas » ou de « tâches » d’un service, est de une. Toutefois, un service peut être créé avec plusieurs tâches à l’aide de l’option `--replicas` pour la commande `docker service create` ou en mettant à l’échelle le service après sa création.

L’évolutivité de service est l’un des principaux avantages de Docker Swarm. Elle peut également être exploitée avec une seule commande Docker :

```none
C:\> docker service scale <SERVICENAME>=<REPLICAS>
```

Ici, \<SERVICENAME\> est le nom du service mis à l’échelle, et \<REPLICAS\> est le nombre de tâches ou d’instances de conteneur, vers lesquelles le service est mis à l’échelle.

## Affichage de l’état Swarm

Plusieurs commandes sont utiles pour afficher l’état d’un cluster Swarm et les services qui y sont exécutés.

### Répertorier les nœuds Swarm
Utilisez la commande suivante pour afficher la liste des nœuds actuellement joints à un cluster Swarm, notamment des informations sur l’état de chaque nœud. Cette commande doit être exécutée à partir d’un **nœud de gestionnaire**.

```none
C:\ docker node ls
```

Dans la sortie de cette commande, vous pouvez remarquer l’un des nœuds marqués d’un astérisque (*) ; l’astérisque indique simplement le nœud actuel, c.-à-d. le nœud à partir duquel la commande `docker node ls` a été exécutée.

### Dresser la liste des réseaux
Utilisez la commande suivante pour afficher la liste des réseaux qui existent sur un nœud donné. Pour afficher les réseaux de superposition, vous devez exécuter cette commande à partir d’un **nœud de gestionnaire** exécuté en mode Swarm.

```none
C:\ docker network ls
```

### Liste des services
Utilisez la commande suivante pour afficher la liste des services en cours d’exécution sur un cluster Swarm, y compris des informations sur leur état.

```none
C:\ docker service ls
```

### Répertorier les instances de conteneur qui définissent un service
Utilisez la commande suivante pour afficher des détails sur les instances de conteneur exécutées pour un service donné. La sortie de cette commande inclut les ID et les nœuds sur lesquels chaque conteneur s’exécute, ainsi que des informations sur l’état des conteneurs.  

```none
C:\ docker service ps <SERVICENAME>
```

## Limitations
Actuellement, le mode Swarm sur Windows connaît les limitations suivantes :
- BOGUE CONNU : le mode Superposition et le mode Swarm sont actuellement pris en charge uniquement sur les hôtes de conteneur connectés Ethernet ; **ils ne fonctionnent pas avec des hôtes connectés Wi-Fi.** Nous travaillons actuellement à la résolution de ce problème.
- Chiffrement de plan-données non pris en charge (autrement dit, le trafic conteneur à conteneur à l’aide de l’option `--opt encrypted`)
- Le [routage de maillage](https://docs.docker.com/engine/swarm/ingress/) pour les hôtes Windows Docker n’est pas encore pris en charge, mais il le sera prochainement. Les utilisateurs recherchant une autre stratégie d’équilibrage de charge dès aujourd'hui peuvent configurer un équilibreur de charge externe (par exemple, NGINX) et utiliser le [mode de publication de port](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) de Swarm pour exposer les ports d’hôte de conteneur sur lesquels équilibrer la charge.  




---
title: "Mise en réseau de conteneurs Windows"
description: "Configurez la mise en réseau pour les conteneurs Windows."
keywords: docker, conteneurs
author: jmesser81
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
translationtype: Human Translation
ms.sourcegitcommit: f489d3e6f98fd77739a2016813506be6962b34d1
ms.openlocfilehash: 499788666f306494c894b2e82f65ab68c9fc295a

---

# Mise en réseau de conteneur

Sur le plan de la mise en réseau, les conteneurs Windows fonctionnent de la même façon que des machines virtuelles. Chaque conteneur a une carte réseau virtuelle qui est connectée à un commutateur virtuel sur lequel le trafic entrant et sortant est transféré. Pour appliquer une isolation entre les conteneurs sur le même hôte, un compartiment réseau est créé pour chaque conteneur Windows Server et Hyper-V dans lequel est installée la carte réseau pour le conteneur. Les conteneurs Windows Server utilisent une carte réseau virtuelle hôte pour s’attacher au commutateur virtuel. Les conteneurs Hyper-V utilisent une carte réseau de machine virtuelle synthétique (non exposée à la machine virtuelle d’utilitaire) pour s’attacher au commutateur virtuel.

Les conteneurs Windows prennent en charge les quatre modes ou pilotes réseau suivants : *nat*, *transparent*, *l2bridge* et *l2tunnel*. Choisissez le mode réseau le mieux adapté à l’infrastructure de votre réseau physique et à la configuration requise du réseau (hôte unique ou hôtes multiples). 

Le moteur Docker crée un réseau NAT par défaut lors de la première exécution du service Docker. Le préfixe IP interne par défaut créé est 172.16.0.0/12. Les points de terminaison du conteneur sont automatiquement attachés à ce réseau par défaut, et une adresse IP leur est affectée à partir de son préfixe interne.

> Remarque : Si l’adresse IP de votre hôte de conteneur est dans ce même préfixe, vous devez modifier le préfixe IP interne du NAT comme indiqué ci-dessous.

Il est possible de créer des réseaux supplémentaires sur le même hôte de conteneur en utilisant un pilote différent (par exemple, transparent, l2bridge). Le tableau ci-dessous montre de quelle manière est fournie la connectivité réseau pour les connexions internes (conteneur-conteneur) et externes dans chaque mode.

- **NAT** : Chaque conteneur reçoit une adresse IP à partir d’un préfixe IP privé interne (par exemple, 172.16.0.0/12). Le réacheminement/mappage de ports à partir de l’hôte de conteneur vers des points de terminaison de conteneur est pris en charge.

- **Transparent** : Chaque point de terminaison de conteneur est directement connecté au réseau physique. Les adresses IP issues du réseau physique peuvent être attribuées de façon statique ou dynamique à l’aide d’un serveur DHCP externe.

- **L2 Bridge** : Chaque point de terminaison de conteneur se trouve dans le même sous-réseau IP que l’hôte de conteneur. Les adresses IP doivent être attribuées de façon statique à partir du même préfixe que l’hôte de conteneur. Tous les points de terminaison de conteneur sur l’hôte ont la même adresse MAC en raison de la traduction d’adresses Layer-2.

- **L2 Tunnel** - _ : Ce mode doit uniquement être utilisé dans une pile Microsoft Cloud Stack_.

> Pour savoir comment connecter des points de terminaison de conteneur à un réseau virtuel de superposition avec la pile Microsoft SDN, consultez la rubrique relative à l’[attachement de conteneurs à un réseau virtuel](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network).

## Nœud unique

|  | Conteneur-conteneur | Conteneur-externe |
| :---: | :---------------     |  :---                |
| nat | Connexion reliée par un pont via un commutateur virtuel Hyper-V | routage via WinNAT avec application des traductions d’adresses | 
| transparent | Connexion reliée par un pont via un commutateur virtuel Hyper-V | accès direct au réseau physique | 
| l2bridge | Connexion reliée par un pont via un commutateur virtuel Hyper-V|  accès au réseau physique avec la traduction d’adresses MAC|  



## Nœuds multiples

|  | Conteneur-conteneur | Conteneur-externe |
| :---: | :----       | :---------- |
| nat | doit référencer le port et l’IP de l’hôte de conteneur externe ; routage via WinNAT avec application des traductions d’adresses | doit référencer l’hôte externe ; routage via WinNAT avec application des traductions d’adresses | 
| transparent | doit directement référencer le point de terminaison IP du conteneur | accès direct au réseau physique | 
| l2bridge | doit directement référencer le point de terminaison IP du conteneur| accès au réseau physique avec la traduction d’adresses MAC| 


## Création de réseau 

### Réseau NAT (par défaut)

Le moteur Docker Windows crée un réseau NAT par défaut (que Docker nomme « nat ») avec le préfixe IP 172.16.0.0/12. Si un utilisateur veut créer un réseau NAT avec un préfixe IP spécifique, il peut le faire de deux façons en modifiant les options définies dans le fichier daemon.json de configuration de Docker (si ce fichier n’existe pas dans C:\ProgramData\Docker\config\daemon.json, créez-le).
 1. Utilisez l’option _« fixed-cidr » : « < Préfixe IP > / Masque »_ pour créer le réseau NAT par défaut avec le préfixe IP et la correspondance spécifiés
 2. Utilisez l’option _"bridge": "none"_ pour ne pas créer le réseau par défaut et permettre à un utilisateur de créer un réseau avec le pilote de son choix à l’aide de la commande *docker network create -d <driver>*

Avant d’appliquer une de ces options de configuration, le service Docker doit d’abord être arrêté et les réseaux NAT existants doivent être supprimés.

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

Si l’option « fixed-cidr » a été ajoutée au fichier daemon.json, le moteur Docker crée un réseau NAT défini par l’utilisateur avec le préfixe IP et le masque personnalisés spécifiés. Si au lieu de cela l’option « bridge:none » est ajoutée, le réseau doit être créé manuellement.

```none
# Create a user-defined NAT network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

Par défaut, les points de terminaison du conteneur sont connectés au réseau « nat » par défaut. Si le réseau « nat » n’a pas été créé (du fait que l’option « bridge:none » a été spécifiée dans le fichier daemon.json) ou si l’accès à un autre réseau défini par l’utilisateur est obligatoire, les utilisateurs peuvent spécifier le paramètre *--network* dans la commande docker run.

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### Mappage de ports

Pour accéder à des applications s’exécutant à l’intérieur d’un conteneur connecté à un réseau NAT, des mappages de ports doivent être créés entre l’hôte de conteneur et le point de terminaison de conteneur. Ces mappages doivent être spécifiés au moment de la création du conteneur ou quand le conteneur est dans un état ARRÊTÉ.

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Creates a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

Les mappages de ports dynamiques sont également autorisés en utilisant le paramètre -p ou la commande EXPOSE dans un fichier Dockerfile avec le paramètre -P. S’il n’est pas spécifié, un port éphémère est choisi au hasard sur l’hôte de conteneur et peut être inspecté lors de l’exécution de « docker ps ».

```none
C:\> docker run -itd -p 80 windowsservercore cmd

# Network services running on port TCP:80 in this container can be accessed externally on port TCP:14824
C:\> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
bbf72109b1fc        windowsservercore   "cmd"               6 seconds ago       Up 2 seconds        *0.0.0.0:14824->80/tcp*   drunk_stonebraker

# Container image specified EXPOSE 80 in Dockerfile - publish this port mapping
C:\> docker network 
```
> Dans WS2016 TP5 et les builds Windows Insider supérieures à 14300, une règle de pare-feu est automatiquement créée pour tous les mappages de ports NAT. Cette règle de pare-feu affecte globalement l’hôte de conteneur et n’est pas limitée à une carte réseau ni un point de terminaison de conteneur spécifique.

L’implémentation Windows NAT (WinNAT) présente quelques lacunes de fonctionnalité décrites dans le billet de blog [WinNAT capabilities and limitations](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations/) (Fonctionnalités et limitations de WinNAT) 
 1. Un seul préfixe IP interne NAT est pris en charge par hôte de conteneur : les réseaux NAT « multiples » doivent donc être définis en partitionnant le préfixe (consultez la section « Réseaux NAT multiples » de ce document).
 2. Les points de terminaison du conteneur sont accessibles seulement à partir de l’hôte de conteneur en utilisant les adresses IP et les ports internes du conteneur (recherchez ces informations en utilisant « docker network inspect <CONTAINER ID> »).

Des réseaux supplémentaires peuvent être créés avec des pilotes différents. 

> Les noms des pilotes réseau Docker s’écrivent en minuscules.

### Réseau transparent

Pour utiliser le mode de mise en réseau transparent, créez un réseau de conteneurs avec le nom de pilote « transparent ». 

```none
C:\> docker network create -d transparent MyTransparentNetwork
```
> Remarque : Si vous rencontrez une erreur lors de la création du réseau transparent, il est possible qu’il existe un commutateur virtuel externe sur votre système qui n’a pas été découvert automatiquement par Docker, et qui par conséquent empêche la liaison du réseau transparent à la carte réseau externe de votre hôte de conteneur. Pour plus d’informations, consultez la section ci-dessous, « Commutateur virtuel existant bloquant la création du réseau transparent » sous « Mises en garde et pièges ».

Si l’hôte de conteneur est virtualisé et que vous souhaitez utiliser DHCP pour l’attribution d’adresses IP, vous devez activer MACAddressSpoofing sur la carte réseau de machines virtuelles. Sinon, l’hôte Hyper-V bloque le trafic réseau provenant des conteneurs sur la machine virtuelle utilisant plusieurs adresses MAC.

```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> Pour créer plusieurs réseaux transparents, vous devez spécifier la carte réseau (virtuelle) à laquelle le commutateur virtuel Hyper-V externe (créé automatiquement) doit être lié.

Pour lier un réseau (attaché via le commutateur virtuel Hyper-V) à une interface réseau spécifique, utilisez l’option *-o com.docker.network.windowsshim.interface=<Interface>*

```none
# Create a transparent network which is attached to the "Ethernet 2" network interface
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

La valeur pour *com.docker.network.windowsshim.interface* est le *Nom* de la carte provenant de : 
```none
Get-NetAdapter
```

Les adresses IP pour les points de terminaison de conteneur connectés à un réseau transparent peuvent être attribuées de façon statique ou dynamique à partir d’un serveur DHCP externe.

Quand vous utilisez l’attribution IP statique, assurez-vous d’abord que les paramètres *--subnet* et *--gateway* sont spécifiés au moment de la création du réseau. L’adresse IP de sous-réseau et de passerelle doit être la même que les paramètres réseau de l’hôte de conteneur (le réseau physique).

```none
# Create a transparent network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 TransparentNet3
```
Spécifiez une adresse IP en utilisant l’option *--ip* dans la commande docker run

```none
C:\> docker run -it --network=TransparentNet3 --ip 10.123.174.105 <image> <cmd>
```

> Assurez-vous que cette adresse IP n’est pas déjà attribuée à un autre périphérique réseau sur le réseau physique

Vous n’avez pas besoin de spécifier les mappages de ports, car les points de terminaison de conteneur ont directement accès au réseau physique

### L2 Bridge 

Pour utiliser le mode de mise en réseau pont de couche 2, créez un réseau de conteneurs avec le nom de pilote « l2bridge ». Vous devez également spécifier un sous-réseau et une passerelle correspondant au réseau physique.

```none
C:\> docker network create -d l2bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyBridgeNetwork
```

Seule l’attribution IP statique est prise en charge pour les réseaux l2bridge. 

> Si vous utilisez un réseau l2bridge sur une structure SDN, seule l’attribution IP dynamique est prise en charge. Pour plus d’informations, consultez la rubrique relative à l’[attachement de conteneurs à un réseau virtuel](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network).

## Autres opérations et configurations

### Répertorier les réseaux disponibles

```none
# list container networks
C:\> docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
0a297065f06a        nat                 nat                 local
d42516aa0250        none                null                local
```

### Supprimer un réseau

Utilisez `docker network rm` pour supprimer un réseau de conteneurs.

```none
C:\> docker network rm "<network name>"
```

Cette commande supprime tous les commutateurs virtuels Hyper-V utilisés par le réseau de conteneurs ainsi que toutes les traductions d’adresses réseau (WinNAT - instances NetNat) créées.

### Inspection du réseau 

Pour afficher les conteneurs qui sont connectés à un réseau spécifique et les adresses IP associées à ces points de terminaison de conteneur, vous pouvez exécuter la commande suivante.

```none
C:\> docker network inspect <network name>
```

### Plusieurs réseaux de conteneurs
 Un seul réseau NAT est actuellement pris en charge dans Windows (une [requête de tirage](https://github.com/docker/docker/pull/25097) en attente peut toutefois vous aider à contourner cette limitation). 

Plusieurs réseaux de conteneurs peuvent être créés sur un hôte de conteneur unique avec les mises en garde suivantes :

* Plusieurs réseaux qui utilisent un vSwitch externe pour la connectivité (par exemple, Transparent, L2 Bridge, L2 Transparent) doivent chacun utiliser leur propre carte réseau.
* Actuellement, notre solution pour créer plusieurs réseaux NAT sur un même hôte de conteneur consiste à partitionner le préfixe interne du réseau NAT existant. Pour obtenir de l’aide sur cette opération, consultez la section « Réseaux NAT multiples » ci-dessous.

### Réseaux NAT multiples
Il est possible de définir plusieurs réseaux NAT sur un même hôte de conteneur en partitionnant le préfixe interne du réseau NAT de l’hôte. 

Les partitions des nouveaux réseaux NAT doivent être créées sous le préfixe du réseau NAT le plus grand. Vous pouvez trouver le préfixe en exécutant la commande suivante à partir de PowerShell et en référençant le champ « InternalIPInterfaceAddressPrefix ».

```none
PS C:\> get-netnat
```

Par exemple, le préfixe interne du réseau NAT de l’hôte peut être 172.16.0.0/12. Dans ce cas, Docker peut être utilisé pour créer d’autres réseaux NAT *dès lors qu’ils sont sous le préfixe 172.16.0.0/12*. Par exemple, deux réseaux NAT peuvent être créés avec les préfixes IP 172.16.0.0/16 (passerelle 172.16.0.1) et 172.17.0.0/16 (passerelle 172.17.0.1). 

```none
C:\> docker network create -d nat --subnet=172.16.0.0/16 --gateway=172.16.0.1 CustomNat1
C:\> docker network create -d nat --subnet=172.17.0.0/16 --gateway=172.17.0.1 CustomNat2
```

Vous pouvez répertorier les réseaux nouvellement créés en utilisant :
```none
C:\> docker network ls
```


### Sélection du réseau

Quand vous créez un conteneur Windows, vous pouvez spécifier un réseau auquel la carte réseau de conteneur est connectée. Si aucun réseau n’est spécifié, le réseau NAT par défaut est utilisé.

Pour pouvoir attacher un conteneur au réseau NAT non défini par défaut, utilisez l’option --network avec la commande docker run.

```none
C:\> docker run -it --network=MyTransparentNet windowsservercore cmd
```

### Adresse IP statique

```none
C:\> docker run -it --network=MyTransparentNet --ip=10.80.123.32 windowsservercore cmd
```

L’attribution d’adresses IP statiques est effectuée directement sur la carte réseau du conteneur et ne doit être réalisée que quand le conteneur est dans un état ARRÊTÉ. Ni un « ajout à chaud » de cartes réseau de conteneurs ni un apport de modifications à la pile réseau ne sont pris en charge pendant l’exécution du conteneur.

## Docker Compose et la découverte de service

> Pour obtenir un exemple concret de l’utilisation de Docker Compose et de la découverte de service pour définir des applications multiservices avec montée en charge, consultez [ce billet](https://blogs.technet.microsoft.com/virtualization/2016/10/18/use-docker-compose-and-service-discovery-on-windows-to-scale-out-your-multi-service-container-application/) sur notre [blog de la virtualisation](https://blogs.technet.microsoft.com/virtualization/).

### Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) peut être utilisé pour définir et configurer des réseaux de conteneur en même temps que les conteneurs/services qui utiliseront ces réseaux. La clé « networks » de Compose est utilisée comme clé de plus haut niveau dans la définition des réseaux auxquels les conteneurs sont connectés. Par exemple, la syntaxe suivante définit le réseau NAT préexistant créé par Docker comme réseau « default » pour tous les conteneurs/services définis dans un fichier Compose donné.

```none
networks:
 default:
  external:
   name: "nat"
```

De même, la syntaxe suivante peut être utilisée pour définir un réseau NAT personnalisé.

> Remarque : Le « réseau NAT personnalisé » défini dans l’exemple ci-dessous est défini comme une partition du préfixe interne NAT préexistant de l’hôte du conteneur. Pour plus de contexte, consultez la section ci-dessus, « Réseaux NAT multiples ».

```none
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.17.0.0/16
```

Pour plus d’informations sur la définition et la configuration des réseaux de conteneur avec Docker Compose, consultez [Compose File reference](https://docs.docker.com/compose/compose-file/).

### Détection du service
La découverte de service, intégrée à Docker, gère l’inscription des services et le mappage des noms aux adresses IP (DNS) pour les conteneurs et les services ; avec la découverte de service, il est possible pour tous les points de terminaison de conteneur de se découvrir mutuellement par nom (le nom du conteneur ou le nom du service). Ceci est particulièrement utile dans les scénarios de montée en charge, où plusieurs points de terminaison de conteneur sont utilisés pour définir un même service. Dans ce cas, la découverte de service rend possible pour un service d’être considéré comme une seule entité, quel que soit le nombre de conteneurs en cours d’exécution en arrière-plan pour ce service. Pour les services multiconteneurs, le trafic réseau entrant est géré selon une approche de type tourniquet (round robin), dans laquelle l’équilibrage de charge DNS est utilisé pour répartir uniformément le trafic entre toutes les instances de conteneur implémentant un service donné.

## Mises en garde et pièges

### Pare-feu

L’hôte de conteneur nécessite la création de règles de pare-feu spécifiques pour activer ICMP (Ping) et DHCP. ICMP et DHCP sont nécessaires aux conteneurs Windows Server pour effectuer un test ping entre deux conteneurs sur le même hôte et pour recevoir dynamiquement des adresses IP attribuées via DHCP. Dans TP5, ces règles sont créées via le script Install-ContainerHost.ps1. Après TP5, ces règles seront créées automatiquement. Toutes les règles de pare-feu correspondant aux règles de réacheminement de port NAT sont créées automatiquement et nettoyées lors de l’arrêt du conteneur.

### Commutateur virtuel existant bloquant la création d’un réseau transparent

Quand vous créez un réseau transparent, Docker crée un commutateur virtuel externe pour le réseau, puis essaie de lier le commutateur à une carte réseau (externe) : l’adaptateur peut être une carte réseau de machine virtuelle ou la carte réseau physique. Si un commutateur virtuel a déjà été créé sur l’hôte du conteneur *et qu’il est visible par Docker*, le moteur Docker Windows utilise ce commutateur au lieu d’en créer un nouveau. Cependant, si le commutateur virtuel a été créé hors bande (c’est-à-dire sur l’hôte du conteneur à l’aide du gestionnaire Hyper-V ou de PowerShell) et qu’il n’est pas encore visible par Docker, le moteur Docker Windows tente de créer un commutateur virtuel sans parvenir par la suite à connecter le nouveau commutateur à la carte réseau externe de l’hôte de conteneur (car la carte réseau est déjà connectée au commutateur qui a été créé hors bande).

Par exemple, ce problème peut se produire si vous avez d’abord créé un commutateur virtuel sur votre hôte alors que le service Docker était en cours d’exécution, puis que vous avez essayé de créer un réseau transparent. Dans ce cas, Docker ne reconnaît pas le commutateur que vous avez créé et crée un commutateur virtuel pour le réseau transparent.

Il existe trois approches pour résoudre ce problème :

* Vous pouvez bien entendu supprimer le commutateur virtuel qui a été créé hors bande, ce qui permet à Docker de créer un commutateur virtuel et de le connecter à la carte réseau hôte sans problème. Avant de choisir cette approche, vérifiez que votre commutateur virtuel hors bande n’est pas utilisé par d’autres services (par exemple Hyper-V).
* Si vous décidez d’utiliser un commutateur virtuel externe qui a été créé hors bande, vous pouvez aussi redémarrer les services Docker et HNS pour *rendre le commutateur visible par Docker*.
```none
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* Une autre option consiste à utiliser l’option « -o com.docker.network.windowsshim.interface » pour lier le commutateur externe du réseau transparent à une carte réseau spécifique qui n’est pas déjà utilisée sur l’hôte du conteneur (par exemple une carte réseau autre que celle utilisée par le commutateur virtuel qui a été créé hors bande). L’option « -o » est décrite ci-dessus, dans la section [Réseau transparent](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/management/container_networking#transparent-network) de ce document.

### Fonctionnalités non prises en charge

Les fonctionnalités de mise en réseau suivantes ne sont pas prises en charge pour le moment via l’interface de ligne de commande Docker
 * pilote réseau de superposition par défaut
 * liaison de conteneurs (par exemple, --link)

Les options réseau suivantes ne sont pas prises en charge sur Windows Docker à ce stade :
 * --add-host
 * --dns
 * --dns-opt
 * --dns-search
 * -h, --hostname
 * --net-alias
 * --aux-address
 * --internal
 * --ip-range



<!--HONumber=Oct16_HO4-->



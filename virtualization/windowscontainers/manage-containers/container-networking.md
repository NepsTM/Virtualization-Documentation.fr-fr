---
title: "Mise en réseau de conteneurs Windows"
description: "Configurez la mise en réseau pour les conteneurs Windows."
keywords: docker, conteneurs
author: jmesser81
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 084ce01473bd3d639cc839dee3d1a49cfc3efa17
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/21/2017
---
# Mise en réseau de conteneur Windows
> ***Reportez-vous à l'article [Mise en réseau de conteneur Docker](https://docs.docker.com/engine/userguide/networking/) pour les commandes, les options et la syntaxe de mise en réseau de docker en général.*** À l’exception des cas décrits dans ce document, toutes les commandes de mise en réseau de Docker sont prises en charge par Windows avec la même syntaxe que sur Linux. Cependant, les piles réseau Windows et Linux sont différentes, par conséquent, vous constaterez que certaines commandes réseau de Linux (par exemple, ifconfig) ne sont pas prises en charge par Windows.

## Architecture de mise en réseau de base
Cette rubrique donne un aperçu de la façon dont Docker crée et gère des réseaux sous Windows. Sur le plan de la mise en réseau, les conteneurs Windows fonctionnent de la même façon que des machines virtuelles. Chaque conteneur possède une carte réseau virtuelle (vNIC) connectée à un commutateur virtuel Hyper-V (vSwitch). Windows prend en charge cinq modes ou pilotes réseau différents qui peuvent être créés au moyen de Docker: *nat*, *superposition*, *transparent*, *l2bridge* et *l2tunnel*. Choisissez le pilote réseau le mieux adapté à l’infrastructure de votre réseau physique et à la configuration requise du réseau (hôte unique ou hôtes multiples).

<figure>
  <img src="media/windowsnetworkstack-simple.png">
</figure>  

Lors de sa première exécution, le moteur Docker crée un réseau NAT par défaut, «nat», qui utilise un vSwitch interne et un composant Windows nommé `WinNAT`. Si des vSwitch externes, créés au moyen de PowerShell ou du Gestionnaire Hyper-V, sont présents sur l’ordinateur hôte, ils sont également disponibles pour Docker à l'aide du pilote réseau *transparent* et sont visibles lorsque vous exécutez la commande ``docker network ls``  

<figure>
  <img src="media/docker-network-ls.png">
</figure>

> - Un vSwitch ***interne*** est un commutateur qui n’est pas connecté directement à une carte réseau sur l’hôte du conteneur 

> - Un vSwitch ***externe*** est un commutateur qui _est_ connecté directement à une carte réseau sur l’hôte du conteneur  

<figure>
  <img src="media/get-vmswitch.png">
</figure>

Le réseau «nat» est le réseau par défaut des conteneurs qui s’exécutent sur Windows. Tous les conteneurs qui s'exécutent sur Windows sans indicateurs ou arguments pour implémenter des configurations de réseau spécifiques sont attachés au réseau «nat» par défaut, et une adresse IP leur est affectée à partir de la plage IP de préfixe interne du réseau «nat». Le préfixe IP interne par défaut utilisé pour «nat» est 172.16.0.0/16. 


## Pilotes réseau de conteneurs Windows  

En plus de tirer parti du réseau «nat» par défaut créé par Docker sur Windows, les utilisateurs peuvent définir des réseaux de conteneurs personnalisés. Il est possible de créer des réseaux définis par l’utilisateur au moyen de l’interface de ligne de commande Docker, en utilisant la commande [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/). Sur Windows, les types de pilote réseau suivants sont disponibles:

- **nat**: les conteneurs reliés à un réseau créé avec le pilote «nat» reçoivent une adresse IP à partir du préfixe IP (``--subnet``) spécifié par l’utilisateur. Le réacheminement/mappage de ports à partir de l’hôte de conteneur vers des points de terminaison de conteneur est pris en charge.
> Remarque: plusieurs réseaux NAT sont désormais pris en charge avec Windows 10 Creators Update! 

- **transparent**: les conteneurs reliés à un réseau créé avec le pilote «transparent» sont connectés directement au réseau physique. Les adressesIP issues du réseau physique peuvent être attribuées de façon statique (nécessite l'option ``--subnet`` spécifiée par l'utilisateur) ou dynamique à l’aide d’un serveur DHCP externe. 

- **superposition** - __Nouveauté!__  si le moteur Docker s'exécute en [mode Swarm](./swarm-mode.md), les conteneurs reliés à un réseau de superposition peuvent communiquer avec d’autres conteneurs attachés au même réseau entre plusieurs hôtes de conteneur. Chaque réseau de superposition créé dans un cluster Swarm possède son propre sous-réseau IP, défini par un préfixe IP privé. Le pilote réseau de superposition utilise l’encapsulation VXLAN.
> Nécessite Windows Server2016 avec [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217) ou Windows10Creators Update 

- **l2bridge**: les conteneurs reliés à un réseau créé avec le pilote «l2bridge» se trouveront dans le même sous-réseau IP que l’hôte de conteneur. Les adresses IP doivent être attribuées de façon statique à partir du même préfixe que l’hôte de conteneur. Tous les points de terminaison de conteneur sur l’ordinateur hôte ont la même adresse MAC en raison de l’opération de traduction d’adresses de couche2 (réécriture d’adresses MAC) en entrée et en sortie.
> Nécessite Windows Server2016 ou Windows10Creators Update

- **l2tunnel** - _: ce pilote doit uniquement être utilisé dans une pile Microsoft Cloud Stack_.

> Pour savoir comment connecter des points de terminaison de conteneur à un réseau virtuel de superposition avec la pile Microsoft SDN, consultez la rubrique relative à l’[attachement de conteneurs à un réseau virtuel](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network).

> Windows 10 Creators Update a introduit la prise en charge de plateforme pour ajouter un nouveau point de terminaison de conteneur à un conteneur en cours d’exécution (autrement dit, «ajout à chaud»). Cela met en évidence une [requête de tirage Docker en attente](https://github.com/docker/libnetwork/pull/1661) de bout en bout

## Topologies de réseau et IPAM
Le tableau ci-dessous montre de quelle manière est fournie la connectivité réseau pour les connexions internes (conteneur-conteneur) et externes dans chaque pilote réseau.

<figure>
  <img src="media/network-modes-table.png">
</figure>

### IPAM 
Les adresses IP sont attribuées et assignées différemment pour chaque pilote réseau. Windows utilise le service HNS (Host Networking Service) pour fournir une IPAM au pilote nat et fonctionne avec le mode Docker Swarm (KVS interne) pour fournir une IPAM de superposition. Tous les autres pilotes réseau utilisent une IPAM externe.

<figure>
  <img src="media/ipam.png">
</figure>

# Détails de la mise en réseau de conteneur Windows

## Isolation (espace de noms) avec compartiments réseau
Chaque point de terminaison de conteneur est placé dans son propre __compartiment réseau__ qui est identique à un espace de noms réseau Linux. La vNIC de l'hôte de gestion et la pile réseau hôte se trouvent dans le compartiment réseau par défaut. Pour appliquer une isolation réseau entre les conteneurs sur le même hôte, un compartiment réseau est créé pour chaque conteneur Windows Server et Hyper-V dans lequel est installée la carte réseau pour le conteneur. Les conteneurs Windows Server utilisent une carte réseau virtuelle hôte pour s’attacher au commutateur virtuel. Les conteneurs Hyper-V utilisent une carte réseau de machine virtuelle synthétique (non exposée à la machine virtuelle d’utilitaire) pour s’attacher au commutateur virtuel. 

<figure>
  <img src="media/network-compartment-visual.png">
</figure>

```powershell 
Get-NetCompartment
```

## Sécurité du pare-feu Windows

Le pare-feu Windows est utilisé pour appliquer la sécurité réseau par le biais des ACL de ports.

> Remarque: par défaut tous les points de terminaison de conteneur attachés à un réseau de superposition ont une règle AUTORISER TOUT qui est créée   

<figure>
  <img src="media/windows-firewall-containers.png">
</figure>

## Gestion de réseau de conteneur avec le service HNS

L’image ci-dessous montre comment le service HNS et le service de calcul hôte (HCS) fonctionnent ensemble pour créer des conteneurs et attacher des points de terminaison à un réseau. 

<figure>
  <img src="media/HNS-Management-Stack.png">
</figure>

# Options réseau avancées dans Windows
Plusieurs options de pilote réseau sont prises en charge pour tirer parti des capacités et des fonctionnalités spécifiques de Windows. 

## SET (Switch Embedded Teaming) avec les réseaux Docker

> S’applique à tous les pilotes réseau 

Vous pouvez tirer parti de [SET (Switch Embedded Teaming)](https://technet.microsoft.com/en-us/windows-server-docs/networking/technologies/hyper-v-virtual-switch/rdma-and-switch-embedded-teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set) lorsque vous créez des réseaux d'hôte de conteneur à utiliser par Docker, en spécifiant plusieurs cartes réseau (séparées par des virgules) à l'aide de l'option `-o com.docker.network.windowsshim.interface`. 

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2", "Ethernet 3" TeamedNet
```

## Définir l’ID de réseau local virtuel pour un réseau

> S’applique aux pilotes réseau transparent et l2bridge 

Pour définir un ID de réseau local virtuel pour un réseau, utilisez l’option `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` pour la commande `docker network create`. Par exemple, vous pouvez utiliser la commande suivante pour créer un réseau transparent avec l'ID de réseau virtuel local 11:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
Lorsque vous définissez l’ID de réseau virtuel local pour un réseau, vous définissez l’isolation VLAN pour des points de terminaison de conteneur qui seront associés à ce réseau.

> Veillez à ce que votre carte réseau hôte (physique) soit en mode trunk pour permettre à l’ensemble du trafic avec balise d’être traité par le vSwitch avec le port de vNIC (point de terminaison de conteneur) en mode d’accès sur le réseau virtuel local approprié.

## Spécification du nom d’un réseau pour le service HNS

> S’applique à tous les pilotes réseau 

En règle générale, lorsque vous créez un réseau de conteneur à l’aide de `docker network create`, le nom de réseau que vous indiquez est utilisé par le service Docker, mais pas par le service HNS. Si vous créez un réseau, vous pouvez spécifier le nom qui lui est attribué par le service HNS en utilisant l’option `-o com.docker.network.windowsshim.networkname=<network name>` pour la commande `docker network create`. Par exemple, vous pouvez utiliser la commande suivante pour créer un réseau transparent avec un nom spécifié pour le service HNS:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

## Lier un réseau à une interface réseau spécifique

> S’applique à tous les pilotes réseau à l’exception de «nat»  

Pour lier un réseau (connecté via le commutateur virtuel Hyper-V) à une interface réseau spécifique, utilisez l’option `-o com.docker.network.windowsshim.interface=<Interface>` pour la commande `docker network create`. Par exemple, vous pouvez utiliser la commande suivante pour créer un réseau transparent associé à l’interface réseau «Ethernet2»:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> Remarque: la valeur *com.docker.network.windowsshim.interface* correspond au *nom* de la carte réseau, qui est trouvé via:

>```none
PS C:\> Get-NetAdapter
```
## Specify the DNS Suffix and/or the DNS Servers of a Network

> Applies to all network drivers 

Use the option, `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` to specify the DNS suffix of a network, and the option, `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` to specify the DNS servers of a network. For example, you might use the following command to set the DNS suffix of a network to "example.com" and the DNS servers of a network to 4.4.4.4 and 8.8.8.8:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## VFP

L’extension plateforme de filtrage virtuel (VFP, Virtual Filtering Platform) est une extension de transfert de commutateur virtuel Hyper-V utilisée pour appliquer la stratégie réseau et manipuler des paquets. Par exemple, VFP est utilisée par le pilote réseau «superposition» pour effectuer une encapsulation VXLAN et par le pilote «l2bridge» pour effectuer une réécriture d'adresses MAC en entrée et en sortie. L’extension VFP est uniquement présente sur Windows Server2016 et Windows 10 Creators Update. Pour vérifier si elle s’exécute correctement, l'utilisateur exécute deux commandes:

```powershell
Get-Service vfpext

# This should indicate the extension is Running: True 
Get-VMSwitchExtension  -VMSwitchName <vSwitch Name> -Name "Microsoft Azure VFP Switch Extension"
```

## Conseils et informations
Voici une liste de conseils et d'informations pratiques, inspirées de questions courantes sur la mise en réseau de conteneur Windows exprimées par la Communauté...

#### HNS nécessite qu'IPv6 soit activé sur les ordinateurs hôtes de conteneur 
Dans le cadre de [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217), HNS nécessite qu'IPv6 soit activé sur les hôtes de conteneur Windows. Si vous rencontrez une erreur comme celle indiquée ci-dessous, il est possible qu'IPv6 soit désactivé sur votre ordinateur hôte.
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
Nous nous efforçons de modifier la plateforme pour détecter/empêcher automatiquement ce problème. Actuellement, la solution de contournement suivante permet de s'assurer qu'IPv6 est activé sur l'ordinateur hôte:

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```

#### Les machines virtuelles Moby Linux utilisent le commutateur DockerNAT avec Docker pour Windows (un produit de [Docker CE](https://www.docker.com/community-edition)) au lieu du vSwitch interne HNS 
Docker pour Windows (le pilote Windows du moteur Docker CE) sur Windows10 utilise un vSwitch interne nommé «DockerNAT» pour connecter des machines virtuelles Moby Linux à l’hôte de conteneur. Les développeurs qui utilisent des machines virtuelles Moby Linux sur Windows doivent savoir que leurs ordinateurs hôtes, utilisent le vSwitch DockerNAT au lieu du vSwitch créé par le service HNS (qui est le commutateur par défaut utilisé pour les conteneurs Windows). 

#### Pour utiliser DHCP pour l'attribution d'adresses IP sur un hôte de conteneur virtuel, vous devez activer MACAddressSpoofing 
Si l’hôte de conteneur est virtualisé et que vous souhaitez utiliser DHCP pour l’attribution d’adressesIP, vous devez activer MACAddressSpoofing sur la carte réseau de la machine virtuelle. Sinon, l’hôte Hyper-V bloque le trafic réseau provenant des conteneurs sur la machine virtuelle utilisant plusieurs adresses MAC. Vous pouvez activer MACAddressSpoofing avec cette commande PowerShell:
```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
#### Création de plusieurs réseaux transparents sur un hôte de conteneur unique
Pour créer plusieurs réseaux transparents, vous devez spécifier la carte réseau (virtuelle) à laquelle le commutateur virtuel Hyper-V externe doit être lié. Pour spécifier l’interface pour un réseau, utilisez la syntaxe suivante:
```
# General syntax:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface=<INTERFACE NAME> <NETWORK NAME> 

# Example:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" myTransparent2
```

#### N’oubliez pas de spécifier *--subnet* et *--gateway* quand vous utilisez l’attribution d'adresses IP statiques
Quand vous utilisez l’attribution d'adresses IP statiques, assurez-vous d’abord que les paramètres *--subnet* et *--gateway* sont spécifiés au moment de la création du réseau. L’adresse IP de sous-réseau et de passerelle doit être la même que les paramètres réseau de l’hôte de conteneur (le réseau physique). Par exemple, voici comment créer un réseau transparent, puis exécuter un point de terminaison sur ce réseau à l’aide de l’attribution d'adresses IP statiques:

```
# Example: Create a transparent network using static IP assignment
# A network create command for a transparent container network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 MyTransparentNet
# Run a container attached to MyTransparentNet
C:\> docker run -it --network=MyTransparentNet --ip=10.123.174.105 windowsservercore cmd
```

#### L'attribution d'adresses IP statiques par DHCP n'est pas pris en charge avec les réseaux L2Bridge
Seule l’attribution d'adresses IP statiques est prise en charge avec les réseaux de conteneur créés à l’aide du pilote l2bridge. Comme indiqué ci-dessus, n’oubliez pas d’utiliser les paramètres *--subnet* et *--gateway* pour créer un réseau configuré pour l’attribution d'adresses IP statiques.

#### Chaque réseau qui exploite un vSwitch externe doit avoir sa propre carte réseau
Si plusieurs réseaux utilisant un vSwitch externe pour la connectivité (par exemple, Transparent, L2 Bridge, L2 Transparent) sont créés sur le même hôte de conteneur, chacun d’eux requiert sa propre carte réseau. 

#### Attribution d'adresses IP sur des conteneurs arrêtés ou en cours d’exécution
L’attribution d’adressesIP statiques est effectuée directement sur la carte réseau du conteneur et ne doit être réalisée que quand le conteneur est dans un état ARRÊTÉ. Ni un «ajout à chaud» de cartes réseau de conteneurs ni un apport de modifications à la pile réseau n'est pris en charge (dans Windows Server2016) pendant l’exécution du conteneur.
> Remarque: ce comportement change sur Windows10 Creators Update, la plateforme prenant désormais en charge l'«ajout à chaud». Cette fonctionnalité mettra en évidence E2E après la fusion de cette [requête de tirage Docker en attente](https://github.com/docker/libnetwork/pull/1661)

#### Un vSwitch existant (non visible pour Docker) peut bloquer la création de réseau transparent
Si vous rencontrez une erreur lors de la création d'un réseau transparent, il est possible qu’il existe un vSwitch externe sur votre système qui n’a pas été découvert automatiquement par Docker, et qui par conséquent empêche la liaison du réseau transparent à la carte réseau externe de votre hôte de conteneur. 

Quand vous créez un réseau transparent, Docker crée un commutateur virtuel externe pour le réseau, puis essaie de lier le commutateur à une carte réseau (externe): l’adaptateur peut être une carte réseau de machine virtuelle ou la carte réseau physique. Si un commutateur virtuel a déjà été créé sur l’hôte du conteneur *et qu’il est visible par Docker*, le moteur Docker Windows utilise ce commutateur au lieu d’en créer un nouveau. Cependant, si le commutateur virtuel a été créé hors bande (c’est-à-dire sur l’hôte du conteneur à l’aide du gestionnaire Hyper-V ou de PowerShell) et qu’il n’est pas encore visible par Docker, le moteur Docker Windows tente de créer un commutateur virtuel sans parvenir par la suite à connecter le nouveau commutateur à la carte réseau externe de l’hôte de conteneur (car la carte réseau est déjà connectée au commutateur qui a été créé hors bande).

Par exemple, ce problème peut se produire si vous avez d’abord créé un commutateur virtuel sur votre hôte alors que le service Docker était en cours d’exécution, puis que vous avez essayé de créer un réseau transparent. Dans ce cas, Docker ne reconnaît pas le commutateur que vous avez créé et crée un commutateur virtuel pour le réseau transparent.

Il existe trois approches pour résoudre ce problème:

* Vous pouvez bien entendu supprimer le commutateur virtuel qui a été créé hors bande, ce qui permet à Docker de créer un commutateur virtuel et de le connecter à la carte réseau hôte sans problème. Avant de choisir cette approche, vérifiez que votre commutateur virtuel hors bande n’est pas utilisé par d’autres services (par exemple Hyper-V).
* Si vous décidez d’utiliser un commutateur virtuel externe qui a été créé hors bande, vous pouvez aussi redémarrer les services Docker et HNS pour *rendre le commutateur visible par Docker*.
```none
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* Une autre option consiste à utiliser l’option «-o com.docker.network.windowsshim.interface» pour lier le commutateur externe du réseau transparent à une carte réseau spécifique qui n’est pas déjà utilisée sur l’hôte du conteneur (par exemple une carte réseau autre que celle utilisée par le commutateur virtuel qui a été créé hors bande). L’option «-o» est décrite ci-dessus, dans la section [Réseau transparent](https://msdn.microsoft.com/virtualization/windowscontainers/management/container_networking#transparent-network) de ce document.


## Fonctionnalités et options réseau non prises en charge 

Les options de mise en réseau suivantes ne sont pas prises en charge dans Windows et ne peuvent pas être passées dans ``docker run``:
 * Liaison de conteneur (par exemple, ``--link``):_l'alternative consiste à s’appuyer sur la découvertes des services_
 * Adresses IPv6 (par exemple, ``--ip6``)
 * Options DNS (par exemple, ``--dns-option``)
 * Plusieurs domaines de recherche DNS (par exemple, ``--dns-search``)
 
Les options et fonctionnalités de mise en réseau suivantes ne sont pas prises en charge dans Windows et ne peuvent pas être passées dans ``docker network create``:
 * --aux-address
 * --internal
 * --ip-range
 * --ipam-driver
 * --ipam-opt
 * --ipv6 

Les options de mise en réseau suivantes ne sont pas prises en charge sur les services Docker
* Chiffrement du plan de données (par exemple, ``--opt encrypted``) 


## Solutions de contournement de Windows Server2016 

Bien que nous continuions à ajouter de nouvelles fonctionnalités et à contribuer au développement, certaines de ces fonctionnalités ne seront pas répercutées sur des plateformes plus anciennes. Le meilleur plan d’action consiste plutôt à «prendre le train en marche» avec les dernières mises à jour vers Windows10 et Windows Server.  La section ci-dessous indique certaines réserves et solutions de contournement qui s’appliquent à Windows Server2016 et aux versions antérieures de Windows10 (c'est-à-dire avant la version1704 de Creators Update)

### Plusieurs réseaux NAT sur l’hôte de conteneur WS2016

Les partitions des nouveaux réseaux NAT doivent être créées sous le préfixe du réseau NAT le plus grand. Vous pouvez trouver le préfixe en exécutant la commande suivante à partir de PowerShell et en référençant le champ «InternalIPInterfaceAddressPrefix».

```none
PS C:\> Get-NetNAT
```

Par exemple, le préfixe interne du réseau NAT de l’hôte peut être 172.16.0.0/16. Dans ce cas, Docker peut être utilisé pour créer d’autres réseaux NAT *dès lors qu’ils sont un sous-ensemble du préfixe 172.16.0.0/16*. Par exemple, deux réseaux NAT peuvent être créés avec les préfixes IP 172.16.1.0/24 (passerelle 172.16.1.1) et 172.16.2.0/24 (passerelle 172.16.2.1).

```none
C:\> docker network create -d nat --subnet=172.16.1.0/24 --gateway=172.16.1.1 CustomNat1
C:\> docker network create -d nat --subnet=172.16.2.0/24 --gateway=172.16.1.1 CustomNat2
```

Vous pouvez répertorier les réseaux nouvellement créés en utilisant:
```none
C:\> docker network ls
```

### Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) peut être utilisé pour définir et configurer des réseaux de conteneur en même temps que les conteneurs/services qui utiliseront ces réseaux. La clé «networks» de Compose est utilisée comme clé de plus haut niveau dans la définition des réseaux auxquels les conteneurs sont connectés. Par exemple, la syntaxe suivante définit le réseau NAT préexistant créé par Docker comme réseau «default» pour tous les conteneurs/services définis dans un fichier Compose donné.

```none
networks:
 default:
  external:
   name: "nat"
```

De même, la syntaxe suivante peut être utilisée pour définir un réseau NAT personnalisé.

> Remarque: Le «réseau NAT personnalisé» défini dans l’exemple ci-dessous est défini comme une partition du préfixe interne NAT préexistant de l’hôte du conteneur. Pour plus de contexte, consultez la section ci-dessus, «Réseaux NAT multiples».

```none
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.16.3.0/24
```

Pour plus d’informations sur la définition et la configuration des réseaux de conteneur avec Docker Compose, consultez [Compose File reference](https://docs.docker.com/compose/compose-file/).

### Découverte des services
La découverte des services est uniquement prise en charge pour certains pilotes réseau Windows.

|  | Découverte des services locale  | Découverte des services globale |
| :---: | :---------------     |  :---                |
| nat | OUI | N/A |  
| superposition | OUI | OUI |
| transparent | NON | NON |
| l2bridge | NON | NON |



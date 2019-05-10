---
title: Mise en réseau de conteneur Windows
description: Mise en réseau avancée pour les conteneurs Windows.
keywords: docker, conteneurs
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 492e3b0ba3b1abe1109de3f6091f5b60831036df
ms.sourcegitcommit: aaf115a9de929319cc893c29ba39654a96cf07e1
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/10/2019
ms.locfileid: "9622974"
---
# <a name="advanced-network-options-in-windows"></a>Options réseau avancées dans Windows

Plusieurs options de pilote réseau sont prises en charge pour tirer parti des capacités et des fonctionnalités spécifiques de Windows. 

## <a name="switch-embedded-teaming-with-docker-networks"></a>SET (Switch Embedded Teaming) avec les réseaux Docker

> S’applique à tous les pilotes réseau

Vous pouvez tirer parti de [SET (Switch Embedded Teaming)](https://docs.microsoft.com/windows-server/virtualization/hyper-v-virtual-switch/RDMA-and-Switch-Embedded-Teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set) lorsque vous créez des réseaux d'hôte de conteneur à utiliser par Docker, en spécifiant plusieurs cartes réseau (séparées par des virgules) à l'aide de l'option `-o com.docker.network.windowsshim.interface`.

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2", "Ethernet 3" TeamedNet
```

## <a name="set-the-vlan-id-for-a-network"></a>Définir l’ID de réseau local virtuel pour un réseau

> S’applique aux pilotes réseau transparent et l2bridge

Pour définir un ID de réseau local virtuel pour un réseau, utilisez l’option `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` pour la commande `docker network create`. Par exemple, vous pouvez utiliser la commande suivante pour créer un réseau transparent avec l'ID de réseau virtuel local 11:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
Lorsque vous définissez l’ID de réseau virtuel local pour un réseau, vous définissez l’isolation VLAN pour des points de terminaison de conteneur qui seront associés à ce réseau.

> Veillez à ce que votre carte réseau hôte (physique) soit en mode trunk pour permettre à l’ensemble du trafic avec balise d’être traité par le vSwitch avec le port de vNIC (point de terminaison de conteneur) en mode d’accès sur le réseau virtuel local approprié.

## <a name="specify-outboundnat-policy-for-a-network"></a>Spécifier la stratégie de OutboundNAT pour un réseau

> S’applique aux réseaux l2bridge

En règle générale, lorsque vous créez un `l2bridge` réseau de conteneur à l’aide `docker network create`, points de terminaison de conteneur n’ont pas d’une stratégie de OutboundNAT HNS appliquée, ce qui entraîne l’impossibilité d’atteindre l’extérieur de conteneurs. Si vous créez un réseau, vous pouvez utiliser la `-o com.docker.network.windowsshim.enable_outboundnat=<true|false>` permet d’appliquer la stratégie OutboundNAT HNS pour donner accès aux conteneurs à l’extérieur:

```
C:\> docker network create -d l2bridge -o com.docker.network.windowsshim.enable_outboundnat=true MyL2BridgeNetwork
```

S’il existe un ensemble de destinations (par exemple, une connectivité conteneur à l’autre est nécessaire) pour où nous ne voulons pas NAT'ing se produise, nous devons également spécifier un ExceptionList:

```
C:\> docker network create -d l2bridge -o com.docker.network.windowsshim.enable_outboundnat=true -o com.docker.network.windowsshim.outboundnat_exceptions=10.244.10.0/24
```

## <a name="specify-the-name-of-a-network-to-the-hns-service"></a>Spécification du nom d’un réseau pour le service HNS

> S’applique à tous les pilotes réseau 

En règle générale, lorsque vous créez un réseau de conteneur à l’aide de `docker network create`, le nom de réseau que vous indiquez est utilisé par le service Docker, mais pas par le service HNS. Si vous créez un réseau, vous pouvez spécifier le nom qui lui est attribué par le service HNS en utilisant l’option `-o com.docker.network.windowsshim.networkname=<network name>` pour la commande `docker network create`. Par exemple, vous pouvez utiliser la commande suivante pour créer un réseau transparent avec un nom spécifié pour le service HNS:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

## <a name="bind-a-network-to-a-specific-network-interface"></a>Lier un réseau à une interface réseau spécifique

> S’applique à tous les pilotes réseau à l’exception de «nat»  

Pour lier un réseau (connecté via le commutateur virtuel Hyper-V) à une interface réseau spécifique, utilisez l’option `-o com.docker.network.windowsshim.interface=<Interface>` pour la commande `docker network create`. Par exemple, vous pouvez utiliser la commande suivante pour créer un réseau transparent associé à l’interface réseau «Ethernet2»:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> Remarque: la valeur *com.docker.network.windowsshim.interface* correspond au *nom* de la carte réseau, qui est trouvé via:

```
PS C:\> Get-NetAdapter
```

## <a name="specify-the-dns-suffix-andor-the-dns-servers-of-a-network"></a>Spécifier le suffixe DNS et/ou des serveurs DNS d’un réseau

> S’applique à tous les pilotes réseau 

Utilisez l’option `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` pour spécifier le suffixe DNS d’un réseau et l’option `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` pour spécifier les serveurs DNS d’un réseau. Par exemple, vous pouvez utiliser la commande suivante pour définir le suffixe DNS d’un réseau sur «example.com», et les serveurs DNS d’un réseau sur 4.4.4.4 et 8.8.8.8:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## <a name="vfp"></a>VFP

Pour plus d’informations, consultez [cet article](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/).

## <a name="tips--insights"></a>Conseils et informations
Voici une liste de conseils et d'informations pratiques, inspirées de questions courantes sur la mise en réseau de conteneur Windows exprimées par la Communauté...

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>HNS nécessite qu'IPv6 soit activé sur les ordinateurs hôtes de conteneur 
Dans le cadre de [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217), HNS nécessite qu'IPv6 soit activé sur les hôtes de conteneur Windows. Si vous rencontrez une erreur comme celle indiquée ci-dessous, il est possible qu'IPv6 soit désactivé sur votre ordinateur hôte.
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
Nous nous efforçons de modifier la plateforme pour détecter/empêcher automatiquement ce problème. Actuellement, la solution de contournement suivante permet de s’assurer qu’IPv6 est activé sur l’ordinateur hôte:

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```


#### <a name="linux-containers-on-windows"></a>Conteneurs Linux sur Windows

**Nouveauté:** nous travaillons à rendre possible l’exécution de conteneurs Linux et Windows côte-à-côte _sans l’ordinateur virtuel Moby Linux_. Consultez ce [billet de blog sur les conteneurs Linux sur Windows (LCOW)](https://blog.docker.com/2017/11/docker-for-windows-17-11/) pour plus d’informations. Voici comment [faire](https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-10-linux).
> Remarque: LCOW remplace l’ordinateur virtuel Moby Linux, il utilisera le vSwitch interne HNS «nat» par défaut.

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-cehttpswwwdockercomcommunity-edition"></a>Les ordinateurs virtuels Moby Linux utilisent le commutateur DockerNAT avec Docker pour Windows (un produit de [Docker CE](https://www.docker.com/community-edition))

Docker pour Windows (le pilote Windows du moteur Docker CE) sur Windows10 utilise un vSwitch interne nommé «DockerNAT» pour connecter des ordinateurs virtuels Moby Linux à l’hôte de conteneur. Les développeurs qui utilisent des machines virtuelles Moby Linux sur Windows doivent savoir que leurs ordinateurs hôtes, utilisent le vSwitch DockerNAT au lieu du vSwitch «nat» créé par le service HNS (qui est le commutateur par défaut utilisé pour les conteneurs Windows).



#### <a name="to-use-dhcp-for-ip-assignment-on-a-virtual-container-host-enable-macaddressspoofing"></a>Pour utiliser DHCP pour l'attribution d'adresses IP sur un hôte de conteneur virtuel, vous devez activer MACAddressSpoofing

Si l’hôte de conteneur est virtualisé et que vous souhaitez utiliser DHCP pour l’attribution d’adressesIP, vous devez activer MACAddressSpoofing sur la carte réseau de la machine virtuelle. Sinon, l’hôte Hyper-V bloque le trafic réseau provenant des conteneurs sur la machine virtuelle utilisant plusieurs adresses MAC. Vous pouvez activer MACAddressSpoofing avec cette commande PowerShell:
```
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
Si vous exécutez VMware en tant qu’hyperviseur, vous devez activer le mode Promiscuous pour que cela fonctionne. Pour plus d’informations, [reportez-vous ici](https://kb.vmware.com/s/article/1004099)


#### <a name="creating-multiple-transparent-networks-on-a-single-container-host"></a>Création de plusieurs réseaux transparents sur un hôte de conteneur unique
Pour créer plusieurs réseaux transparents, vous devez spécifier la carte réseau (virtuelle) à laquelle le commutateur virtuel Hyper-V externe doit être lié. Pour spécifier l’interface pour un réseau, utilisez la syntaxe suivante:
```
# General syntax:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface=<INTERFACE NAME> <NETWORK NAME> 

# Example:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" myTransparent2
```

#### <a name="remember-to-specify---subnet-and---gateway-when-using-static-ip-assignment"></a>N’oubliez pas de spécifier *--subnet* et *--gateway* quand vous utilisez l’attribution d'adresses IP statiques
Quand vous utilisez l’attribution d'adresses IP statiques, assurez-vous d’abord que les paramètres *--subnet* et *--gateway* sont spécifiés au moment de la création du réseau. L’adresse IP de sous-réseau et de passerelle doit être la même que les paramètres réseau de l’hôte de conteneur (le réseau physique). Par exemple, voici comment créer un réseau transparent, puis exécuter un point de terminaison sur ce réseau à l’aide de l’attribution d'adresses IP statiques:

```
# Example: Create a transparent network using static IP assignment
# A network create command for a transparent container network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 MyTransparentNet
# Run a container attached to MyTransparentNet
C:\> docker run -it --network=MyTransparentNet --ip=10.123.174.105 windowsservercore cmd
```

#### <a name="dhcp-ip-assignment-not-supported-with-l2bridge-networks"></a>L'attribution d'adresses IP statiques par DHCP n'est pas pris en charge avec les réseaux L2Bridge
Seule l’attribution d'adresses IP statiques est prise en charge avec les réseaux de conteneur créés à l’aide du pilote l2bridge. Comme indiqué ci-dessus, n’oubliez pas d’utiliser les paramètres *--subnet* et *--gateway* pour créer un réseau configuré pour l’attribution d'adresses IP statiques.

#### <a name="networks-that-leverage-external-vswitch-must-each-have-their-own-network-adapter"></a>Chaque réseau qui exploite un vSwitch externe doit avoir sa propre carte réseau
Si plusieurs réseaux utilisant un vSwitch externe pour la connectivité (par exemple, Transparent, L2 Bridge, L2 Transparent) sont créés sur le même hôte de conteneur, chacun d’eux requiert sa propre carte réseau. 

#### <a name="ip-assignment-on-stopped-vs-running-containers"></a>Attribution d'adresses IP sur des conteneurs arrêtés ou en cours d’exécution
L’attribution d’adressesIP statiques est effectuée directement sur la carte réseau du conteneur et ne doit être réalisée que quand le conteneur est dans un état ARRÊTÉ. Ni un «ajout à chaud» de cartes réseau de conteneurs ni un apport de modifications à la pile réseau n'est pris en charge (dans Windows Server2016) pendant l’exécution du conteneur.
> Remarque: ce comportement change sur Windows10 Creators Update, la plateforme prenant désormais en charge l'«ajout à chaud». Cette fonctionnalité mettra en évidence E2E après la fusion de cette [requête de tirage Docker en attente](https://github.com/docker/libnetwork/pull/1661)

#### <a name="existing-vswitch-not-visible-to-docker-can-block-transparent-network-creation"></a>Un vSwitch existant (non visible pour Docker) peut bloquer la création de réseau transparent
Si vous rencontrez une erreur lors de la création d'un réseau transparent, il est possible qu’il existe un vSwitch externe sur votre système qui n’a pas été découvert automatiquement par Docker, et qui par conséquent empêche la liaison du réseau transparent à la carte réseau externe de votre hôte de conteneur. 

Quand vous créez un réseau transparent, Docker crée un commutateur virtuel externe pour le réseau, puis essaie de lier le commutateur à une carte réseau (externe): l’adaptateur peut être une carte réseau de machine virtuelle ou la carte réseau physique. Si un commutateur virtuel a déjà été créé sur l’hôte du conteneur *et qu’il est visible par Docker*, le moteur Docker Windows utilise ce commutateur au lieu d’en créer un nouveau. Cependant, si le commutateur virtuel a été créé hors bande (c’est-à-dire sur l’hôte du conteneur à l’aide du gestionnaire Hyper-V ou de PowerShell) et qu’il n’est pas encore visible par Docker, le moteur Docker Windows tente de créer un commutateur virtuel sans parvenir par la suite à connecter le nouveau commutateur à la carte réseau externe de l’hôte de conteneur (car la carte réseau est déjà connectée au commutateur qui a été créé hors bande).

Par exemple, ce problème peut se produire si vous avez d’abord créé un commutateur virtuel sur votre hôte alors que le service Docker était en cours d’exécution, puis que vous avez essayé de créer un réseau transparent. Dans ce cas, Docker ne reconnaît pas le commutateur que vous avez créé et crée un commutateur virtuel pour le réseau transparent.

Il existe trois approches pour résoudre ce problème:

* Vous pouvez bien entendu supprimer le commutateur virtuel qui a été créé hors bande, ce qui permet à Docker de créer un commutateur virtuel et de le connecter à la carte réseau hôte sans problème. Avant de choisir cette approche, vérifiez que votre commutateur virtuel hors bande n’est pas utilisé par d’autres services (par exemple Hyper-V).
* Si vous décidez d’utiliser un commutateur virtuel externe qui a été créé hors bande, vous pouvez aussi redémarrer les services Docker et HNS pour *rendre le commutateur visible par Docker*.
```
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* Une autre option consiste à utiliser l’option «-o com.docker.network.windowsshim.interface» pour lier le commutateur externe du réseau transparent à une carte réseau spécifique qui n’est pas déjà utilisée sur l’hôte du conteneur (par exemple une carte réseau autre que celle utilisée par le commutateur virtuel qui a été créé hors bande). La «-o ' option est décrite dans la section [Création de plusieurs réseaux transparents sur un hôte de conteneur unique](advanced.md#creating-multiple-transparent-networks-on-a-single-container-host) de ce document.


## <a name="windows-server-2016-work-arounds"></a>Solutions de contournement de Windows Server2016 

Bien que nous continuions à ajouter de nouvelles fonctionnalités et à contribuer au développement, certaines de ces fonctionnalités ne seront pas répercutées sur des plateformes plus anciennes. Le meilleur plan d’action consiste plutôt à «prendre le train en marche» avec les dernières mises à jour vers Windows10 et Windows Server.  La section ci-dessous indique certaines réserves et solutions de contournement qui s’appliquent à Windows Server2016 et aux versions antérieures de Windows10 (c'est-à-dire avant la version1704 de Creators Update)

### <a name="multiple-nat-networks-on-ws2016-container-host"></a>Plusieurs réseaux NAT sur l’hôte de conteneur WS2016

Les partitions des nouveaux réseaux NAT doivent être créées sous le préfixe du réseau NAT le plus grand. Vous pouvez trouver le préfixe en exécutant la commande suivante à partir de PowerShell et en référençant le champ «InternalIPInterfaceAddressPrefix».

```
PS C:\> Get-NetNAT
```

Par exemple, le préfixe interne du réseau NAT de l’hôte peut être 172.16.0.0/16. Dans ce cas, Docker peut être utilisé pour créer d’autres réseaux NAT *dès lors qu’ils sont un sous-ensemble du préfixe 172.16.0.0/16*. Par exemple, deux réseaux NAT peuvent être créés avec les préfixes IP 172.16.1.0/24 (passerelle 172.16.1.1) et 172.16.2.0/24 (passerelle 172.16.2.1).

```
C:\> docker network create -d nat --subnet=172.16.1.0/24 --gateway=172.16.1.1 CustomNat1
C:\> docker network create -d nat --subnet=172.16.2.0/24 --gateway=172.16.1.1 CustomNat2
```

Vous pouvez répertorier les réseaux nouvellement créés en utilisant:
```
C:\> docker network ls
```

### <a name="docker-compose"></a>Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) peut être utilisé pour définir et configurer des réseaux de conteneur en même temps que les conteneurs/services qui utiliseront ces réseaux. La clé «networks» de Compose est utilisée comme clé de plus haut niveau dans la définition des réseaux auxquels les conteneurs sont connectés. Par exemple, la syntaxe suivante définit le réseau NAT préexistant créé par Docker comme réseau «default» pour tous les conteneurs/services définis dans un fichier Compose donné.

```
networks:
 default:
  external:
   name: "nat"
```

De même, la syntaxe suivante peut être utilisée pour définir un réseau NAT personnalisé.

> Remarque: Le «réseau NAT personnalisé» défini dans l’exemple ci-dessous est défini comme une partition du préfixe interne NAT préexistant de l’hôte du conteneur. Pour plus de contexte, consultez la section ci-dessus, «Réseaux NAT multiples».

```
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.16.3.0/24
```

Pour plus d’informations sur la définition et la configuration des réseaux de conteneur avec Docker Compose, consultez [référence concernant le fichier Compose](https://docs.docker.com/compose/compose-file/).
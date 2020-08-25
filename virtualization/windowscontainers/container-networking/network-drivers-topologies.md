---
title: Pilotes réseau de conteneur Windows
description: Pilotes réseau et topologies pour les conteneurs Windows.
keywords: docker, conteneurs
author: daschott
ms.date: 08/13/2020
ms.topic: conceptual
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: d28b1cf2bd8295001b26b821c1269efb1f7b4070
ms.sourcegitcommit: ecbee9197074eeb0ae612b28fcf1144e28bf9789
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/25/2020
ms.locfileid: "88810119"
---
# <a name="windows-container-network-drivers"></a>Pilotes réseau de conteneur Windows

En plus de tirer parti du réseau « nat » par défaut créé par Docker sur Windows, les utilisateurs peuvent définir des réseaux de conteneurs personnalisés. Les réseaux définis par l’utilisateur peuvent être créés à l’aide de la commande CLI de l’amarrage [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) . Sur Windows, les types de pilote réseau suivants sont disponibles :


### <a name="nat-network-driver"></a>Pilote réseau NAT

Les conteneurs attachés à un réseau créé avec le pilote « NAT » sont connectés à un commutateur Hyper-V *interne* et reçoivent une adresse IP du préfixe IP spécifié par l’utilisateur ( ``--subnet`` ). Le réacheminement/mappage de ports à partir de l’hôte de conteneur vers des points de terminaison de conteneur est pris en charge.

  >[!TIP]
  > Il est possible de personnaliser le sous-réseau utilisé par le réseau « NAT » par défaut via le `fixed-cidr` paramètre du [fichier de configuration du démon](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)de l’ancrage.

  >[!NOTE]
  > Les réseaux NAT créés sur Windows Server 2019 (ou version ultérieure) ne sont plus conservés après le redémarrage.

##### <a name="creating-a-nat-network"></a>Création d’un réseau NAT

Pour créer un réseau NAT avec sous-réseau `10.244.0.0/24` :
```powershell
docker network create -d "nat" --subnet "10.244.0.0/24" my_nat
```


### <a name="transparent-network-driver"></a>Pilote réseau transparent

Les conteneurs attachés à un réseau créé avec le pilote « transparent » seront directement connectés au réseau physique via un commutateur Hyper-V *externe* . Les adresses IP issues du réseau physique peuvent être attribuées de façon statique (nécessite l'option ``--subnet`` spécifiée par l'utilisateur) ou dynamique à l’aide d’un serveur DHCP externe.

  >[!NOTE]
  >En raison des exigences suivantes, la connexion de vos hôtes de conteneur sur un réseau transparent n’est pas prise en charge sur les machines virtuelles Azure.

  > Requiert : quand ce mode est utilisé dans un scénario de virtualisation (un hôte de conteneur est une machine virtuelle), une _usurpation d’adresse Mac est nécessaire_.

##### <a name="creating-a-transparent-network"></a>Création d’un réseau transparent

Pour créer un réseau transparent avec sous-réseau `10.244.0.0/24` , passerelle `10.244.0.1` , serveur DNS `10.244.0.7` et ID de réseau local virtuel `7` :
```powershell
docker network create -d "transparent" --subnet 10.244.0.0/24 --gateway 10.244.0.1 -o com.docker.network.windowsshim.vlanid=7 -o com.docker.network.windowsshim.dnsservers="10.244.0.7" my_transparent
```


### <a name="overlay-network-driver"></a>Pilote réseau de superposition 

Couramment utilisé par les orchestrateurs de conteneurs tels que le Dockeur essaim et Kubernetes, les conteneurs attachés à un réseau de superposition peuvent communiquer avec d’autres conteneurs attachés au même réseau sur plusieurs hôtes de conteneur. Chaque réseau de superposition est créé avec son propre sous-réseau IP, défini par un préfixe d’adresse IP privée. Le pilote de réseau de superposition utilise l’encapsulation VXLAN pour assurer l’isolement du trafic réseau entre les réseaux de conteneurs de locataire et permet la réutilisation d’adresses IP sur des réseaux de superposition.
  > Nécessite : Assurez-vous que votre environnement satisfait aux [conditions préalables](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) requises pour créer des réseaux de superposition.

  > Requiert : sur Windows Server 2019, cela nécessite [KB4489899](https://support.microsoft.com/help/4489899).

  > Requiert : sur Windows Server 2016, cela nécessite [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217).

  >[!NOTE]
  >Sur Windows Server 2019 et versions ultérieures, les réseaux de superposition créés par l’Assistant de connexion par essaim tirent parti des règles NAT VFP pour la connectivité sortante. Cela signifie qu’un conteneur donné reçoit 1 adresse IP. Cela signifie également que les outils basés sur ICMP tels que `ping` ou `Test-NetConnection` doivent être configurés à l’aide de leurs options TCP/UDP dans des situations de débogage.

##### <a name="creating-a-overlay-network"></a>Création d’un réseau de superposition

Pour créer un réseau de superposition avec un sous-réseau `10.244.0.0/24` , un serveur DNS et une extension \ système `168.63.129.16` `4096` :
```powershell
docker network create -d "overlay" --attachable --subnet "10.244.0.0/24" -o com.docker.network.windowsshim.dnsservers="168.63.129.16" -o com.docker.network.driver.overlay.vxlanid_list="4096" my_overlay
```


### <a name="l2bridge-network-driver"></a>Pilote réseau L2bridge

Les conteneurs attachés à un réseau créé avec le pilote « l2bridge » sont connectés au réseau physique via un commutateur Hyper-V *externe* . Dans l2bridge, le trafic réseau de conteneurs aura la même adresse MAC que l’hôte en raison de l’opération de traduction d’adresse de couche 2 (réécriture MAC) sur l’entrée et la sortie. Dans les centres de centres, cela permet de réduire la contrainte sur les commutateurs qui ont besoin d’apprendre les adresses MAC des conteneurs parfois à courte durée de vie. Les réseaux L2bridge peuvent être configurés de deux manières différentes :
  1. Le réseau L2bridge est configuré avec le même sous-réseau IP que l’hôte de conteneur
  2. Le réseau L2bridge est configuré avec un nouveau sous-réseau IP personnalisé

  Dans la configuration 2, les utilisateurs doivent ajouter un point de terminaison sur le compartiment réseau hôte qui agit en tant que passerelle et configurer des fonctionnalités de routage pour le préfixe désigné.

##### <a name="creating-a-l2bridge-network"></a>Création d’un réseau l2bridge

Pour créer un réseau l2bridge avec le sous-réseau `10.244.0.0/24` , la passerelle `10.244.0.1` , le serveur DNS et l’ID de réseau `10.244.0.7` local virtuel 7 :
```powershell
docker network create -d "l2bridge" --subnet 10.244.0.0/24 --gateway 10.244.0.1 -o com.docker.network.windowsshim.vlanid=7 -o com.docker.network.windowsshim.dnsservers="10.244.0.7" my_transparent
```

  >[!TIP]
  >Les réseaux L2bridge sont hautement programmables ; Vous trouverez plus d’informations sur la configuration de l2bridge [ici](https://techcommunity.microsoft.com/t5/networking-blog/l2bridge-container-networking/ba-p/1180923).


### <a name="l2tunnel-network-driver"></a>Pilote réseau L2tunnel

La création est identique à l2bridge, mais _ce pilote ne doit être utilisé que dans une pile de Microsoft Cloud (Azure)_. La seule différence par rapport à l2bridge est que tout le trafic de conteneur est envoyé à l’hôte de virtualisation où la stratégie SDN est appliquée, ce qui permet d’activer des fonctionnalités telles que les [groupes de sécurité réseau Azure](/azure/virtual-network/security-overview) pour les conteneurs.


## <a name="network-topologies-and-ipam"></a>Topologies de réseau et IPAM

Le tableau ci-dessous montre de quelle manière est fournie la connectivité réseau pour les connexions internes (conteneur-conteneur) et externes dans chaque pilote réseau.

### <a name="networking-modesdocker-drivers"></a>Modes de mise en réseau/pilotes de l’ancrage

  | Pilote réseau Windows de l’amarrage | Utilisations classiques | Conteneur à conteneur (nœud unique) | Conteneur vers externe (nœud unique + plusieurs nœuds) | Conteneur à conteneur (nœuds multiples) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (par défaut)** | Idéal pour les développeurs | <ul><li>Même sous-réseau : connexion par pont via le commutateur virtuel Hyper-V</li><li> Sous-réseau croisé : non pris en charge (un seul préfixe NAT interne)</li></ul> | Routé via la gestion carte réseau virtuelle (lié à Winnat) | Non pris en charge directement : nécessite l’exposition des ports via l’hôte |
  | **Transparente** | Idéal pour les développeurs ou les petits déploiements | <ul><li>Même sous-réseau : connexion par pont via le commutateur virtuel Hyper-V</li><li>Inter-réseaux : routé via un hôte de conteneur</li></ul> | Routé via un hôte de conteneur avec accès direct à la carte réseau (physique) | Routé via un hôte de conteneur avec accès direct à la carte réseau (physique) |
  | **Superposition** | Adapté à plusieurs nœuds ; requis pour le essaimeur essaim, disponible dans Kubernetes | <ul><li>Même sous-réseau : connexion par pont via le commutateur virtuel Hyper-V</li><li>Sous-réseau croisé : le trafic réseau est encapsulé et routé via la gestion carte réseau virtuelle</li></ul> | Non directement pris en charge : nécessite un deuxième point de terminaison de conteneur attaché au réseau NAT sur Windows Server 2016 ou une règle NAT VFP sur Windows Server 2019.  | Identique/sous-réseau croisé : le trafic réseau est encapsulé à l’aide de VXLAN et routé via Mgmt carte réseau virtuelle |
  | **L2Bridge** | Utilisé pour Kubernetes et Microsoft SDN | <ul><li>Même sous-réseau : connexion par pont via le commutateur virtuel Hyper-V</li><li> Sous-réseau croisé : adresse MAC du conteneur réécrite à l’entrée et à la sortie et acheminée</li></ul> | Réécriture de l’adresse MAC du conteneur à l’entrée et à la sortie | <ul><li>Même sous-réseau : connexion en pont</li><li>Inter-réseaux : routé via la gestion carte réseau virtuelle sur WSv1809 et versions ultérieures</li></ul> |
  | **L2Tunnel**| Azure uniquement | Identique/sous-réseau entre les cheveux et le commutateur virtuel Hyper-V de l’hôte physique vers l’emplacement d’application de la stratégie | Le trafic doit passer par la passerelle de réseau virtuel Azure | Identique/sous-réseau entre les cheveux et le commutateur virtuel Hyper-V de l’hôte physique vers l’emplacement d’application de la stratégie |

### <a name="ipam"></a>IPAM

Les adresses IP sont attribuées et assignées différemment pour chaque pilote réseau. Windows utilise le service HNS (Host Networking Service) pour fournir une IPAM au pilote nat et fonctionne avec le mode Docker Swarm (KVS interne) pour fournir une IPAM de superposition. Tous les autres pilotes réseau utilisent une IPAM externe.

| Mode de mise en réseau/pilote | IPAM |
| -------------------------|:----:|
| NAT | Attribution et attribution d’adresses IP dynamiques par le service de mise en réseau hôte (HNS) à partir du préfixe de sous-réseau NAT interne |
| Mode transparent | Allocation d’adresses IP statiques ou dynamiques (à l’aide du serveur DHCP externe) à partir d’adresses IP dans le préfixe réseau de l’hôte de conteneur |
| Overlay | Allocation d’adresses IP dynamiques à partir du moteur de l’amarrage les préfixes et l’affectation managés par le biais de HNS |
| L2Bridge | Attribution et attribution d’adresses IP dynamiques par le service de mise en réseau hôte (HNS) à partir du préfixe de sous-réseau fourni |
| L2Tunnel | Azure uniquement-allocation d’adresse IP dynamique et attribution à partir du plug-in |

### <a name="service-discovery"></a>Découverte de service

La découverte des services est uniquement prise en charge pour certains pilotes réseau Windows.

| Nom du pilote | Découverte des services locale  | Découverte des services globale |
| :--- | :---------------     |  :---                |
| nat | YES | Oui avec docker EE |
| superposition | YES | Oui avec docker EE ou Kube-DNS |
| transparent | Non | Non |
| l2bridge | Oui avec Kube-DNS | Oui avec Kube-DNS |

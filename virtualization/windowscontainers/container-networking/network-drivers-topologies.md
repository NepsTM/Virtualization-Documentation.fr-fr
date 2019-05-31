---
title: Gestion des conteneurs Windows
description: Pilotes et topologies réseau pour les conteneurs Windows.
keywords: docker, conteneurs
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: f044cf6f9d0457dd4cc9b444dcbeebc97f22f17b
ms.sourcegitcommit: bea2c90f31a38fc7fda356619f0dd812f79d008f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/31/2019
ms.locfileid: "9685286"
---
# <a name="windows-container-network-drivers"></a>Pilotes réseau des conteneurs Windows  

En plus de tirer parti du réseau «nat» par défaut créé par Docker sur Windows, les utilisateurs peuvent définir des réseaux de conteneurs personnalisés. Les réseaux définis par l’utilisateur peuvent être créés à l’aide [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) de la commande ILC de l’ancrage. Sur Windows, les types de pilote réseau suivants sont disponibles:

- **nat**: les conteneurs reliés à un réseau créé avec le pilote «nat» sont connectés à un commutateur Hyper-V *interne* reçoivent une adresseIP à partir du préfixeIP (``--subnet``) spécifié par l’utilisateur. Le réacheminement/mappage de ports à partir de l’hôte de conteneur vers des points de terminaison de conteneur est pris en charge.
  
  >[!NOTE]
  > Les réseaux NAT créés sur Windows Server 2019 (ou versions ultérieures) ne persistent plus après le redémarrage.

  > Plusieurs réseaux NAT sont pris en charge si Windows 10 Creators Update est installé (ou une version ultérieure).
  
- **transparent**: les conteneurs reliés à un réseau créé avec le pilote «transparent» sont connectés directement au réseau physique via un commutateur Hyper-V *externe*. Les adressesIP issues du réseau physique peuvent être attribuées de façon statique (nécessite l’option ``--subnet`` spécifiée par l’utilisateur) ou dynamique à l’aide d’un serveur DHCP externe.
  
  >[!NOTE]
  >En raison de la configuration requise suivante, la connexion de vos hôtes de conteneur via un réseau transparent n’est pas prise en charge sur les ordinateurs virtuels Azure.
  
  > Nécessite: lorsque ce mode est utilisé dans un scénario de virtualisation (un hôte de conteneur est une VM) l' _usurpation d’adresse Mac est requise_.

- **superposition** - si le moteur Docker s’exécute en [mode Swarm](../manage-containers/swarm-mode.md), les conteneurs reliés à un réseau de superposition peuvent communiquer avec d’autres conteneurs attachés au même réseau entre plusieurs hôtes de conteneur. Chaque réseau de superposition créé dans un cluster Swarm possède son propre sous-réseau IP, défini par un préfixe IP privé. Le pilote réseau de superposition utilise l’encapsulation VXLAN. **Il peut être utilisé avec Kubernetes lors de l’utilisation des plans de contrôle de réseau approprié (Flannel ou OVN).**
  > Nécessite: Assurez-vous que votre environnement remplit les [conditions préalables](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) requises pour la création de réseaux superposés.

  > Nécessite: nécessite Windows Server 2016 avec [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217), Windows 10 Creators Update ou une version ultérieure.

  >[!NOTE]
  >Sur Windows Server 2019 exécutant Dockr EE 18,03 et versions ultérieures, superposer des réseaux créés par le biais des règles NAT VFP pour la connectivité sortante. Cela signifie que le conteneur fourni reçoit 1 adresse IP. Cela signifie également que les outils ICMP tels que `ping` ou `Test-NetConnection` doivent être configurés à l’aide de leurs options TCP/UDP dans les situations de débogage.

- **l2bridge** - des conteneurs reliés à un réseau créé avec le pilote «l2bridge» se trouvent dans le même sous-réseau IP que l’hôte de conteneur et connectés au réseau physique via un commutateur Hyper-V *externe*. Les adresses IP doivent être attribuées de façon statique à partir du même préfixe que l’hôte de conteneur. Tous les points de terminaison de conteneur sur l’ordinateur hôte ont la même adresse MAC que celles de l’hôte en raison de l’opération de traduction d’adresses de couche2 (réécriture d’adresses MAC) en entrée et en sortie.
  > Nécessite: nécessite Windows Server 2016, Windows 10 Creators Update ou une version ultérieure.

  > Nécessite: [stratégie OutboundNAT](./advanced.md#specify-outboundnat-policy-for-a-network) pour la connectivité externe.

- **l2tunne**l: Semblable à l2bridge. Cependant _ce pilote doit uniquement être utilisé dans une pile Microsoft Cloud Stack_. Les paquets provenant d’un conteneur sont envoyés à l’hôte de virtualisation où la stratégie SDN est appliquée.


## <a name="network-topologies-and-ipam"></a>Topologies réseau et IPAM

Le tableau ci-dessous montre de quelle manière est fournie la connectivité réseau pour les connexions internes (conteneur-conteneur) et externes dans chaque pilote réseau.

### <a name="networking-modesdocker-drivers"></a>Modes réseau/pilotes de l’amarrage

  | Pilote de réseau Windows Docker | OS typiques | Conteneur à conteneur (nœud unique) | Conteneur vers externe (nœud unique + plusieurs nœuds) | Conteneur à conteneur (plusieurs nœuds) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (par défaut)** | Bon pour les développeurs | <ul><li>Même sous-réseau: Connexion reliée via un commutateur virtuel Hyper-V</li><li> Sous-réseau croisé: non pris en charge (un seul préfixe NAT)</li></ul> | Acheminé via la carte réseau virtuelle de gestion (lié à WinNAT) | Pas directement pris en charge: nécessite l’exposition de ports via un ordinateur hôte |
  | **Transparent** | Bon pour les développeurs ou les petits déploiements | <ul><li>Même sous-réseau: Connexion reliée via un commutateur virtuel Hyper-V</li><li>Entre sous-réseaux: Acheminé via l’hôte de conteneur</li></ul> | Acheminé via l’hôte de conteneur avec un accès direct à l’adaptateur réseau (physique) | Acheminé via l’hôte de conteneur avec un accès direct à l’adaptateur réseau (physique) |
  | **Superposition** | Bonne pour les multiples nœuds; requis pour l’essaimeur d’amarrage, disponible dans Kubernetes | <ul><li>Même sous-réseau: Connexion reliée via un commutateur virtuel Hyper-V</li><li>Entre sous-réseaux: Le trafic réseau est encapsulé et acheminé via la carte réseau virtuelle de gestion</li></ul> | Pas pris en charge directement - nécessite un deuxième point de terminaison de conteneur attaché au réseau NAT | Même/entre sous-réseaux: Le trafic réseau est encapsulé avec VXLAN et acheminé via la carte réseau virtuelle de gestion |
  | **L2Bridge** | Utilisé pour Kubernetes et Microsoft SDN | <ul><li>Même sous-réseau: Connexion reliée via un commutateur virtuel Hyper-V</li><li> Entre sous-réseaux: Adresse MAC de conteneur réécrite sur les éléments entrants et sortants et acheminés</li></ul> | Adresse MAC de conteneur réécrite sur les éléments entrants et sortants et acheminés | <ul><li>Même sous-réseau: Connexion reliée par un pont</li><li>Sous-réseau: routés via la gestion des vNIC sur WSv1709 et les versions ultérieures</li></ul> |
  | **L2Tunnel**| Azure uniquement | Entre/même sous-réseau: Commutateur virtuel Hyper-V de l’hôte physique épinglé à l’endroit où la stratégie est appliquée | Le trafic doit passer par la passerelle de réseau virtuel Azure | Entre/même sous-réseau: Commutateur virtuel Hyper-V de l’hôte physique épinglé à l’endroit où la stratégie est appliquée |

### <a name="ipam"></a>IPAM

Les adresses IP sont attribuées et assignées différemment pour chaque pilote réseau. Windows utilise le service HNS (Host Networking Service) pour fournir une IPAM au pilote nat et fonctionne avec le mode Docker Swarm (KVS interne) pour fournir une IPAM de superposition. Tous les autres pilotes réseau utilisent une IPAM externe.

| Mode de mise en réseau / Pilote | IPAM |
| -------------------------|:----:|
| NAT | Attribution et affectation dynamiques d’adresses IP par le service de mise en réseau d’hébergement (HNS) à partir du préfixe de sous-réseau NAT interne |
| Transparent | Allocation IP et affectation d’IP statiques ou dynamiques (à l’aide du serveur DHCP externe) à partir d’adressesIP au sein du préfixe réseau de l’hôte de conteneur |
| Superposition | Allocation IP dynamique à partir de préfixes gérés et d’affectations en mode Swarm du moteur Docker via HNS |
| L2Bridge | Attribution d’adresse IP statique et affectation à partir d’adresses IP dans le préfixe de réseau de l’hôte du conteneur (peut également être attribué via HNS) |
| L2Tunnel | Azure uniquement - Allocation IP et affectation dynamiques à partir d’un plug-in |

### <a name="service-discovery"></a>Découverte des services

La découverte des services est uniquement prise en charge pour certains pilotes réseau Windows.

|  | Découverte des services locale  | Découverte des services globale |
| :---: | :---------------     |  :---                |
| nat | OUI | OUI avec Docker EE |  
| superposition | OUI | Oui avec docker EE ou Kube-DNS |
| transparent | NO | NON |
| l2bridge | NON | Oui avec Kube-DNS |

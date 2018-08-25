---
title: Mise en réseau de conteneur Windows
description: Pilotes et topologies réseau pour les conteneurs Windows.
keywords: docker, conteneurs
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: bb3681a83991b3d4e24348b686146616d4a88c4f
ms.sourcegitcommit: db508decd9bf6c0dce9952e1a86bf80f00d025eb
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/20/2018
ms.locfileid: "2315662"
---
# <a name="windows-container-network-drivers"></a>Pilotes réseau de conteneurs Windows  

En plus de tirer parti du réseau «nat» par défaut créé par Docker sur Windows, les utilisateurs peuvent définir des réseaux de conteneurs personnalisés. Il est possible de créer des réseaux définis par l’utilisateur au moyen de l’interface de ligne de commande Docker, en utilisant la commande [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/). Sur Windows, les types de pilote réseau suivants sont disponibles:

- **nat**: les conteneurs reliés à un réseau créé avec le pilote «nat» sont connectés à un commutateur Hyper-V *interne* reçoivent une adresseIP à partir du préfixeIP (``--subnet``) spécifié par l’utilisateur. Le réacheminement/mappage de ports à partir de l’hôte de conteneur vers des points de terminaison de conteneur est pris en charge.
  > Plusieurs réseaux NAT sont désormais pris en charge si Windows10 Creators Update est installé!

- **transparent**: les conteneurs reliés à un réseau créé avec le pilote «transparent» sont connectés directement au réseau physique via un commutateur Hyper-V *externe*. Les adressesIP issues du réseau physique peuvent être attribuées de façon statique (nécessite l’option ``--subnet`` spécifiée par l’utilisateur) ou dynamique à l’aide d’un serveur DHCP externe. 
  > Remarque: pour les raisons du au-dessous de la spécification, connecter vos hôtes conteneur via un réseau transparent n'est pas pris en charge sur les machines virtuelles Azure.
  
  > Nécessite: Lorsque ce mode est utilisé dans un scénario de la virtualisation (hôte de conteneur est une machine virtuelle) _usurpation d’adresse MAC est requis_.

- **superposition** - si le moteur Docker s’exécute en [mode Swarm](../manage-containers/swarm-mode.md), les conteneurs reliés à un réseau de superposition peuvent communiquer avec d’autres conteneurs attachés au même réseau entre plusieurs hôtes de conteneur. Chaque réseau de superposition créé dans un cluster Swarm possède son propre sous-réseau IP, défini par un préfixe IP privé. Le pilote réseau de superposition utilise l’encapsulation VXLAN. **Il peut être utilisé avec Kubernetes lors de l’utilisation des plans de contrôle de réseau approprié (Flannel ou OVN).**
  > Nécessite: Assurez-vous que votre environnement répond aux ces [conditions préalables](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) de *requis* pour créer des réseaux de superposition.

  > Nécessite: Requiert Windows Server 2016 avec [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217), mise à jour des créateurs de 10 Windows ou une version ultérieure.

- **l2bridge** - des conteneurs reliés à un réseau créé avec le pilote «l2bridge» se trouvent dans le même sous-réseau IP que l’hôte de conteneur et connectés au réseau physique via un commutateur Hyper-V *externe*. Les adresses IP doivent être attribuées de façon statique à partir du même préfixe que l’hôte de conteneur. Tous les points de terminaison de conteneur sur l’ordinateur hôte ont la même adresse MAC que celles de l’hôte en raison de l’opération de traduction d’adresses de couche2 (réécriture d’adresses MAC) en entrée et en sortie.
  > Nécessite: Lorsque ce mode est utilisé dans un scénario de la virtualisation (hôte de conteneur est une machine virtuelle) _usurpation d’adresse MAC est requis_.
  
  > Nécessite: Requiert Windows Server 2016, mise à jour des créateurs de 10 Windows ou une version ultérieure.

- **l2tunne**l: Semblable à l2bridge. Cependant _ce pilote doit uniquement être utilisé dans une pile Microsoft Cloud Stack_. Les paquets provenant d’un conteneur sont envoyés à l’hôte de virtualisation où la stratégie SDN est appliquée.

> Pour savoir comment connecter des points de terminaison de conteneur à un réseau virtuel de client existant avec la pile Microsoft SDN, consultez la rubrique relative à l’[attachement de conteneurs à un réseau virtuel](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network).


## <a name="network-topologies-and-ipam"></a>Topologies de réseau et IPAM
Le tableau ci-dessous montre de quelle manière est fournie la connectivité réseau pour les connexions internes (conteneur-conteneur) et externes dans chaque pilote réseau.

### <a name="networking-modes--docker-drivers"></a>Modes de mise en réseau / pilotes Docker

  | Pilote de réseau Windows Docker | Utilisations classiques | Conteneur-conteneur (nœud unique) | Conteneur-externe (nœud unique + nœuds multiples) | Conteneur-conteneur (nœuds multiples) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (par défaut)** | Bon pour les développeurs | <ul><li>Même sous-réseau: Connexion reliée via un commutateur virtuel Hyper-V</li><li> Entre-sous-réseaux: Non pris en charge dans WS2016 (un seul préfixe interne NAT)</li></ul> | Acheminé via la carte réseau virtuelle de gestion (lié à WinNAT) | Pas directement pris en charge: nécessite l’exposition de ports via un ordinateur hôte |
  | **Transparent** | Bon pour les développeurs ou les petits déploiements | <ul><li>Même sous-réseau: Connexion reliée via un commutateur virtuel Hyper-V</li><li>Entre sous-réseaux: Acheminé via l’hôte de conteneur</li></ul> | Acheminé via l’hôte de conteneur avec un accès direct à l’adaptateur réseau (physique) | Acheminé via l’hôte de conteneur avec un accès direct à l’adaptateur réseau (physique) |
  | **Superposition** | Requis pour Docker Swarm, nœuds multiples | <ul><li>Même sous-réseau: Connexion reliée via un commutateur virtuel Hyper-V</li><li>Entre sous-réseaux: Le trafic réseau est encapsulé et acheminé via la carte réseau virtuelle de gestion</li></ul> | Pas pris en charge directement - nécessite un deuxième point de terminaison de conteneur attaché au réseau NAT | Même/entre sous-réseaux: Le trafic réseau est encapsulé avec VXLAN et acheminé via la carte réseau virtuelle de gestion |
  | **L2Bridge** | Utilisé pour Kubernetes et Microsoft SDN | <ul><li>Même sous-réseau: Connexion reliée via un commutateur virtuel Hyper-V</li><li> Entre sous-réseaux: Adresse MAC de conteneur réécrite sur les éléments entrants et sortants et acheminés</li></ul> | Adresse MAC de conteneur réécrite sur les éléments entrants et sortants et acheminés | <ul><li>Même sous-réseau: Connexion reliée par un pont</li><li>Entre sous-réseaux: Non pris en charge dans WS2016.</li></ul> |
  | **L2Tunnel**| Azure uniquement | Entre/même sous-réseau: Commutateur virtuel Hyper-V de l’hôte physique épinglé à l’endroit où la stratégie est appliquée | Le trafic doit passer par la passerelle de réseau virtuel Azure | Entre/même sous-réseau: Commutateur virtuel Hyper-V de l’hôte physique épinglé à l’endroit où la stratégie est appliquée |

### <a name="ipam"></a>IPAM 
Les adresses IP sont attribuées et assignées différemment pour chaque pilote réseau. Windows utilise le service HNS (Host Networking Service) pour fournir une IPAM au pilote nat et fonctionne avec le mode Docker Swarm (KVS interne) pour fournir une IPAM de superposition. Tous les autres pilotes réseau utilisent une IPAM externe.

| Mode de mise en réseau / Pilote | IPAM |
| -------------------------|:----:|
| NAT | AllocationIP et affectation dynamiques par le service de mise en réseau d’hôte (HNS) à partir du préfixe de sous-réseau NAT interne |
| Transparent | Allocation IP et affectation d’IP statiques ou dynamiques (à l’aide du serveur DHCP externe) à partir d’adressesIP au sein du préfixe réseau de l’hôte de conteneur |
| Superposition | Allocation IP dynamique à partir de préfixes gérés et d’affectations en mode Swarm du moteur Docker via HNS |
| L2Bridge | Allocation IP et affectation d’IP statiques à partir d’adressesIP au sein du préfixe réseau de l’hôte de conteneur (peut également être affecté via un plug-in HNS) |
| L2Tunnel | Azure uniquement - Allocation IP et affectation dynamiques à partir d’un plug-in |

### <a name="service-discovery"></a>Découverte des services
La découverte des services est uniquement prise en charge pour certains pilotes réseau Windows.

|  | Découverte des services locale  | Découverte des services globale |
| :---: | :---------------     |  :---                |
| nat | OUI | N/A |  
| superposition | OUI | OUI avec Docker EE |
| transparent | NON | NON |
| l2bridge | NON | OUI avec Kube-DNS |

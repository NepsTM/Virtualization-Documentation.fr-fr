---
title: Mise en réseau de conteneurs Windows
description: Pilotes et topologies réseau pour les conteneurs Windows.
keywords: docker, conteneurs
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: f54c715f474c50c4b3073912adc4e0ab1c42d662
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853923"
---
# <a name="windows-container-network-drivers"></a>Pilotes réseau de conteneur Windows  

En plus de tirer parti du réseau « nat » par défaut créé par Docker sur Windows, les utilisateurs peuvent définir des réseaux de conteneurs personnalisés. Les réseaux définis par l’utilisateur peuvent être créés à l’aide de la commande [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) de l’interface de commande de l’ancrage. Sur Windows, les types de pilote réseau suivants sont disponibles :

- **nat** : les conteneurs reliés à un réseau créé avec le pilote « nat » sont connectés à un commutateur Hyper-V *interne* reçoivent une adresse IP à partir du préfixe IP (``--subnet``) spécifié par l’utilisateur. Le réacheminement/mappage de ports à partir de l’hôte de conteneur vers des points de terminaison de conteneur est pris en charge.
  
  >[!NOTE]
  > Les réseaux NAT créés sur Windows Server 2019 (ou version ultérieure) ne sont plus conservés après le redémarrage.

  > Plusieurs réseaux NAT sont pris en charge si vous avez installé Windows 10 Creators Update (ou version ultérieure).
  
- **transparent** : les conteneurs reliés à un réseau créé avec le pilote « transparent » sont connectés directement au réseau physique via un commutateur Hyper-V *externe*. Les adresses IP issues du réseau physique peuvent être attribuées de façon statique (nécessite l'option ``--subnet`` spécifiée par l'utilisateur) ou dynamique à l’aide d’un serveur DHCP externe.
  
  >[!NOTE]
  >En raison des exigences suivantes, la connexion de vos hôtes de conteneur sur un réseau transparent n’est pas prise en charge sur les machines virtuelles Azure.
  
  > Requiert : quand ce mode est utilisé dans un scénario de virtualisation (un hôte de conteneur est une machine virtuelle), une _usurpation d’adresse Mac est nécessaire_.

- **superposition** - si le moteur Docker s’exécute en [mode Swarm](../manage-containers/swarm-mode.md), les conteneurs reliés à un réseau de superposition peuvent communiquer avec d’autres conteneurs attachés au même réseau entre plusieurs hôtes de conteneur. Chaque réseau de superposition créé dans un cluster Swarm possède son propre sous-réseau IP, défini par un préfixe IP privé. Le pilote réseau de superposition utilise l’encapsulation VXLAN. **Peut être utilisé avec Kubernetes lors de l’utilisation de plans de contrôle de réseau appropriés (par exemple, Flannel).**
  > Nécessite : Assurez-vous que votre environnement satisfait aux [conditions préalables](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) requises pour créer des réseaux de superposition.

  > Requiert : sur Windows Server 2019, cela nécessite [KB4489899](https://support.microsoft.com/help/4489899).

  > Requiert : sur Windows Server 2016, cela nécessite [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217).

  >[!NOTE]
  >Sur Windows Server 2019, les réseaux de superposition créés par l’Assistant de connexion par essaim tirent parti des règles NAT VFP pour la connectivité sortante. Cela signifie qu’un conteneur donné reçoit 1 adresse IP. Cela signifie également que les outils basés sur ICMP tels que `ping` ou `Test-NetConnection` doivent être configurés à l’aide de leurs options TCP/UDP dans des situations de débogage.

- **l2bridge** : similaire au mode de mise en réseau `transparent`, les conteneurs attachés à un réseau créé avec le pilote « l2bridge » sont connectés au réseau physique via un commutateur Hyper-V *externe* . La différence dans l2bridge réside dans le fait que les points de terminaison de conteneur auront la même adresse MAC que l’hôte en raison de l’opération de traduction d’adresse de couche 2 (réécriture MAC) sur l’entrée et la sortie. Dans les scénarios de clustering, cela permet d’atténuer la contrainte sur les commutateurs qui ont besoin d’apprendre des adresses MAC de conteneurs parfois à courte durée de vie. Les réseaux L2bridge peuvent être configurés de deux manières différentes :
  1. Le réseau L2bridge est configuré avec le même sous-réseau IP que l’hôte de conteneur
  2. Le réseau L2bridge est configuré avec un nouveau sous-réseau IP personnalisé
  
  Dans la configuration 2, les utilisateurs doivent ajouter un point de terminaison sur le compartiment réseau hôte qui agit en tant que passerelle et configurer des fonctionnalités de routage pour le préfixe désigné. 
  > Requiert : requiert Windows Server 2016, Windows 10 Creators Update ou une version ultérieure.


- **l2bridge** : similaire au mode de mise en réseau `transparent`, les conteneurs attachés à un réseau créé avec le pilote « l2bridge » sont connectés au réseau physique via un commutateur Hyper-V *externe* . La différence dans l2bridge réside dans le fait que les points de terminaison de conteneur auront la même adresse MAC que l’hôte en raison de l’opération de traduction d’adresse de couche 2 (réécriture MAC) sur l’entrée et la sortie. Dans les scénarios de clustering, cela permet d’atténuer la contrainte sur les commutateurs qui ont besoin d’apprendre des adresses MAC de conteneurs parfois à courte durée de vie. Les réseaux L2bridge peuvent être configurés de deux manières différentes :
  1. Le réseau L2bridge est configuré avec le même sous-réseau IP que l’hôte de conteneur
  2. Le réseau L2bridge est configuré avec un nouveau sous-réseau IP personnalisé
  
  Dans la configuration 2, les utilisateurs doivent ajouter un point de terminaison sur le compartiment réseau hôte qui agit en tant que passerelle et configurer des fonctionnalités de routage pour le préfixe désigné. 
  >[!TIP]
  >Vous trouverez plus d’informations sur la configuration et l’installation de l2bridge [ici](https://techcommunity.microsoft.com/t5/networking-blog/l2bridge-container-networking/ba-p/1180923).

- **l2tunnel** : similaire à l2bridge, mais _ce pilote ne doit être utilisé que dans une pile de Microsoft Cloud (Azure)_ . Les paquets provenant d’un conteneur sont envoyés à l’hôte de virtualisation où la stratégie SDN est appliquée.


## <a name="network-topologies-and-ipam"></a>Topologies de réseau et IPAM

Le tableau ci-dessous montre de quelle manière est fournie la connectivité réseau pour les connexions internes (conteneur-conteneur) et externes dans chaque pilote réseau.

### <a name="networking-modesdocker-drivers"></a>Modes de mise en réseau/pilotes de l’ancrage

  | Pilote de réseau Windows Docker | Utilisations courantes | Conteneur à conteneur (nœud unique) | Conteneur vers externe (nœud unique + plusieurs nœuds) | Conteneur à conteneur (nœuds multiples) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (par défaut)** | Bon pour les développeurs | <ul><li>Même sous-réseau : Connexion reliée via un commutateur virtuel Hyper-V</li><li> Sous-réseau croisé : non pris en charge (un seul préfixe NAT interne)</li></ul> | Acheminé via la carte réseau virtuelle de gestion (lié à WinNAT) | Pas directement pris en charge : nécessite l’exposition de ports via un ordinateur hôte |
  | **Transparente** | Bon pour les développeurs ou les petits déploiements | <ul><li>Même sous-réseau : Connexion reliée via un commutateur virtuel Hyper-V</li><li>Entre sous-réseaux : Acheminé via l’hôte de conteneur</li></ul> | Acheminé via l’hôte de conteneur avec un accès direct à l’adaptateur réseau (physique) | Acheminé via l’hôte de conteneur avec un accès direct à l’adaptateur réseau (physique) |
  | **Overlay** | Adapté à plusieurs nœuds ; requis pour le essaimeur essaim, disponible dans Kubernetes | <ul><li>Même sous-réseau : Connexion reliée via un commutateur virtuel Hyper-V</li><li>Entre sous-réseaux : Le trafic réseau est encapsulé et acheminé via la carte réseau virtuelle de gestion</li></ul> | Non directement pris en charge : nécessite un deuxième point de terminaison de conteneur attaché au réseau NAT sur Windows Server 2016 ou une règle NAT VFP sur Windows Server 2019.  | Même/entre sous-réseaux : Le trafic réseau est encapsulé avec VXLAN et acheminé via la carte réseau virtuelle de gestion |
  | **L2Bridge** | Utilisé pour Kubernetes et Microsoft SDN | <ul><li>Même sous-réseau : Connexion reliée via un commutateur virtuel Hyper-V</li><li> Entre sous-réseaux : Adresse MAC de conteneur réécrite sur les éléments entrants et sortants et acheminés</li></ul> | Adresse MAC de conteneur réécrite sur les éléments entrants et sortants et acheminés | <ul><li>Même sous-réseau : Connexion reliée par un pont</li><li>Inter-réseaux : routé via la gestion carte réseau virtuelle sur WSv1809 et versions ultérieures</li></ul> |
  | **L2Tunnel**| Azure uniquement | Entre/même sous-réseau : Commutateur virtuel Hyper-V de l’hôte physique épinglé à l’endroit où la stratégie est appliquée | Le trafic doit passer par la passerelle de réseau virtuel Azure | Entre/même sous-réseau : Commutateur virtuel Hyper-V de l’hôte physique épinglé à l’endroit où la stratégie est appliquée |

### <a name="ipam"></a>IPAM

Les adresses IP sont attribuées et assignées différemment pour chaque pilote réseau. Windows utilise le service HNS (Host Networking Service) pour fournir une IPAM au pilote nat et fonctionne avec le mode Docker Swarm (KVS interne) pour fournir une IPAM de superposition. Tous les autres pilotes réseau utilisent une IPAM externe.

| Mode de mise en réseau / Pilote | IPAM |
| -------------------------|:----:|
| NAT | Attribution et attribution d’adresses IP dynamiques par le service de mise en réseau hôte (HNS) à partir du préfixe de sous-réseau NAT interne |
| Transparent | Allocation IP et affectation d’IP statiques ou dynamiques (à l’aide du serveur DHCP externe) à partir d’adresses IP au sein du préfixe réseau de l’hôte de conteneur |
| Overlay | Allocation IP dynamique à partir de préfixes gérés et d’affectations en mode Swarm du moteur Docker via HNS |
| L2Bridge | Allocation et attribution d’adresses IP statiques à partir d’adresses IP au préfixe réseau de l’hôte de conteneur (peut également être affectée via HNS) |
| L2Tunnel | Azure uniquement - Allocation IP et affectation dynamiques à partir d’un plug-in |

### <a name="service-discovery"></a>Détection du service

La découverte des services est uniquement prise en charge pour certains pilotes réseau Windows.

|  | Découverte des services locale  | Découverte des services globale |
| :---: | :---------------     |  :---                |
| nat | OUI | OUI avec Docker EE |  
| superposition | OUI | Oui avec docker EE ou Kube-DNS |
| transparent | NON | NON |
| l2bridge | NON | Oui avec Kube-DNS |

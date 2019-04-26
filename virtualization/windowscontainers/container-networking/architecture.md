---
title: Mise en réseau de conteneur Windows
description: Présentation de l’architecture des réseaux de conteneur Windows.
keywords: docker, conteneurs
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: ac0088995dfbda73351d39a494435c431e0939e7
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576330"
---
# <a name="windows-container-networking"></a>Mise en réseau de conteneur Windows

>[!IMPORTANT]
>Veuillez vous reporter à [Mise en réseau de conteneur Docker](https://docs.docker.com/engine/userguide/networking/) pour la mise en réseau de commandes, les options et syntax.* ** à l’exception des cas décrits dans [non pris en charge des fonctionnalités et options réseau](#unsupported-features-and-network-options), Docker toutes les commandes de mise en réseau de docker en général prise en charge sur Windows avec la même syntaxe que sur Linux. Toutefois, les piles réseau Windows et Linux sont différentes, et par conséquent, vous constaterez que certaines commandes réseau de Linux (par exemple, ifconfig) ne sont pas pris en charge sur Windows.

## <a name="basic-networking-architecture"></a>Architecture de mise en réseau de base

Cette rubrique donne un aperçu de la façon dont Docker crée et gère des réseaux hôtes sous Windows. Sur le plan de la mise en réseau, les conteneurs Windows fonctionnent de la même façon que des machines virtuelles. Chaque conteneur possède une carte réseau virtuelle (vNIC) connectée à un commutateur virtuel Hyper-V (vSwitch). Windows prend en charge cinq [modes ou pilotes réseau](./network-drivers-topologies.md) différents qui peuvent être créés au moyen de Docker: *nat*, *superposition*, *transparent*, *l2bridge* et *l2tunnel*. Choisissez le pilote réseau le mieux adapté à l’infrastructure de votre réseau physique et à la configuration requise du réseau (hôte unique ou hôtes multiples).

![texte](media/windowsnetworkstack-simple.png)

Lors de sa première exécution, le moteur Docker crée un réseau NAT par défaut, «nat», qui utilise un vSwitch interne et un composant Windows nommé `WinNAT`. Si des vSwitch externes, créés au moyen de PowerShell ou du Gestionnaire Hyper-V, sont présents sur l’ordinateur hôte, ils sont également disponibles pour Docker à l’aide du pilote réseau *transparent* et sont visibles lorsque vous exécutez la commande ``docker network ls``.  

![texte](media/docker-network-ls.png)

- Un vSwitch **interne** est une application qui n’est pas connecté directement à une carte réseau sur l’hôte de conteneur.
- Un commutateur virtuel **externe** est une application qui est connecté directement à une carte réseau sur l’hôte de conteneur.

![texte](media/get-vmswitch.png)

Le réseau «nat» est le réseau par défaut des conteneurs qui s’exécutent sur Windows. Tous les conteneurs qui s'exécutent sur Windows sans indicateurs ou arguments pour implémenter des configurations de réseau spécifiques sont attachés au réseau «nat» par défaut, et une adresse IP leur est affectée à partir de la plage IP de préfixe interne du réseau «nat». Le préfixe IP interne par défaut utilisé pour «nat» est 172.16.0.0/16. 

## <a name="container-network-management-with-host-network-service"></a>Gestion de réseau de conteneur avec le service HNS

Le service HNS et le service de calcul hôte (HCS) fonctionnent ensemble pour créer des conteneurs et attacher des points de terminaison à un réseau.

### <a name="network-creation"></a>Création de réseau

- HNS crée un commutateur virtuel Hyper-V pour chaque réseau.
- HNS crée des pools NAT et IP en fonction des besoins

### <a name="endpoint-creation"></a>Création de point de terminaison

- HNS crée un espace de noms réseau par point de terminaison de conteneur
- Lieux HNS/HCS v(m)NIC dans l’espace de noms de réseau
- HNS crée des ports (vSwitch)
- HNS attribue une adresse IP, des informations DNS, des itinéraires, etc. (selon le mode de mise en réseau) vers le point de terminaison

### <a name="policy-creation"></a>Création de stratégies

- Réseau NAT par défaut: HNS crée des règles de réacheminement de port WinNAT / mappages avec les règles d’autorisation de Pare-feu Windows correspondantes
- Tous les autres réseaux: HNS utilise la plateforme de filtrage virtuel (VFP, Virtual Filtering Platform) pour la création de stratégie
    - Cela inclut: l’équilibrage de charge, les ACL, l’encapsulation, etc.
    - Recherchez nos API et schéma HNS **bientôt publiés **.

![texte](media/HNS-Management-Stack.png)

## <a name="unsupported-features-and-network-options"></a>Fonctionnalités et options réseau non prises en charge

Les options de mise en réseau suivantes ne sont actuellement **pas** pris en charge sur Windows:

- Les conteneurs Windows connectés à des réseaux de superposition, NAT et l2bridge ne gèrent pas la communication par la pile IPv6.
- Chiffrements conteneur via IPsec.
- Prise en charge des proxy HTTP pour les conteneurs.
- Attacher des points de terminaison à l’exécution dans l’isolation Hyper-V (ajout à chaud).
- Mise en réseau sur une infrastructure virtualisée Azure via le pilote de réseau transparent.

| Commande        | Option non prise en charge   |
|---------------|:--------------------:|
| ``docker run``|   ``--ip6``, ``--dns-option`` |
| ``docker network create``| ``--aux-address``, ``--internal``, ``--ip-range``, ``--ipam-driver``, ``--ipam-opt``, ``--ipv6``, ``--opt encrypted`` |
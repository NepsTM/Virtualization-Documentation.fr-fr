---
title: Mise en réseau de conteneur Windows
description: Isolation et sécurité réseau dans les conteneurs Windows.
keywords: docker, conteneurs
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: d5081104f1614a91d6441a5e879a439f1df1bf77
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439286"
---
# <a name="network-isolation-and-security"></a>Isolement et sécurité réseau

## <a name="isolation-with-network-namespaces"></a>Isolation avec les espaces de noms de réseau

Chaque point de terminaison de conteneur est placé dans son propre __espace de noms de réseau__. La vNIC de l’hôte de gestion et la pile réseau hôte se trouvent dans l’espace de nom réseau par défaut. Pour appliquer l’isolement réseau entre les conteneurs sur le même hôte, un espace de noms réseau est créé pour chaque conteneur Windows Server et les conteneurs s’exécutent sous l’isolation Hyper-V dans laquelle la carte réseau du conteneur est installée. Les conteneurs Windows Server utilisent une carte réseau virtuelle hôte pour s’attacher au commutateur virtuel. L’isolation Hyper-V utilise une carte réseau de machine virtuelle synthétique (non exposée à la machine virtuelle de l’utilitaire) à attacher au commutateur virtuel.

![texte](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>Sécurité réseau

Selon le conteneur et le pilote réseau utilisé, les ACL de port sont appliquées par une combinaison du Pare-feu Windows et [VFP ](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/).

### <a name="windows-server-containers"></a>Conteneurs Windows Server

Ils utilisent le pare-feu des hôtes de Windows (compatible avec les espaces de noms réseau), ainsi que VFP

* Sortant par défaut : Autoriser tout
* Entrant par défaut : Autoriser tout le trafic réseau non sollicité (TCP, UDP, ICMP, IGMP)
  * Refuser tous les autres trafics réseau ne provenant pas de ces protocoles

  >[!NOTE]
  >Avant la mise à jour de Windows Server, version 1709 et Windows 10 automne Creators, la règle de trafic entrant par défaut était refuser tout. Les utilisateurs qui exécutent ces versions antérieures peuvent créer des règles d’autorisation entrantes avec ``docker run -p`` (réacheminement de port).

### <a name="hyper-v-isolation"></a>Isolation Hyper-V

Les conteneurs exécutés dans l’isolation Hyper-V ont leur propre noyau isolé et, par conséquent, exécutent leur propre instance du pare-feu Windows avec la configuration suivante :

* Par défaut tout autoriser dans les deux Pare-feu Windows (en cours d’exécution dans l’utilitaire VM) et VFP

![texte](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Pod Kubernetes

Dans un [Pod Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/pod/), un conteneur d’infrastructure est d’abord créé auquel un point de terminaison est attaché. Les conteneurs qui appartiennent au même Pod, y compris les conteneurs d’infrastructure et de travail, partagent un espace de noms réseau commun (même adresse IP et espace de port).

![texte](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>Personnalisation des ACL de port par défaut

Si vous souhaitez modifier les listes de contrôle d’accès des ports par défaut, consultez d’abord notre documentation sur le service de mise en réseau de l’hôte (lien à ajouter bientôt). Vous devez mettre à jour les stratégies à l’intérieur des composants suivants :

>[!NOTE]
>Pour l’isolation Hyper-V en mode transparent et NAT, vous ne pouvez pas reprogrammer actuellement les listes de contrôle d’accès des ports par défaut. Cela est matérialisé par un « X » dans le tableau.

| Pilote réseau | Conteneurs Windows Server | Isolation Hyper-V  |
| -------------- |-------------------------- | ------------------- |
| Transparent | Pare-feu Windows | X |
| NAT | Pare-feu Windows | X |
| L2Bridge | Les deux | VFP |
| L2Tunnel | Les deux | VFP |
| Superposition  | Les deux | VFP |
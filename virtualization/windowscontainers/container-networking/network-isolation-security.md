---
title: Isolement et sécurité réseau
description: Isolement réseau et sécurité dans les conteneurs Windows.
keywords: docker, conteneurs
author: jmesser81
ms.date: 03/27/2018
ms.topic: conceptual
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: e629666a36c3e742a970a60adcf13c526c710eec
ms.sourcegitcommit: bb18e6568393da748a6d511d41c3acbe38c62668
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/12/2020
ms.locfileid: "88161768"
---
# <a name="network-isolation-and-security"></a>Isolement et sécurité réseau

## <a name="isolation-with-network-namespaces"></a>Isolation avec les espaces de noms de réseau

Chaque point de terminaison de conteneur est placé dans son propre __espace de noms de réseau__. L’hôte de gestion carte réseau virtuelle et la pile réseau de l’ordinateur hôte se trouvent dans l’espace de noms réseau par défaut. Pour appliquer l’isolement réseau entre les conteneurs sur le même hôte, un espace de noms réseau est créé pour chaque conteneur Windows Server et les conteneurs s’exécutent sous l’isolation Hyper-V dans laquelle la carte réseau du conteneur est installée. Les conteneurs Windows Server utilisent une carte réseau virtuelle hôte pour s’attacher au commutateur virtuel. L’isolation Hyper-V utilise une carte réseau de machine virtuelle synthétique (non exposée à la machine virtuelle de l’utilitaire) à attacher au commutateur virtuel.

![text](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>Sécurité du réseau

Selon le conteneur et le pilote réseau utilisés, les listes de contrôle d’accès des ports sont appliquées par une combinaison du pare-feu Windows et de la [VFP](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/).

### <a name="windows-server-containers"></a>Conteneurs Windows Server

Ils utilisent le pare-feu Windows hosts (avec les espaces de noms de réseau), ainsi que la VFP

* Par défaut sortant : autoriser tout
* Par défaut entrant : autoriser tout le trafic réseau non sollicité (TCP, UDP, ICMP, IGMP)
  * REFUSER tout autre trafic réseau qui ne provient pas de ces protocoles

  >[!NOTE]
  >Avant la mise à jour de Windows Server, version 1709 et Windows 10 automne Creators, la règle de trafic entrant par défaut était refuser tout. Les utilisateurs qui exécutent ces versions antérieures peuvent créer des règles d’autorisation entrantes avec ``docker run -p`` (réacheminement de port).

### <a name="hyper-v-isolation"></a>Isolation Hyper-V

Les conteneurs exécutés dans l’isolation Hyper-V ont leur propre noyau isolé et, par conséquent, exécutent leur propre instance du pare-feu Windows avec la configuration suivante :

* AUTORISATION par défaut tous dans le pare-feu Windows (en cours d’exécution dans la machine virtuelle utilitaire) et VFP

![text](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Pods Kubernetes

Dans un [Pod Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/pod/), un conteneur d’infrastructure est d’abord créé auquel un point de terminaison est attaché. Les conteneurs qui appartiennent au même Pod, y compris les conteneurs d’infrastructure et de travail, partagent un espace de noms réseau commun (même adresse IP et espace de port).

![text](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>Personnalisation des ACL de port par défaut

Si vous souhaitez modifier les listes de contrôle d’accès des ports par défaut, consultez d’abord notre documentation sur le service de mise en réseau de l’hôte (lien à ajouter bientôt). Vous devez mettre à jour les stratégies à l’intérieur des composants suivants :

>[!NOTE]
>Pour l’isolation Hyper-V en mode transparent et NAT, vous ne pouvez pas reprogrammer actuellement les listes de contrôle d’accès des ports par défaut. Cela est reflété par un « X » dans la table.

| Pilote réseau | Conteneurs Windows Server | Isolation Hyper-V  |
| -------------- |-------------------------- | ------------------- |
| Mode transparent | Pare-feu Windows | X |
| NAT | Pare-feu Windows | X |
| L2Bridge | Les deux | VFP |
| L2Tunnel | Les deux | VFP |
| Overlay  | Les deux | VFP |
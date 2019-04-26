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
ms.openlocfilehash: 1c0a3fd25a5572604db59e0c68d8b4a3d84b00e9
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576310"
---
# <a name="network-isolation-and-security"></a>L’isolement réseau et sécurité

## <a name="isolation-with-network-namespaces"></a>Isolation des espaces de noms réseau

Chaque point de terminaison de conteneur est placé dans son propre __espace de noms de réseau__. La vNIC de l’hôte de gestion et la pile réseau hôte se trouvent dans l’espace de nom réseau par défaut. Pour appliquer une isolation réseau entre les conteneurs sur le même hôte, un espace de noms réseau est créé pour chaque conteneur Windows Server et les conteneurs exécutés sous isolation Hyper-V dans lequel est installée la carte réseau pour le conteneur. Les conteneurs Windows Server utilisent une carte réseau virtuelle hôte pour s’attacher au commutateur virtuel. Isolation Hyper-V utilise une carte réseau virtuelle synthétique (non exposée à la machine virtuelle d’utilitaire) pour s’attacher au commutateur virtuel.

![texte](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>Sécurité réseau

Selon le conteneur et le pilote réseau utilisé, les ACL de port sont appliquées par une combinaison du Pare-feu Windows et [VFP ](https://www.microsoft.com/en-us/research/project/azure-virtual-filtering-platform/).

### <a name="windows-server-containers"></a>Conteneurs WindowsServer

Ils utilisent le pare-feu des hôtes de Windows (compatible avec les espaces de noms réseau), ainsi que VFP

* Sortant par défaut: Autoriser tout
* Entrant par défaut: Autoriser tout le trafic réseau non sollicité (TCP, UDP, ICMP, IGMP)
  * Refuser tous les autres trafics réseau ne provenant pas de ces protocoles

  >[!NOTE]
  >Avant Windows Server, version 1709 et Windows 10 Fall Creators Update, la règle de trafic entrant par défaut était de tout refuser. Les utilisateurs exécutant ces versions plus anciennes peuvent créer des règles d’autorisation entrantes avec ``docker run -p`` (réacheminement de port).

### <a name="hyper-v-isolation"></a>Isolation Hyper-V

Les conteneurs en cours d’exécution dans l’isolation Hyper-V ont leur propre noyau isolé et exécutent donc leur propre instance du pare-feu Windows avec la configuration suivante:

* Par défaut tout autoriser dans les deux Pare-feu Windows (en cours d’exécution dans l’utilitaire VM) et VFP

![texte](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Pods Kubernetes

Dans un [bloc de Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/pod/), un conteneur d’infrastructure est d’abord créé auquel un point de terminaison est attaché. Les conteneurs qui appartiennent au même pod, y compris les conteneurs d’infrastructure et de travail, partagent un espace de noms réseau commun (même adresse IP et espace de port).

![texte](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>Personnalisation des ACL de port par défaut

Si vous souhaitez modifier les ACL, lisez d’abord notre documentation sur le Service de mise en réseau hôte (lien bientôt ajouté) de port par défaut. Vous devez mettre à jour des stratégies à l’intérieur des composants suivants:

>[!NOTE]
>Pour l’isolation Hyper-V en mode Transparent et NAT, vous ne pouvez pas actuellement reprogrammer les ACL de port par défaut. Cela est matérialisé par un «X» dans le tableau.

| Pilote de réseau | Conteneurs WindowsServer | Isolation Hyper-V  |
| -------------- |-------------------------- | ------------------- |
| Transparent | Pare-feu Windows | X |
| NAT | Pare-feu Windows | X |
| L2Bridge | Les deux | VFP |
| L2Tunnel | Les deux | VFP |
| Superposition  | Les deux | VFP |
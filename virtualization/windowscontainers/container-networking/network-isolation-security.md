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
ms.openlocfilehash: 7203989483cb07423b70ff8cc644f715ba4be274
ms.sourcegitcommit: ec186664e76d413d3bf75f2056d5acb556f4205d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/11/2018
ms.locfileid: "1876053"
---
# <a name="network-isolation-and-security"></a>Isolation et sécurité de réseau

## <a name="isolation-with-network-namespaces"></a>Isolation avec des espace de noms réseau
Chaque point de terminaison de conteneur est placé dans son propre __espace de noms de réseau__. La vNIC de l’hôte de gestion et la pile réseau hôte se trouvent dans l’espace de nom réseau par défaut. Pour appliquer une isolation réseau entre les conteneurs sur le même hôte, un espace de nom réseau est créé pour chaque conteneur Windows Server et Hyper-V dans lequel est installée la carte réseau pour le conteneur. Les conteneurs Windows Server utilisent une carte réseau virtuelle hôte pour s’attacher au commutateur virtuel. Les conteneurs Hyper-V utilisent une carte réseau de machine virtuelle synthétique (non exposée à la machine virtuelle d’utilitaire) pour s’attacher au commutateur virtuel.


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

  > Remarque: Avant Windows Server, version1709 et Windows10 Fall Creators Update, la règle *entrante* par défaut était de tout refuser. Les utilisateurs exécutant ces versions plus anciennes peuvent créer des règles d’autorisation entrantes à l’aide de ``docker run -p`` (réacheminement de port)


### <a name="hyper-v-containers"></a>Conteneurs Hyper-V
Les conteneurs Hyper-V ont leur propre noyau isolé et exécutent donc leur propre instance du Pare-feu Windows avec la configuration suivante:
  * Par défaut tout autoriser dans les deux Pare-feu Windows (en cours d’exécution dans l’utilitaire VM) et VFP


![texte](media/windows-firewall-containers.png)


### <a name="kubernetes-pods"></a>Pods Kubernetes
Dans les [pods Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/pod/), un conteneur d’infrastructure est d’abord créé auquel un point de terminaison est attaché. Les conteneurs (y compris les conteneurs d’infrastructure et de travail) appartenant au même pod partagent un espace de noms réseau commun (même adresse IP et espace de port).


![texte](media/pod-network-compartment.png)


### <a name="customizing-default-port-acls"></a>Personnalisation des ACL de port par défaut
Si vous souhaitez modifier les ACL de port par défaut, consultez notre documentation HNS (lien bientôt ajouté). Vous devez mettre à jour les stratégies à l’intérieur des composants suivants:

> Remarque: Pour les conteneurs Hyper-V en mode Transparent et NAT, vous ne pouvez pas reprogrammer les ACL de port par défaut actuellement. Cela est matérialisé par un «X» dans le tableau.

| Pilote de réseau | Conteneurs Windows Server | Conteneurs Hyper-V  |
| -------------- |-------------------------- | ------------------- |
| Transparent | Pare-feu Windows | X |
| NAT | Pare-feu Windows | X |
| L2Bridge | Les deux | VFP |
| L2Tunnel | Les deux | VFP |
| Superposition  | Les deux | VFP |
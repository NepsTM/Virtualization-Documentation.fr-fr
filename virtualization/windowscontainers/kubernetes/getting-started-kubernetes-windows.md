---
title: Kubernetes sur Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.prod: containers
ms.topic: overview
description: Joindre un nœud Windows à un cluster Kubernetes avec v 1.14.
keywords: kubernetes, 1,14, Windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 43525b86e00ad3d855f98394aa96ffe987fb51d6
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192326"
---
# <a name="kubernetes-on-windows"></a>Kubernetes sur Windows

Cette page fournit une vue d’ensemble de la prise en main de Kubernetes sur Windows en joignant des nœuds Windows à un cluster Linux. Avec la sortie de Kubernetes 1,14 sur Windows Server [version 1809](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes), les utilisateurs peuvent tirer parti des fonctionnalités suivantes de Kubernetes sur Windows :

- superposition de la **mise en réseau**: utiliser Flannel en mode vxlan pour configurer un réseau de superposition virtuelle
    - requiert l’installation de Windows Server 2019 avec [KB4489899](https://support.microsoft.com/help/4489899) ou [Windows Server vNext Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) Build 18317 +
    - requiert Kubernetes v 1.14 (ou version ultérieure) avec `WinOverlay` fonctionnalité Gate activée
    - requiert Flannel v 0.11.0 (ou version ultérieure)
- **gestion de réseau simplifiée**: utilisez Flannel en mode Host-Gateway pour la gestion automatique des itinéraires entre les nœuds.
- **améliorations de l’évolutivité**: Profitez de temps de démarrage de conteneur plus rapides et plus fiables grâce aux [cartes réseau virtuelles sans appareil pour les conteneurs Windows Server](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716).
- **Isolation Hyper-v (alpha)**: orchestrez l' [isolation Hyper-v](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) avec l’isolation en mode noyau pour renforcer la sécurité. Pour plus d’informations, retrouvez les [types de conteneurs Windows](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types).
    - requiert Kubernetes v 1.10 (ou version ultérieure) avec `HyperVContainer` Feature Gate activé.
- **plug-ins de stockage**: utilisez le [plug-in de stockage FlexVolume](https://github.com/Microsoft/K8s-Storage-Plugins) avec la prise en charge SMB et iSCSI pour les conteneurs Windows.

>[!TIP]
>Si vous souhaitez déployer un cluster sur Azure, l’outil open source AKS-Engine vous facilite la tâche. Pour plus d’informations, consultez notre [procédure pas](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)à pas.

## <a name="prerequisites"></a>Prérequis

### <a name="plan-ip-addressing-for-your-cluster"></a>Planifier l’adressage IP pour votre cluster

<a name="definitions"></a>Comme les clusters Kubernetes introduisent de nouveaux sous-réseaux pour les POD et les services, il est important de s’assurer qu’aucun d’entre eux n’entre en conflit avec d’autres réseaux existants dans votre environnement. Voici tous les espaces d’adressage qui doivent être libérés afin de déployer Kubernetes avec succès :

| Sous-réseau/plage d’adresses | Description | Valeur par défaut |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Sous-réseau de service** | Un sous-réseau non routable, purement virtuel qui est utilisé par les Pod pour accéder uniformément aux services sans se soucier de la topologie du réseau. Il est converti vers/depuis l’espace d’adressage routable par `kube-proxy` en cours d’exécution sur les nœuds. | « 10.96.0.0/12 » |
| <a name="cluster-subnet-def"></a>**Sous-réseau de cluster** |  Il s’agit d’un sous-réseau global qui est utilisé par tous les Pod du cluster. Un sous-réseau plus petit/24 est attribué à chaque nœud pour les Pod à utiliser. Elle doit être suffisamment grande pour accueillir tous les modules utilisés dans votre cluster. Pour calculer la taille *minimale* du sous-réseau : `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Exemple pour un cluster à 5 nœuds pour 100 Pod par nœud : `(5) + (5 *  100) = 505` .  | « 10.244.0.0/16 » |
| **Adresse IP du service DNS Kubernetes** | Adresse IP du service « Kube-DNS » qui sera utilisée pour la résolution DNS & la détection du service de cluster. | "10.96.0.10" |

> [!NOTE]
> Il existe un autre réseau d’ancrage (NAT) qui est créé par défaut lorsque vous installez le Dockr. Il n’est pas nécessaire d’utiliser Kubernetes sur Windows, car nous attribuons des adresses IP à partir du sous-réseau du cluster à la place.

## <a name="what-you-will-accomplish"></a>Les tâches que vous allez accomplir

À la fin de ce guide, vous aurez :

> [!div class="checklist"]
> * Création d’un nœud [maître Kubernetes](./creating-a-linux-master.md) .
> * Sélection d’une [solution réseau](./network-topologies.md).
> * Joint un nœud [Worker Windows](./joining-windows-workers.md) ou un [nœud Worker Linux](./joining-linux-workers.md) à celui-ci.
> * Déploiement d’un [exemple de ressource Kubernetes](./deploying-resources.md).
> * Abordé les [erreurs et problèmes courants](./common-problems.md).

## <a name="next-steps"></a>Étapes suivantes

Dans cette section, nous avons parlé des conditions préalables importantes & hypothèses nécessaires au déploiement de Kubernetes sur Windows avec succès. Continuez à apprendre à configurer un maître Kubernetes :

>[!div class="nextstepaction"]
>[Créer un maître Kubernetes](./creating-a-linux-master.md)
---
title: Kubernetes sur Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Connexion d’un nœud Windows à un cluster Kubernetes avec la version 1.14.
keywords: kubernetes, 1,14, Windows, mise en route
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 18734f102042ec951255061dcd82229e18d29a15
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998386"
---
# <a name="kubernetes-on-windows"></a>Kubernetes sur Windows

Cette page vous fournit une vue d’ensemble de la mise en route de Kubernetes sur Windows en rejoignant des nœuds Windows à un cluster Linux. Avec la publication de Kubernetes 1,14 sur Windows Server [version 1809](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes), les utilisateurs peuvent tirer parti des fonctionnalités suivantes dans Kubernetes sur Windows:

- **superposition réseau**: utilisez Flannel en mode vxlan pour configurer un réseau de superposition virtuelle.
    - nécessite Windows Server 2019 avec [KB4489899](https://support.microsoft.com/help/4489899) installé ou [Windows Server vNext Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) Build 18317 +
    - nécessite Kubernetes v 1.14 (ou une version ultérieure `WinOverlay` ) avec l’option portail de fonctionnalités activée
    - nécessite Flannel v 0.11.0 (ou une version ultérieure)
- **gestion de réseau simplifiée**: utilisez Flannel en mode passerelle hôte pour la gestion automatique des itinéraires entre les nœuds.
- **améliorations de l’évolutivité**: Profitez de temps de démarrage de conteneur plus rapide et plus fiable grâce au [système vNICs pour les conteneurs Windows Server](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716).
- **Isolation Hyper-v (alpha)**: orchestrer l' [isolation Hyper-v](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) avec l’isolation en mode noyau pour renforcer la sécurité. Pour plus d’informations, sur les [types de conteneur Windows](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types).
    - nécessite Kubernetes version 1,10 (ou une version ultérieure `HyperVContainer` ) avec l’option portail de fonctionnalités activée.
- **plug-ins de stockage**: utiliser le [plug-in de stockage FlexVolume](https://github.com/Microsoft/K8s-Storage-Plugins) avec le support SMB et iSCSI pour les conteneurs Windows.

>[!TIP]
>Si vous voulez déployer un cluster sur Azure, l’outil open source AKS-Engine vous simplifie la tâche. Pour en savoir plus, consultez la [procédure pas à pas](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)ci-dessous.

## <a name="prerequisites"></a>Prérequis

### <a name="plan-ip-addressing-for-your-cluster"></a>Planifier l’adressage IP pour votre cluster

<a name="definitions"></a>Étant donné que les clusters Kubernetes introduisent de nouveaux sous-réseaux pour les gousses et les services, il est important de s’assurer qu’aucun d’entre eux ne peut entrer en conflit avec d’autres réseaux existants dans votre environnement. Voici tous les espaces d’adresses qui doivent être libérés pour déployer Kubernetes correctement:

| Intervalle de sous-réseau/adresse | Description | Valeur par défaut |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Sous-réseau de service** | Un sous-réseau virtuel non routable utilisé par les gousses pour accéder uniformément aux services sans vous soucier de la topologie du réseau. Il est converti vers/depuis l’espace d’adressage routable par `kube-proxy` en cours d’exécution sur les nœuds. | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**Sous-réseau de cluster** |  Il s’agit d’un sous-réseau global utilisé par tous les gousses du cluster. Un sous-réseau de plus petite/24 est attribué à chaque nœud pour pouvoir utiliser ses gousses. Le volume doit être suffisant pour accueillir tous les modules utilisés dans votre cluster. Pour calculer la taille de sous-réseau *minimum* : `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Exemple de groupe de cinq nœuds pour les 100 de blocs `(5) + (5 *  100) = 505`par nœud:.  | "10.244.0.0/16" |
| **Adresse IP du service DNS Kubernetes** | Adresse IP du service «Kube-DNS» qui sera utilisé pour la résolution DNS & la découverte du service de cluster. | "10.96.0.10" |

> [!NOTE]
> Il existe un autre réseau d’Arrimement qui est créé par défaut lors de l’installation de l’amarrage. Il n’est pas nécessaire d’utiliser Kubernetes sur Windows, car nous affectons plutôt IPs à partir du sous-réseau de cluster.

## <a name="what-you-will-accomplish"></a>Les tâches que vous allez accomplir

À la fin de ce guide, vous aurez:

> [!div class="checklist"]
> * Création d’un nœud [maître Kubernetes](./creating-a-linux-master.md) .  
> * Sélection d’une [solution réseau](./network-topologies.md).  
> * Vous avez rejoint un [nœud Windows Worker](./joining-windows-workers.md) ou un [nœud Worker Linux](./joining-linux-workers.md) ;  
> * A déployé un [exemple de ressource Kubernetes](./deploying-resources.md).  
> * Abordé les [erreurs et problèmes courants](./common-problems.md).

## <a name="next-steps"></a>Étapes suivantes

Dans cette section, nous avons parlé des conditions préalables importantes & hypothèses nécessaires au déploiement de Kubernetes sur Windows avec succès dès aujourd’hui. Continuez à apprendre à configurer un maître Kubernetes:

>[!div class="nextstepaction"]
>[Créer un maître Kubernetes](./creating-a-linux-master.md)
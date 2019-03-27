---
title: Kubernetes sur Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Jonction d’un nœud Windows à un cluster Kubernetes avec v1.13.
keywords: kubernetes, 1.13, windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: df3185db086e8e38143fe60d90db864038980603
ms.sourcegitcommit: 1715411ac2768159cd9c9f14484a1cad5e7f2a5f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/26/2019
ms.locfileid: "9263486"
---
# <a name="kubernetes-on-windows"></a>Kubernetes sur Windows #
Cette page explique comment une vue d’ensemble de prise en main avec Kubernetes sur Windows en joignant les nœuds de Windows à un cluster Linux. Avec la version de Kubernetes 1.14 sur Windows Server, [version 1809](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes), les utilisateurs peuvent tirer parti des fonctionnalités suivantes dans Kubernetes sur Windows:

  - **mise en réseau de superposition**: utilisez Flannel en mode vxlan pour configurer un réseau virtuel de superposition
    - nécessite deux Windows Server 2019 avec [KB4489899](https://support.microsoft.com/en-us/help/4489899) installé ou [Windows Server vNext Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) Build 18317 +
    - nécessite v1.14 Kubernetes (ou version ultérieure) avec `WinOverlay` porte fonctionnalité activée
    - nécessite Flannel v0.11.0 (ou version ultérieure)
  - **une gestion réseau simplifiée**: utiliser Flannel en mode hôte-passerelle pour la gestion de gamme automatique entre les nœuds
  - **améliorations de l’évolutivité**: profitez de temps de démarrage plus rapide et plus fiable conteneur grâce à [une carte réseau virtuelle sans périphérique pour les conteneurs Windows Server](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/)
  - **isolation Hyper-v (alpha)**: orchestrer ces [conteneurs hyper-v](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) avec l’isolation en mode noyau pour une sécurité améliorée ([voir les types de conteneurs Windows](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types))
    - nécessite la version 1.10 Kubernetes (ou version ultérieure) avec `HyperVContainer` porte fonctionnalité activée
  - **plug-ins de stockage**: utiliser le [plug-in de stockage FlexVolume](https://github.com/Microsoft/K8s-Storage-Plugins) SMB et iSCSI prenant en charge pour les conteneurs Windows

> [!TIP] 
> Si vous souhaitez déployer un cluster sur Azure, l’outil open source AKS-Engine facilite cette opération. Une [procédure pas à pas](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md) est disponible.

## <a name="prerequisites"></a>Prérequis ##

### <a name="plan-ip-addressing-for-your-cluster"></a>Planifier des adresses IP pour votre cluster ###
<a name="definitions"></a>Comme les clusters Kubernetes introduisent de nouveaux sous-réseaux des pods et des services, il est important de s’assurer qu’aucun d'entre eux entrer en collision avec tous les autres réseaux existants dans votre environnement. Voici tous les espaces d’adressage qui doivent être libérées pour déployer correctement de Kubernetes:

| Sous-réseau / plage d’adresses | Description | Valeur par défaut |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Sous-réseau de service** | Un sous-réseau non routable et purement virtuel qui est utilisé par le service pods pour accéder uniformément aux services sans se préoccuper de la topologie du réseau. Il est converti vers/depuis l’espace d’adressage routable par `kube-proxy` en cours d’exécution sur les nœuds. | «10.96.0.0/12» |
| <a name="cluster-subnet-def"></a>**Sous-réseau de cluster** |  Il s’agit d’un sous-réseau global qui est utilisé par tous les pods du cluster. Chaque nœud est attribué à une plus petite /24 sous-réseau à partir de cette pour leurs pods. Il doit être suffisamment large pour prendre en charge tous les pods utilisés dans votre cluster. Pour calculer la taille *minimale* de sous-réseau: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Exemple pour un cluster de 5 nœuds pour 100 POD par nœud: `(5) + (5 *  100) = 505`.  | «10.244.0.0/16» |
| **IP de Service DNS Kubernetes** | Adresse IP de service «kube-dns» qui sera utilisé pour la découverte de service DNS résolution & cluster. | «10.96.0.10» |
> [!NOTE]
> Il existe une autre Docker réseau (NAT) qui est créé par défaut lorsque vous installez Docker. Il n’est pas nécessaire de faire fonctionner Kubernetes sur Windows, comme nous affectons à la place des adresses IP à partir du sous-réseau de cluster.



## <a name="what-you-will-accomplish"></a>Les tâches que vous allez accomplir ##

À la fin de ce guide, vous aurez:

> [!div class="checklist"]
> * Créer un nœud [maître Kubernetes](./creating-a-linux-master.md) .  
> * Sélectionné une [solution de réseau](./network-topologies.md).  
> * Joint à un [nœud de travail Windows](./joining-windows-workers.md) ou d’un [nœud de travail Linux](./joining-linux-workers.md) à celui-ci.  
> * Déployer une [ressource de Kubernetes exemple](./deploying-resources.md).  
> * Abordé les [erreurs et problèmes courants](./common-problems.md).

## <a name="next-steps"></a>Étapes suivantes ##
Dans cette section, nous avons parlé des conditions préalables importantes hypothèses & nécessaires pour déployer correctement aujourd'hui de Kubernetes sur Windows. Continuer à apprendre à configurer un nœud maître Kubernetes:

> [!div class="nextstepaction"]
> [Créer un nœud maître Kubernetes](./creating-a-linux-master.md)
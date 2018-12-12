---
title: Kubernetes sur Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Jonction d’un nœud Windows à un cluster Kubernetes avec v1.12.
keywords: kubernetes, 1.12, windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 0e43b2ac5b19d16721c1ba0dd1f34e339223bdaf
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178902"
---
# <a name="kubernetes-on-windows"></a>Kubernetes sur Windows #
Cette page explique comment une vue d’ensemble de prise en main avec Kubernetes sur Windows en joignant les nœuds de Windows à un cluster Linux. Avec la version de Kubernetes 1.12 sur la version bêta de Windows Server [version 1803](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1803#kubernetes) , les utilisateurs peuvent tirer parti des [fonctionnalités les plus récentes](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features) dans Kubernetes sur Windows:

  - **une gestion réseau simplifiée**: utiliser des Flannel en mode hôte-passerelle pour la gestion de gamme automatique entre les nœuds
  - **améliorations de l’évolutivité**: profitez d’un temps de démarrage plus rapide et plus fiable conteneur grâce à [des cartes réseau virtuelles sans périphérique pour les conteneurs Windows Server](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/)
  - **isolation Hyper-v (alpha)**: orchestrer ces [conteneurs hyper-v](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) avec l’isolation en mode noyau pour une sécurité améliorée ([voir les types de conteneurs Windows](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types))
  - **plug-ins de stockage**: utiliser le [plug-in de stockage FlexVolume](https://github.com/Microsoft/K8s-Storage-Plugins) SMB et iSCSI prenant en charge pour les conteneurs Windows

> [!TIP] 
> Si vous souhaitez déployer un cluster sur Azure, l’outil open source ACS-Engine facilite cette opération. Une [procédure pas à pas](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/windows.md) est disponible.

## <a name="prerequisites"></a>Conditions préalables ##

### <a name="plan-ip-addressing-for-your-cluster"></a>Planifier l’adressage IP pour votre cluster ###
<a name="definitions"></a>Comme les clusters Kubernetes introduisent de nouveaux sous-réseaux des pods et des services, il est important de s’assurer qu’aucun d'entre eux n’entrent en collision avec tous les autres réseaux existants dans votre environnement. Voici tous les espaces d’adressage qui doivent être libérées pour déployer correctement de Kubernetes:

| Sous-réseau / de la plage d’adresses | Description | Valeur par défaut |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Sous-réseau de service** | Un sous-réseau non routable et purement virtuel qui est utilisé par le service pods pour accéder uniformément aux services sans se préoccuper de la topologie du réseau. Il est converti vers/depuis l’espace d’adressage routable par `kube-proxy` en cours d’exécution sur les nœuds. | «10.96.0.0/12» |
| <a name="cluster-subnet-def"></a>**Sous-réseau de cluster** |  Il s’agit d’un sous-réseau global qui est utilisé par tous les pods du cluster. Chaque nœud est attribué à une plus petite /24 sous-réseau à partir de cette pour leurs pods. Il doit être suffisamment large pour prendre en charge tous les pods utilisés dans votre cluster. Pour calculer la taille *minimale* de sous-réseau: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Exemple pour un cluster de 5 nœuds pour 100 POD par nœud: `(5) + (5 *  100) = 505`.  | «10.244.0.0/16» |
| **IP de Service DNS de Kubernetes** | Adresse IP de service «kube-dns» qui sera utilisé pour la découverte de service de cluster et de résolution DNS. | «10.96.0.10» |
> [!NOTE]
> Il existe un autre réseau de Docker (NAT) qui est créé par défaut lorsque vous installez Docker. Il n’est pas nécessaire de faire fonctionner Kubernetes sur Windows, comme nous attribuons à la place des adresses IP à partir du sous-réseau de cluster.

### <a name="disable-anti-spoofing-protection"></a>Désactiver la protection usurpation ###
> [!Important] 
> Lisez attentivement cette section qu’elle est nécessaire pour tout le monde d’utiliser correctement les ordinateurs virtuels à déployer Kubernetes sur Windows actuellement.

Assurez-vous que l’usurpation des adresses MAC et la virtualisation est activée pour l’hôte de conteneur Windows machines virtuelles (invités). Pour ce faire, vous devez exécuter la commande suivante en tant qu’administrateur sur l’ordinateur qui héberge les machines virtuelles (exemple donné pour Hyper-V):

```powershell
Set-VMProcessor -VMName "<name>" -ExposeVirtualizationExtensions $true 
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```
> [!TIP]
> Si vous utilisez un produit VMware pour répondre aux besoins de la virtualisation, recherchez dans l’activation du [mode promiscuous](https://kb.vmware.com/s/article/1004099) pour l’exigence d’usurpation d’identité MAC.

>[!TIP]
> Si vous déployez Kubernetes sur les machines virtuelles Azure IaaS vous-même, recherchez dans des machines virtuelles qui prennent en charge [la virtualisation imbriquée](https://azure.microsoft.com/en-us/blog/nested-virtualization-in-azure/) pour cette exigence.

## <a name="what-you-will-accomplish"></a>Les tâches que vous allez accomplir ##

À la fin de ce guide, vous aurez:

> [!div class="checklist"]
> * Créer un nœud [maître Kubernetes](./creating-a-linux-master.md) .  
> * Sélectionné une [solution de réseau](./network-topologies.md).  
> * Joint à un [nœud de travail Windows](./joining-windows-workers.md) ou d’un [nœud de travail Linux](./joining-linux-workers.md) à celui-ci.  
> * Déployer une [ressource de Kubernetes exemple](./deploying-resources.md).  
> * Abordé les [erreurs et problèmes courants](./common-problems.md).

## <a name="next-steps"></a>Étapes suivantes ##
Dans cette section, nous avons parlé importants conditions préalables et les hypothèses nécessaires pour déployer correctement aujourd'hui de Kubernetes sur Windows. Continuer à apprendre à configurer un master Kubernetes:

> [!div class="nextstepaction"]
> [Créer un nœud maître Kubernetes](./creating-a-linux-master.md)
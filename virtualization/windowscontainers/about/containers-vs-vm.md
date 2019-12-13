---
title: Conteneurs et machines virtuelles
description: 'Cette rubrique présente quelques-unes des similitudes et différences clés entre les conteneurs et les machines virtuelles, et quand vous souhaitez les utiliser. Les conteneurs et les machines virtuelles ont chacun leur propre utilisation : en fait, de nombreux déploiements de conteneurs utilisent des machines virtuelles comme système d’exploitation hôte plutôt que de s’exécuter directement sur le matériel, en particulier lors de l’exécution de conteneurs dans le Cloud.'
keywords: ancrage, conteneurs, machines virtuelles, machines virtuelles
author: jasongerend
ms.author: jgerend
ms.date: 10/21/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 63150dfde007ec942446387064ad59f05b0aaa43
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910819"
---
# <a name="containers-vs-virtual-machines"></a>Conteneurs et machines virtuelles

Cette rubrique présente quelques-unes des similitudes et des différences clés entre les conteneurs et les machines virtuelles, et quand vous souhaitez les utiliser. Les conteneurs et les machines virtuelles ont chacun leur utilisation : en fait, de nombreux déploiements de conteneurs utilisent des machines virtuelles comme système d’exploitation hôte plutôt que de s’exécuter directement sur le matériel, en particulier lors de l’exécution de conteneurs dans le Cloud.

Pour obtenir une vue d’ensemble des conteneurs, consultez [fenêtres et conteneurs](index.md).

## <a name="container-architecture"></a>Architecture de conteneur

Un conteneur est un silo léger isolé pour l’exécution d’une application sur le système d’exploitation hôte. Les conteneurs sont générés par-dessus le noyau du système d’exploitation hôte (ce qui peut être considéré comme le plomberie du système d’exploitation) et contiennent uniquement des applications et des services et des API de système d’exploitation légers qui s’exécutent en mode utilisateur, comme indiqué dans ce diagramme.

![Diagramme architectural montrant comment les conteneurs s’exécutent au-dessus du noyau](media/container-diagram.svg)

## <a name="virtual-machine-architecture"></a>Architecture d'ordinateurs virtuels

Contrairement aux conteneurs, les machines virtuelles exécutent un système d’exploitation complet, y compris son propre noyau, comme indiqué dans ce diagramme.

![Diagramme architectural montrant comment les machines virtuelles exécutent un système d’exploitation complet à côté du système d’exploitation hôte](media/virtual-machine-diagram.svg)

## <a name="containers-vs-virtual-machines"></a>Conteneurs et machines virtuelles

Le tableau suivant présente certaines similarités et différences de ces technologies complémentaires.

|                 | Ordinateur virtuel  | Conteneur  |
| --------------  | ---------------- | ---------- |
| Isolation       | Fournit une isolation complète du système d’exploitation hôte et des autres machines virtuelles. Cela est utile quand une limite de sécurité forte est essentielle, par exemple l’hébergement d’applications provenant de sociétés concurrentes sur le même serveur ou cluster. | Fournit généralement une isolation légère de l’hôte et d’autres conteneurs, mais ne fournit pas autant de limite de sécurité qu’une machine virtuelle. (Vous pouvez augmenter la sécurité en utilisant le [mode d’isolation Hyper-V](../manage-containers/hyperv-container.md) pour isoler chaque conteneur dans une machine virtuelle légère). |
| Système d’exploitation | Exécute un système d’exploitation complet, y compris le noyau, ce qui nécessite davantage de ressources système (processeur, mémoire et stockage). | Exécute la partie mode utilisateur d’un système d’exploitation et peut être personnalisée pour contenir uniquement les services nécessaires à votre application, en utilisant moins de ressources système. |
| Compatibilité des invités | S’exécute sur n’importe quel système d’exploitation à l’intérieur de la machine virtuelle | S’exécute sur la [même version du système d’exploitation que l’hôte (l'](../deploy-containers/version-compatibility.md) isolation Hyper-V vous permet d’exécuter des versions antérieures du même système d’exploitation dans un environnement de machine virtuelle léger)
| Déploiement     | Déployer des machines virtuelles individuelles à l’aide du centre d’administration Windows ou du Gestionnaire Hyper-V ; Déployez plusieurs machines virtuelles à l’aide de PowerShell ou de System Center Virtual Machine Manager. | Déployer des conteneurs individuels à l’aide de l’ancrage via la ligne de commande ; Déployez plusieurs conteneurs à l’aide d’un orchestrateur tel que le service Azure Kubernetes. |
| Mises à jour et mises à niveau du système d’exploitation | Téléchargez et installez les mises à jour du système d’exploitation sur chaque machine virtuelle. L’installation d’une nouvelle version du système d’exploitation nécessite la mise à niveau ou la création d’une machine virtuelle entièrement nouvelle. Cela peut prendre du temps, en particulier si vous avez beaucoup de machines virtuelles... | La mise à jour ou la mise à niveau des fichiers du système d’exploitation au sein d’un conteneur est la même : <br><ol><li>Modifiez le fichier de build de votre image conteneur (connu sous le nom de fichier dockerfile) pour pointer vers la dernière version de l’image de base Windows. </li><li>Régénérez votre image conteneur avec cette nouvelle image de base.</li><li>Transmettent l’image conteneur à votre registre de conteneurs.</li> <li>Redéploiement à l’aide d’un orchestrateur.<br>L’orchestrateur offre une automatisation puissante pour ce faire à grande échelle. Pour plus d’informations, consultez [Didacticiel : mettre à jour une application dans le service Azure Kubernetes](https://docs.microsoft.com/azure/aks/tutorial-kubernetes-app-update).</li></ol> |
| Stockage persistant | Utiliser un disque dur virtuel (VHD) pour le stockage local pour une machine virtuelle unique, ou un partage de fichiers SMB pour le stockage partagé par plusieurs serveurs | Utilisez des disques Azure pour le stockage local pour un seul nœud, ou Azure Files (partages SMB) pour le stockage partagé par plusieurs nœuds ou serveurs. |
| Équilibrage de charge | L’équilibrage de charge des ordinateurs virtuels déplace les machines virtuelles en cours d’exécution vers d’autres serveurs dans un cluster de basculement. | Les conteneurs eux-mêmes ne sont pas déplacés ; au lieu de cela, un orchestrateur peut démarrer ou arrêter automatiquement des conteneurs sur des nœuds de cluster pour gérer les modifications de charge et de disponibilité. |
| Tolérance de panne | Les machines virtuelles peuvent basculer vers un autre serveur dans un cluster, avec le redémarrage du système d’exploitation de la machine virtuelle sur le nouveau serveur.  | En cas de défaillance d’un nœud de cluster, tous les conteneurs en cours d’exécution sont recréés rapidement par l’orchestrateur sur un autre nœud de cluster. |
| Mise en réseau     | Utilise des cartes réseau virtuelles. | Utilise une vue isolée d’une carte réseau virtuelle, ce qui fournit un peu moins de virtualisation : le pare-feu de l’ordinateur hôte est partagé avec des conteneurs, tout en utilisant moins de ressources. Pour plus d’informations, consultez [Windows Container Networking](../container-networking/architecture.md). |
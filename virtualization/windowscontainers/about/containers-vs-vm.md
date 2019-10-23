---
title: Conteneurs et machines virtuelles
description: 'Cette rubrique décrit certaines des similitudes et des différences principales entre les conteneurs et les machines virtuelles, et les situations dans lesquelles vous pouvez les utiliser. Les conteneurs et machines virtuelles ont chacun de leurs usages: en fait, de nombreux déploiements de conteneurs utilisent des machines virtuelles comme système d’exploitation hôte plutôt que de s’exécuter directement sur le matériel, en particulier lors de l’exécution de conteneurs dans le Cloud.'
keywords: docker, conteneurs, VM, machines virtuelles
author: jasongerend
ms.author: jgerend
ms.date: 10/21/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 63150dfde007ec942446387064ad59f05b0aaa43
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254394"
---
# <a name="containers-vs-virtual-machines"></a>Conteneurs et machines virtuelles

Cette rubrique présente certaines des similitudes et des différences clés entre les conteneurs et les machines virtuelles, et les situations dans lesquelles vous pouvez les utiliser. Les conteneurs et machines virtuelles disposent chacun de leurs usages (en fait, de nombreux déploiements de conteneurs utilisent des VM comme système d’exploitation hôte plutôt que de s’exécuter directement sur le matériel, en particulier lors de l’exécution de conteneurs dans le Cloud).

Pour obtenir une vue d’ensemble des conteneurs, voir [fenêtres et conteneurs](index.md).

## <a name="container-architecture"></a>Architecture de conteneur

Un conteneur est un silo léger isolé d’exécution d’une application sur le système d’exploitation hôte. Les conteneurs se créent au-dessus du noyau du système d’exploitation hôte (qui peut être considéré comme la plomberie du système d’exploitation) et contiennent uniquement des applications et des API et services de système d’exploitation léger qui s’exécutent en mode utilisateur, comme illustré dans ce diagramme.

![Diagramme architectural montrant la façon dont les conteneurs s’exécutent au-dessus du noyau](media/container-diagram.svg)

## <a name="virtual-machine-architecture"></a>Architecture d’ordinateur virtuel

Contrairement aux conteneurs, VMs exécute un système d’exploitation complet, y compris son propre noyau, comme le montre ce diagramme.

![Diagramme architectural montrant comment les ordinateurs virtuels exécutent un système d’exploitation complet en regard du système d’exploitation hôte](media/virtual-machine-diagram.svg)

## <a name="containers-vs-virtual-machines"></a>Conteneurs et machines virtuelles

Le tableau suivant présente certaines des similitudes et des différences entre ces technologies complémentaires.

|                 | Machine virtuelle  | Servoir  |
| --------------  | ---------------- | ---------- |
| Problèmes       | Fournit un isolement complet du système d’exploitation hôte et des autres machines virtuelles. Cela est utile lorsqu’une limite de sécurité forte est essentielle, par exemple pour héberger des applications d’entreprises concurrentes sur le même serveur ou cluster. | Fournit généralement un isolement léger à partir de l’hôte et d’autres conteneurs, mais ne fournit pas une limite de sécurité en tant qu’ordinateur virtuel. (Vous pouvez augmenter la sécurité en utilisant le [mode d’isolation Hyper-V](../manage-containers/hyperv-container.md) pour isoler chaque conteneur dans une VM légère). |
| Système d’exploitation | Exécute un système d’exploitation complet, y compris le noyau, qui nécessite plus de ressources système (processeur, mémoire et stockage). | Exécute la partie du mode utilisateur d’un système d’exploitation et peut être personnalisée pour contenir uniquement les services nécessaires pour votre application, en utilisant moins de ressources système. |
| Compatibilité des invités | S’exécute sur tout système d’exploitation de l’ordinateur virtuel | S’exécute sur la [même version de système d’exploitation que l’hôte (l'](../deploy-containers/version-compatibility.md) isolation Hyper-V vous permet d’exécuter des versions antérieures du même système d’exploitation dans un environnement VM léger);
| Déploiement     | Déploiement d’ordinateurs virtuels individuels à l’aide du centre d’administration Windows ou du Gestionnaire Hyper-V; déploiement de plusieurs machines virtuelles à l’aide de PowerShell ou System Center Virtual Machine Manager. | Déployez des conteneurs individuels à l’aide de l’Arrimateur via la ligne de commande. déploiement de plusieurs conteneurs à l’aide d’un Orchestrator tel qu’Azure Kubernetes service. |
| Mises à jour et mises à niveau du système d’exploitation | Téléchargez et installez les mises à jour du système d’exploitation sur chaque VM. L’installation d’une nouvelle version du système d’exploitation nécessite une mise à niveau ou la création d’un nouvel ordinateur virtuel uniquement. Cela peut prendre du temps, en particulier si vous disposez d’un grand nombre d’ordinateurs virtuels... | La mise à jour ou la mise à niveau des fichiers du système d’exploitation dans un conteneur est la même: <br><ol><li>Modifiez le fichier de build de votre image conteneur (appelé Dockerfile) pour qu’il pointe vers la dernière version de l’image de base Windows. </li><li>Reconstruisez votre image de conteneur avec cette nouvelle image de base.</li><li>Envoyez l’image du conteneur vers le registre de votre conteneur.</li> <li>Redéploiement à l’aide d’un Orchestrator.<br>EPolicy Orchestrator fournit une Automation puissante pour cela à l’échelle. Pour plus d’informations, voir [Didacticiel: mettre à jour une application dans Azure Kubernetes service](https://docs.microsoft.com/azure/aks/tutorial-kubernetes-app-update).</li></ol> |
| Stockage persistant | Utiliser un disque dur virtuel (VHD) pour le stockage local d’un VM unique ou d’un partage de fichiers SMB pour le stockage partagé par plusieurs serveurs | Utilisez les disques Azure pour le stockage local d’un nœud unique, ou des fichiers Azure (partages SMB) pour le stockage partagé par plusieurs nœuds ou serveurs. |
| Équilibrage de la charge | Le service d’équilibrage de la charge de l’ordinateur virtuel bascule vers d’autres serveurs dans un cluster de basculement. | Les conteneurs ne bougent pas; au lieu de cela, un Orchestrator peut démarrer ou arrêter automatiquement des conteneurs sur des nœuds de cluster pour gérer les modifications du chargement et de la disponibilité. |
| Tolérance de panne | Les VM peuvent basculer sur un autre serveur dans un cluster, avec le système d’exploitation de l’ordinateur virtuel redémarré sur le nouveau serveur.  | En cas d’échec d’un nœud de cluster, tous les conteneurs en cours d’exécution sont recréés par l’Orchestrator sur un autre nœud de cluster. |
| Réseaux     | Utilise des cartes réseau virtuelles. | Utilise un affichage isolé d’une carte de réseau virtuel, offrant ainsi un peu moins de virtualisation: le pare-feu de l’hôte est partagé avec des conteneurs, tout en utilisant moins de ressources. Pour plus d’informations, consultez [mise en réseau de conteneurs Windows](../container-networking/architecture.md). |
---
title: Vue d’ensemble de l’orchestration de conteneur Windows
description: En savoir plus sur les orchestrateurs de conteneurs Windows.
keywords: docker, conteneurs
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 23dd1e56ba68a679945779f5e7dbc15225412934
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853904"
---
# <a name="windows-container-orchestration-overview"></a>Vue d’ensemble de l’orchestration de conteneur Windows

En raison de leur taille réduite et de leur orientation d’application, les conteneurs sont idéaux pour les environnements de livraison agile et les architectures basées sur les microservices. Toutefois, un environnement qui utilise des conteneurs et des microservices peut avoir des centaines ou des milliers de composants à suivre. Vous serez peut-être en mesure de gérer manuellement quelques douzaines de machines virtuelles ou de serveurs physiques, mais il n’existe aucun moyen de gérer correctement un environnement de conteneur à l’échelle de la production sans l’automatisation. Cette tâche doit appartenir à votre orchestrateur, qui est un processus qui automatise et gère un grand nombre de conteneurs et comment ils interagissent entre eux.

Les orchestrateurs effectuent les tâches suivantes :

- Planification : quand une image de conteneur et une demande de ressource sont fournies, l’orchestrateur trouve un ordinateur approprié sur lequel exécuter le conteneur.
- Affinité/anti-affinité : spécifiez si un ensemble de conteneurs doit s’exécuter à proximité les uns des autres pour des performances ou de loin pour la disponibilité.
- Contrôle d’intégrité : surveiller les défaillances des conteneurs et les replanifier automatiquement.
- Basculement : effectuer le suivi de ce qui s’exécute sur chaque ordinateur et Replanifier des conteneurs à partir de machines défaillantes vers des nœuds sains.
- Mise à l’échelle : ajoutez ou supprimez des instances de conteneur pour correspondre à la demande, manuellement ou automatiquement.
- Mise en réseau : fournissez un réseau de superposition qui coordonne les conteneurs pour communiquer sur plusieurs ordinateurs hôtes.
- Détection du service : activer les conteneurs pour qu’ils puissent se localiser les uns et les autres automatiquement même s’ils sont déplacés d’un ordinateur hôte à l’autre et que les adresses IP sont changées.
- Mises à niveau d’applications coordonnées : gérer les mises à niveau des conteneurs pour éviter les temps d’arrêt des applications et permettre la restauration en cas de problème.

## <a name="orchestrator-types"></a>Types Orchestrator

Azure offre deux orchestrateurs de conteneurs : Azure Kubernetes service (AKS) et Service Fabric.

[Azure Kubernetes service (AKS)](/azure/aks/) simplifie la création, la configuration et la gestion d’un cluster de machines virtuelles préconfigurées pour exécuter des applications en conteneur. Cela vous permet d’utiliser vos compétences existantes et de tirer parti d’une grande expertise de la communauté pour déployer et gérer des applications basées sur des conteneurs sur Microsoft Azure. En utilisant AKS, vous pouvez tirer parti des fonctionnalités d’entreprise d’Azure tout en conservant la portabilité des applications via Kubernetes et le format d’image de l’arrimeur.

[Azure Service Fabric](/azure/service-fabric/) est une plateforme de systèmes distribués qui facilite la création de packages pour déployer et gérer des microservices et des conteneurs fiables et évolutifs. Service Fabric permet de relever les défis importants dans le développement et la gestion d’applications natives cloud. Les développeurs et les administrateurs peuvent éviter des problèmes d’infrastructure complexes et mettre l’accent sur la mise en œuvre de charges de travail critiques et importantes qui soient évolutives, fiables et facilement gérées. Service Fabric représente la plateforme de nouvelle génération pour la création et la gestion de ces applications d’entreprise, de niveau 1, pour le cloud exécutées dans des conteneurs.

## <a name="getting-started"></a>Mise en route

Pour commencer à déployer le service Azure Kubernetes, consultez le [Guide d’installation de Kubernetes](../kubernetes/getting-started-kubernetes-windows.md).

Pour commencer à déployer Azure Service Fabric, consultez le Guide de [démarrage rapide service Fabric](/azure/service-fabric/service-fabric-quickstart-containers.md).

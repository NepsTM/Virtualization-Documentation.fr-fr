---
title: Présentation de l'orchestration de conteneurs Windows
description: En savoir plus sur les orchestrateurs de conteneurs Windows
keywords: docker, conteneurs
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 23dd1e56ba68a679945779f5e7dbc15225412934
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/24/2020
ms.locfileid: "78853904"
---
# <a name="windows-container-orchestration-overview"></a>Présentation de l'orchestration de conteneurs Windows

En raison de leur petite taille et de leur orientation application, les conteneurs sont parfaits pour les environnements de livraison agiles et les architectures basées sur les microservices. Toutefois, un environnement qui utilise des conteneurs et des microservices peut avoir des centaines ou des milliers de composants à suivre. Vous pouvez peut-être gérer manuellement quelques dizaines de machines virtuelles ou de serveurs physiques, mais il est impossible de gérer correctement un environnement de conteneurs à l'échelle de la production sans automatisation. Cette tâche doit incomber à votre orchestrateur ; il s'agit d'un processus qui automatise et gère un grand nombre de conteneurs ainsi que la façon dont ils interagissent les uns avec les autres.

Les orchestrateurs effectuent les tâches suivantes :

- Planification : lorsqu'il reçoit une image de conteneur et une demande de ressource, l'orchestrateur recherche un ordinateur approprié sur lequel exécuter le conteneur.
- Affinité/Anti-affinité : spécifier si des conteneurs doivent être exécutés les uns à côté des autres pour des raisons de performances ou à distance pour des raisons de disponibilité.
- Surveillance de l'intégrité : surveiller les défaillances des conteneurs et les replanifier automatiquement.
- Basculement : suivre ce qui est en cours d'exécution sur chaque ordinateur et replanifier les conteneurs exécutés sur des ordinateurs en état d'échec vers des nœuds intègres.
- Évolutivité : ajouter ou supprimer des conteneurs pour répondre à la demande, manuellement ou automatiquement.
- Réseaux : fournir un réseau de superposition qui coordonne les conteneurs de manière à leur permettre de communiquer à travers plusieurs ordinateurs hôtes.
- Détection de service : permettre aux conteneurs de se localiser automatiquement, même s'ils sont déplacés d'un ordinateur hôte à un autre et changent d'adresse IP.
- Mises à niveau d'applications coordonnées : gérer les mises à niveau des conteneurs pour éviter les temps d'arrêt des applications et permettre une restauration en cas de problème.

## <a name="orchestrator-types"></a>Types d'orchestrateurs

Azure propose deux orchestrateurs de conteneurs : Azure Kubernetes Service (AKS) et Service Fabric.

[Azure Kubernetes Service (AKS)](/azure/aks/) facilite la création, la configuration et la gestion d'un cluster de machines virtuelles préconfigurées pour exécuter des applications en conteneur. Cela vous permet d'utiliser vos compétences et d'exploiter le savoir-faire d'une communauté toujours plus importante pour déployer et gérer les applications basées sur conteneurs sur Microsoft Azure. À l'aide d'AKS, vous pouvez tirer parti des fonctionnalités d'entreprise d'Azure tout en conservant la portabilité des applications par le biais de Kubernetes et du format d'image Docker.

[Azure Service Fabric](/azure/service-fabric/) est une plateforme de systèmes distribués qui facilite la création de packages pour déployer et gérer des microservices et des conteneurs fiables et évolutifs. Service Fabric permet de relever les défis importants dans le développement et la gestion d’applications natives cloud. Les développeurs et les administrateurs peuvent éviter des problèmes d’infrastructure complexes et mettre l’accent sur la mise en œuvre de charges de travail critiques et importantes qui soient évolutives, fiables et facilement gérées. Service Fabric représente la plateforme de nouvelle génération pour la création et la gestion de ces applications d’entreprise, de niveau 1, pour le cloud exécutées dans des conteneurs.

## <a name="getting-started"></a>Mise en route

Pour plus d'informations sur le déploiement d'Azure Kubernetes Service, consultez le [Guide de configuration de Kubernetes](../kubernetes/getting-started-kubernetes-windows.md).

Pour plus d'informations sur le déploiement d'Azure Service Fabric, consultez le [Guide de démarrage rapide de Service Fabric](/azure/service-fabric/service-fabric-quickstart-containers.md).

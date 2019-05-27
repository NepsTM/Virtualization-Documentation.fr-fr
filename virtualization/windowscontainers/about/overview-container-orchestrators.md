---
title: À propos des orchestreurs de conteneur Windows
description: En savoir plus sur les orchestreurs de conteneur Windows.
keywords: docker, conteneurs
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 1ccf63b0ae55501ba32f8bdd61994e7f8006b5e6
ms.sourcegitcommit: daf1d2b5879c382404fc4d59f1c35c88650e20f7
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/23/2019
ms.locfileid: "9674878"
---
# <a name="about-windows-container-orchestrators"></a>À propos des orchestreurs de conteneur Windows

En raison de leurs petites tailles et de leur orientation d’application, les conteneurs sont adaptés aux environnements de remise et aux architectures de type microservice flexibles. Toutefois, un environnement qui utilise des conteneurs et des microserveurs peut avoir des centaines ou des milliers de composants à suivre. Il est possible que vous puissiez gérer manuellement quelques douzaines de machines virtuelles ou serveurs physiques, mais il n’est pas possible de gérer correctement un environnement de conteneur d’évolution de production sans Automation. Cette tâche doit concerner votre Orchestrator, qui est un processus qui permet d’automatiser et de gérer un grand nombre de conteneurs et la manière dont ils interagissent entre eux.

Les orchestreurs effectuent les tâches suivantes:

- Planification: lorsqu’une image de conteneur est spécifiée et qu’une demande de ressource est spécifiée, le Orchestrator recherche une machine appropriée sur laquelle exécuter le conteneur.
- Affinité/anti-affinité: indiquez si un ensemble de conteneurs doit s’exécuter près les uns des autres pour des performances ou éloignées en fonction de la disponibilité.
- Contrôle d’intégrité: surveiller les défaillances des conteneurs et les replanifier automatiquement.
- Basculement: effectuer le suivi de ce qui est en cours d’exécution sur chaque ordinateur et replanifier les conteneurs à partir d’ordinateurs défectueux vers des nœuds sains.
- Mise à l’échelle: ajoutez ou supprimez des instances de conteneur pour correspondre à la demande, manuellement ou automatiquement.
- Réseau: fournissez un réseau de superposition qui incoordonnéesa les conteneurs pour communiquer sur plusieurs machines hôtes.
- Détection du service: activer les conteneurs pour qu’ils puissent se localiser les uns et les autres automatiquement même s’ils sont déplacés d’un ordinateur hôte à l’autre et que les adresses IP sont changées.
- Mises à niveau d’applications coordonnées: gérer les mises à niveau des conteneurs pour éviter les temps d’arrêt des applications et permettre la restauration en cas de problème.

## <a name="orchestrator-types"></a>Types Orchestrator

Azure propose deux services d’orchestration de conteneur: Azure Kubernetes service (AKS) et service fabric.

Le [service Azure Kubernetes (AKS)](/azure/aks/) simplifie la création, la configuration et la gestion d’un cluster d’applications virtuelles préconfigurées pour exécuter des applications contenant des conteneurs. Cela vous permet d’utiliser vos compétences existantes et d’avoir recours à un vaste éventail de compétences de la communauté pour le déploiement et la gestion des applications basées sur des conteneurs sur Microsoft Azure. En utilisant AKS, vous pouvez tirer parti des fonctionnalités de niveau entreprise d’Azure tout en préservant la portabilité de l’application par le biais de Kubernetes et du format d’image de l’ancrage.

[Azure Service Fabric](/azure/service-fabric/) est une plateforme de systèmes distribués qui facilite la création de packages pour déployer et gérer des microservices et des conteneurs fiables et évolutifs. Service Fabric permet de relever les défis importants dans le développement et la gestion d’applications natives cloud. Les développeurs et les administrateurs peuvent éviter des problèmes d’infrastructure complexes et mettre l’accent sur la mise en œuvre de charges de travail critiques et importantes qui soient évolutives, fiables et facilement gérées. Service Fabric représente la plateforme de nouvelle génération pour la création et la gestion de ces applications d’entreprise, de niveau 1, pour le cloud exécutées dans des conteneurs.

## <a name="getting-started"></a>Prise en main

Pour commencer à déployer le service Azure Kubernetes, voir le [Guide de configuration d’Kubernetes](../kubernetes/getting-started-kubernetes-windows.md).

Pour commencer le déploiement d’Azure Service fabric, voir [démarrage rapide de service Fabric](/azure/service-fabric/service-fabric-quickstart-containers.md).
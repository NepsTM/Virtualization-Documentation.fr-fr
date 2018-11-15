---
title: Écosystème de conteneurs
description: Création d’un écosystème de conteneurs.
keywords: métadonnées, conteneurs
author: PatrickLang
ms.date: 04/20/2016
ms.topic: about-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 29fbe13a-228a-4eaa-9d4d-90ae60da5965
ms.openlocfilehash: e88f2951abef72bdd938c23a068cc912e31628cb
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2018
ms.locfileid: "6948068"
---
# <a name="building-a-container-ecosystem"></a>Création d’un écosystème de conteneurs

Pour comprendre pourquoi la création d’un écosystème de conteneurs est si importante, parlons tout d’abord de Docker.

## <a name="dockers-appeal"></a>Appel de Docker

Le concept de conteneurs (isolation des espaces de noms et gouvernance des ressources) est connu depuis longtemps et remonte aux jails BSD, aux zones Solaris et au mécanisme chroot (change root) UNIX de base.   La contribution de Docker a consisté à fournir un ensemble d’outils communs, un modèle de package et un mécanisme de déploiement.  En procédant ainsi, Docker a considérablement simplifié la mise en conteneur et la distribution des applications.  Ces applications peuvent ensuite être exécutées n’importe où sur tout hôte Linux, une fonctionnalité que nous fournissons également sur Windows.

Cette technologie très répandue simplifie non seulement la gestion en offrant les mêmes commandes d’administration par rapport à n’importe quel hôte, mais crée également une opportunité unique pour les opérations de développement transparentes.

Qu’il s’agisse de l’ordinateur de bureau d’un développeur, d’un ordinateur de test ou d’un groupe de machines de production, il est possible de créer une image Docker qui sera déployée de façon identique dans tous les environnements en quelques secondes. Cette histoire a généré un écosystème croissant et à grande échelle d’applications empaquetées dans des conteneurs Docker, avec Docker Hub, le registre public des applications en conteneur géré par Docker.

Docker pose des fondements solides pour le développement.

Parlons maintenant de cet écosystème d’applications et de la façon dont vous pouvez vous baser sur les concepts de Docker pour créer un workflow de développement et de déploiement adapté à vos besoins.

## <a name="components-in-a-container-ecosystem"></a>Composants d’un écosystème de conteneurs

Les conteneurs Windows sont un composant clé d’un important écosystème de conteneurs. Nous travaillons dans toute l’industrie pour offrir les choix des développeurs à chaque niveau de la pile de solution.

L’écosystème de conteneurs fournit des méthodes pour gérer des conteneurs, partager des conteneurs et développer des applications qui s’exécutent dans des conteneurs.

![](media/containerEcosystem.png)

Microsoft souhaite encourager la productivité et les choix des développeurs à mesure qu’ils génèrent ces applications de nouvelle génération.  Notre objectif est d’alimenter la productivité des développeurs, ce qui signifie que les applications doivent pouvoir cibler n’importe quel cloud Microsoft sans avoir à modifier, réécrire ni reconfigurer le code.

Microsoft s’engage à se montrer ouvert et soucieux des écosystèmes.  Nous supportons de façon active l’union de plusieurs écosystèmes de développeurs dignes d’intérêt, tels que Windows et Linux, pour stimuler l’innovation.

Dans les prochains mois, nous vous fournirons des informations supplémentaires sur d’autres partenaires dans cet écosystème en expansion.

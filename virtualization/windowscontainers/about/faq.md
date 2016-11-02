---
title: FAQ sur les conteneurs Windows
description: FAQ sur les conteneurs Windows
keywords: docker, conteneurs
author: PatrickLang
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 90e32caabde1afafbd8170db77c6e84696395870

---

# Forum Aux Questions

## À propos des conteneurs Windows

**Qu’est-ce qu’un conteneur Windows Server ?**

Les conteneurs Windows Server constituent une méthode de virtualisation de système d’exploitation légère et utilisée pour séparer des applications ou des services d’autres services exécutés sur le même hôte de conteneur. Pour ce faire, chaque conteneur possède sa propre vue du système d’exploitation, ses processus, son système de fichiers, son Registre et ses adresses IP.  

**Qu’est-ce qu’un conteneur Hyper-V ?**

Vous pouvez considérer un conteneur Hyper-V comme un conteneur Windows Server exécuté dans une partition Hyper-V.

Les conteneurs Hyper-V présentent une option de déploiement supplémentaire entre le conteneur Windows Server haute densité très efficace et la machine virtuelle Hyper-V hautement isolée avec virtualisation matérielle. Pour les environnements avec des applications de différentes délimitations d’approbation sur le même hôte, une isolation supplémentaire peut être nécessaire. Les conteneurs Hyper-V fournissent une isolation supérieure avec une virtualisation optimisée et le système d’exploitation Windows Server qui sépare les conteneurs entre eux et du système d’exploitation hôte. Les deux options de déploiement de conteneur utilisent les mêmes API de gestion, outils et formats d’images ; au moment du déploiement, les clients peuvent simplement choisir le mode de déploiement qui répond le mieux à leurs besoins.

**Quelle est la différence entre les conteneurs Linux et Windows Server ?**

Les conteneurs Linux et Windows Server sont semblables : ils implémentent tous deux des technologies similaires au sein de leurs noyau et système d’exploitation principal. La différence provient de la plateforme et des charges de travail qui s’exécutent dans les conteneurs.  
Quand un client utilise des conteneurs Windows Server, ils peuvent être intégrés aux technologies Windows existantes, comme .NET, ASP.NET, PowerShell et bien plus encore.

**En tant que développeur, dois-je réécrire mon application pour chaque type de conteneur ?**

Non, les images des conteneurs Windows sont communes aux conteneurs Windows Server et conteneurs Hyper-V. Le choix du type de conteneur est effectué quand vous démarrez le conteneur. Du point de vue d’un développeur, les conteneurs Windows Server et les conteneurs Hyper-V sont deux versions du même élément.  Ils offrent la même expérience de développement, de programmation et de gestion, sont ouverts et extensibles, et présentent le même niveau d’intégration et de prise en charge via Docker.

Un développeur peut créer une image de conteneur à l’aide d’un conteneur Windows Server et la déployer en tant que conteneur Hyper-V ou inversement sans modification autre que la spécification de l’indicateur d’exécution approprié.

Les conteneurs Windows Server fournissent une densité et des performances supérieures (par exemple, une durée de rotation inférieure, des performances d’exécution plus rapides par rapport aux configurations imbriquées) quand la vitesse est la clé. Les conteneurs Hyper-V offrent une isolation supérieure en garantissant que le code s’exécutant dans un conteneur ne peut pas compromettre ni affecter le système d’exploitation hôte ni d’autres conteneurs exécutés sur le même hôte. Cela est utile pour les scénarios avec plusieurs clients (où l’hébergement de code non fiable est requis), y compris les applications SaaS et l’hébergement d’ordinateurs.

**Est-ce que les conteneurs Hyper-V/Windows Server représentent un module complémentaire ou seront-ils intégrés à Windows Server ?**

Les fonctionnalités de conteneur seront intégrées à Windows Server 2016. Tenez-vous informé pour plus d’informations concernant la disponibilité générale.  

**Quelle est la relation entre les conteneurs Windows Server et Drawbridge ?**

Drawbridge est un des nombreux projets de recherche qui nous ont permis d’acquérir une bonne connaissance des conteneurs.  La majeure partie de la technologie des conteneurs dans Windows Server 2016 est née de notre expérience avec Drawbridge : nous sommes maintenant heureux d’apporter des technologies de conteneur à nos clients dans Windows Server 2016 à l’échelle internationale.

**Quelles sont les conditions préalables pour les conteneurs Windows Server et Hyper-V ?**

Les conteneurs Windows Server et Hyper-V requièrent tous deux Windows Server 2016. Ces technologies ne fonctionnent pas avec les versions précédentes de Windows.


## Gestion des conteneurs Windows

**Les conteneurs Hyper-V seront-ils également disponibles pour l’écosystème Docker ?**

Oui : les conteneurs Hyper-V fournissent le même niveau d’intégration et de gestion avec Docker que les conteneurs Windows Server.  L’objectif est d’avoir une expérience ouverte, cohérente et multiplateforme.  
La plateforme Docker va aussi considérablement simplifier et améliorer l’utilisation de toutes les options de conteneur. Une application développée à l’aide de conteneurs Windows Server peut être déployée comme un conteneur Hyper-V sans modification.


## Écosystème ouvert de Microsoft

**Est-ce que Microsoft participe à l’Open Container Initiative (OCI) ?**

Pour garantir que le format des packages reste universel, Docker a récemment organisé l’Open Container Initiative (OCI) visant à garantir que le package de conteneur conserve un format ouvert respectant les fondements, avec Microsoft comme l’un des membres fondateurs.

**Microsoft est-il vraiment associé à Docker ?**

Oui.  
Notre association à Docker permet aux développeurs de créer, gérer et déployer des conteneurs Windows Server et Linux à l’aide du même ensemble d’outils Docker. Les développeurs ciblant Windows Server n’ont plus à choisir entre la large gamme de technologies Windows Server et la création des applications en conteneur.  

Docker présente deux facettes, le groupe open source de projets et Docker la société. Nous partons du principe que nous sommes associés aux deux. La réussite de Docker s’explique en partie par l’écosystème dynamique développé autour de la technologie de conteneur Docker. Microsoft contribue au Projet Docker, en permettant la prise en charge des conteneurs Windows Server et Hyper-V.  

Pour plus d’informations, consultez le billet de blog [New Windows Server containers and Azure support for Docker](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/?WT.mc_id=Blog_ServerCloud_Announce_TTD).



<!--HONumber=Oct16_HO4-->



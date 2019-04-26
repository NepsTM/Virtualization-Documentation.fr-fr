---
title: FAQ sur les conteneurs Windows
description: FAQ sur les conteneurs Windows Server
keywords: docker, conteneurs
author: PatrickLang
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 69783f0fc3dcc80eb9614031dc6c9b2c35eeefd1
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9577140"
---
# <a name="frequently-asked-questions"></a>Forum aux questions

## <a name="general"></a>Général

### <a name="what-is-wcow-what-is-lcow"></a>Qu’est WCOW? Qu’est LCOW?

WCOW est que l’abréviation de conteneurs Windows sur Windows et LCOW est l’abréviation de conteneurs Linux sur Windows.

### <a name="what-is-the-difference-between-linux-and-windows-server-containers"></a>Quelle est la différence entre les conteneurs Linux et Windows Server?

Conteneurs Linux et Windows Server sont similaires dans la mesure où ils sont tous deux mettre en œuvre des technologies similaires au sein de leur système d’exploitation du noyau et standard. La différence provient de la plateforme et des charges de travail qui s’exécutent dans les conteneurs.  

Lorsqu’un client utilise des conteneurs Windows Server, ils peuvent intégrer avec Windows existantes technologies telles que .NET, ASP.NET, PowerShell et bien plus encore.

### <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>En tant que développeur, dois-je réécrire mon application pour chaque type de conteneur?

Non. Images de conteneur Windows sont communes aux conteneurs Windows Server et l’isolation Hyper-V. Le choix du type de conteneur est effectué quand vous démarrez le conteneur. À partir d’un point de vue du développeur, l’isolation de conteneurs Windows Server et Hyper-V sont deux versions du même élément. Ils offrent la même expérience de développement, de programmation et de gestion, sont ouverts et extensibles et présentent le même niveau d’intégration et de prise en charge avec Docker.

Un développeur peut créer une image de conteneur à l’aide d’un conteneur Windows Server et déployez-le dans l’isolation Hyper-V ou inversement sans modification autre que la spécification de l’indicateur d’exécution approprié.

Les conteneurs Windows Server offrent une densité et des performances pour les lorsque la vitesse est la clé, par exemple, de rotation inférieure temps et les performances d’exécution plus rapides par rapport aux configurations imbriquées. Isolation Hyper-V, son nom, offre une isolation supérieure s’assurer que le code s’exécutant dans un conteneur ne peut pas compromettre ou avoir un impact sur le système d’exploitation hôte ou autres conteneurs exécutés sur le même hôte. Cela est utile pour les scénarios mutualisées avec la configuration requise pour l’hébergement de code non fiable, y compris les applications SaaS et l’hébergement d’ordinateurs.

### <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Quelles sont les conditions requises pour les conteneurs en cours d’exécution sur Windows?

Les conteneurs ont été introduites pour la plateforme avec Windows Server 2016. Pour utiliser les conteneurs, vous aurez besoin de Windows Server 2016 ou Windows 10 Anniversary update (version 1607) ou une version ultérieure.

### <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>Puis-je exécuter les conteneurs Windows en mode isolées du processus sur Windows 10 entreprise ou Professionnel?

Mise à jour, nous n’est plus à partir de Windows 10 octobre 2018 interdisez à un utilisateur de s’exécuter un conteneur Windows avec l’isolation de processus. Toutefois, vous devez demander directement pour l’isolation des processus à l’aide de la `--isolation=process` indicateur lors de l’exécution de vos conteneurs via `docker run`.

Si c’est un élément que vous intéresse, vous devez vous assurer que votre ordinateur hôte est en cours d’exécution Windows 10, build 17763 + et vous disposez d’une version de docker avec le moteur 18.09 ou plus récente.

> [!WARNING]
> Cette fonctionnalité sert uniquement pour le test/développement. Vous devez continuer à utiliser Windows Server que l’hôte pour les déploiements de production.
>
> À l’aide de cette fonctionnalité, vous devez également vous assurer que les balises de version de votre hôte et conteneur correspondent, dans le cas contraire, le conteneur peut échouer à démarrer ou peuvent présenter un comportement indéfini.

## <a name="windows-container-management"></a>Gestion des conteneurs Windows

### <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>Comment faire Mes images de conteneur disponibles sur les ordinateurs exploitant?

Les images de base de conteneur Windows contiennent des artefacts dont la distribution est limitée par la licence. Lorsque vous générez sur ces images et les distribuer à un registre public ou privé, vous remarquerez que la couche de base est transmise jamais. Au lieu de cela, nous utilisons le concept d’une couche étrangère qui pointe vers la couche de base réel résidant dans un stockage cloud Azure.

Cela peut poser un problème lorsque vous disposez d’un ordinateur exploitant qui peut extraire uniquement des images à partir de l’adresse de votre Registre de conteneur privé. Les tentatives de suivre la couche externe pour obtenir l’image de base échoue dans ce cas. Pour remplacer le comportement de la couche étrangère, vous pouvez utiliser la `--allow-nondistributable-artifacts` indicateur dans le démon Docker.

> [!IMPORTANT]
> L’utilisation de cet indicateur ne fait pas obstacle votre obligation de respecter les conditions de la licence image de base du conteneur Windows; Vous devez valider pas de contenu Windows de redistribution public ou tiers. L’utilisation au sein de votre environnement est autorisée.

## <a name="microsofts-open-ecosystem"></a>Écosystème ouvert de Microsoft

### <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>Est-ce que Microsoft participe à l’Open Container Initiative (OCI)?

Pour garantir que le format des packages reste universel, Docker a récemment organisé l’Open Container Initiative (OCI) visant à garantir que le package de conteneur conserve un format ouvert respectant les fondements, avec Microsoft comme l’un des membres fondateurs.

> [!TIP]
> Vous avez une recommandation pour un ajout au Forum aux questions? Ouvrir un nouveau problème de commentaires dans la section commentaires ou GitHub permet d’ouvrir une requête de tirage contre ces documents!

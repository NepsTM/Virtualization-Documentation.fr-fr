---
title: FAQ sur les conteneurs Windows
description: FAQ sur les conteneurs Windows Server
keywords: docker, conteneurs
author: PatrickLang
ms.date: 10/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 405b2abc43a4ae2c546de351679deb755e4a9317
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910799"
---
# <a name="frequently-asked-questions-about-containers"></a>Forum aux questions sur les conteneurs

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Quelle est la différence entre les conteneurs Linux et Windows Server ?

Linux et Windows Server implémentent tous deux des technologies similaires au sein de leurs systèmes d’exploitation noyau et principal. La différence provient de la plateforme et des charges de travail qui s’exécutent dans les conteneurs.  

Lorsqu’un client utilise des conteneurs Windows Server, il peut s’intégrer à des technologies Windows existantes, telles que .NET, ASP.NET et PowerShell.

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Quelles sont les conditions préalables à l’exécution des conteneurs sur Windows ?

Les conteneurs ont été introduits sur la plateforme avec Windows Server 2016. Pour utiliser des conteneurs, vous avez besoin de Windows Server 2016 ou de la mise à jour anniversaire Windows 10 (version 1607) ou plus récente. Pour en savoir plus, consultez la [Configuration système requise](../deploy-containers/system-requirements.md) .

## <a name="what-are-wcow-and-lcow"></a>Qu’est-ce que WCOW et LCOW ?

WCOW est short pour « conteneurs Windows sur Windows ». LCOW est short pour les « conteneurs Linux sur Windows ».

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>Comment les conteneurs sont-ils sous licence ? Le nombre de conteneurs que je peux exécuter est-il limité ?

Le [CLUF](../images-eula.md) de l’image de conteneur Windows décrit une utilisation qui dépend d’un utilisateur disposant d’un système d’exploitation hôte sous licence valide. Le nombre de conteneurs qu’un utilisateur est autorisé à exécuter dépend de l’édition du système d’exploitation hôte et du [mode d’isolation](../manage-containers/hyperv-container.md) avec lequel un conteneur est exécuté, ainsi que de l’exécution de ces conteneurs à des fins de développement/test ou en production.

|Système d’exploitation hôte                                                         |Limite de conteneur isolée dans les processus                   |Limite de conteneur Hyper-V-isolé                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |Illimité                                          |2                                                  |
|Windows Server Datacenter                                       |Illimité                                          |Illimité                                          |
|Windows 10 professionnel et entreprise                                   |Illimité *(à des fins de test ou de développement uniquement)*|Illimité *(à des fins de test ou de développement uniquement)*|
|Windows 10 IoT Core et Enterprise                             |Quantité                                         |Quantité                                          |

L’utilisation des images de conteneur Windows Server est déterminée en lisant le nombre d’invités de virtualisation pris en charge pour cette [édition](/windows-server/get-started-19/editions-comparison-19.md). <br/>

>[!NOTE]
>\*l’utilisation de la production de conteneurs sur une édition IoT de Windows dépend de si vous avez convenu des conditions d’utilisation commerciales Microsoft pour les images du runtime Windows 10 Core ou la licence d’appareil Windows 10 IoT Enterprise (« contrat commercial Windows IoT »). Des termes et restrictions supplémentaires dans les contrats commerciaux Windows IoT s’appliquent à votre utilisation de l’image conteneur dans un environnement de production. Veuillez lire le [CLUF de l’image conteneur](../images-eula.md) pour comprendre exactement ce qui est autorisé et ce qui ne l’est pas.

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>En tant que développeur, dois-je réécrire mon application pour chaque type de conteneur ?

Non. Les images de conteneur Windows sont communes aux conteneurs Windows Server et à l’isolation Hyper-V. Le choix du type de conteneur est effectué quand vous démarrez le conteneur. Du point de vue du développeur, les conteneurs Windows Server et l’isolation Hyper-V sont deux versions du même élément. Ils offrent la même expérience de développement, de programmation et de gestion, et sont ouverts et extensibles, et incluent le même niveau d’intégration et de prise en charge de l’arrimeur.

Un développeur peut créer une image de conteneur à l’aide d’un conteneur Windows Server et la déployer dans l’isolation Hyper-V ou inversement sans modification autre que la spécification de l’indicateur d’exécution approprié.

Les conteneurs Windows Server offrent une plus grande densité et de meilleures performances quand la vitesse est importante, par exemple une réduction du temps de rotation et des performances d’exécution plus rapides par rapport aux configurations imbriquées. L’isolation Hyper-V, true pour son nom, offre une isolation plus importante, en garantissant que le code s’exécutant dans un conteneur ne peut pas compromettre ni affecter le système d’exploitation hôte ou d’autres conteneurs exécutés sur le même hôte. Cela est utile pour les scénarios multi-locataires avec des exigences pour l’hébergement de code non fiable, y compris les applications SaaS et l’hébergement de calcul.

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10"></a>Puis-je exécuter des conteneurs Windows en mode isolé du processus sur Windows 10 ?

À compter de la mise à jour 2018 de Windows 10 octobre, vous pouvez exécuter un conteneur Windows avec l’isolation des processus, mais vous devez d’abord demander l’isolation du processus à l’aide de l’indicateur `--isolation=process` lors de l’exécution de vos conteneurs avec `docker run`. L’isolation des processus est compatible sur Windows 10 professionnel, Windows 10 entreprise, Windows 10 IoT Core et Windows 10 IoT Enterprise.

Si vous souhaitez exécuter vos conteneurs Windows de cette manière, vous devez vous assurer que votre ordinateur hôte exécute Windows 10 Build 17763 + et que vous disposez d’une version de l’arrimeur avec le moteur 18,09 ou une version ultérieure.

> [!WARNING]
> Hormis sur les hôtes IoT Core et IoT Enterprise (après avoir accepté des conditions générales supplémentaires), cette fonctionnalité est uniquement destinée au développement et aux tests. Vous devez continuer à utiliser Windows Server comme hôte pour les déploiements de production. En utilisant cette fonctionnalité, vous devez également vous assurer que les balises de version de l’hôte et du conteneur correspondent. sinon, le conteneur peut ne pas démarrer ou présenter un comportement indéfini.

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>Comment faire rendre mes images de conteneur disponibles sur les machines à airer ?

Les images de base de conteneur Windows contiennent des artefacts dont la distribution est limitée par la licence. Lorsque vous créez sur ces images et les transmettent à un registre public ou privé, vous remarquerez que la couche de base n’est jamais envoyée. Au lieu de cela, nous utilisons le concept de couche étrangère qui pointe vers la couche de base réelle résidant dans le stockage cloud Azure.

Cela peut compliquer les choses lorsque vous avez une machine de type « air » qui ne peut extraire que des images à partir de l’adresse de votre registre de conteneurs privé. Dans ce cas, les tentatives de suivi de la couche étrangère pour récupérer l’image de base ne fonctionneront pas. Pour remplacer le comportement de la couche étrangère, vous pouvez utiliser l’indicateur `--allow-nondistributable-artifacts` dans le démon de l’ancrage.

> [!IMPORTANT]
> L’utilisation de cet indicateur n’exclut pas votre obligation de se conformer aux termes de la licence d’image de base du conteneur Windows ; vous ne devez pas poster un contenu Windows pour une redistribution publique ou tierce. L’utilisation dans votre propre environnement est autorisée.

## <a name="additional-feedback"></a>Commentaires supplémentaires

Vous souhaitez ajouter des éléments au Forum aux questions ? Ouvrez un nouveau problème de commentaires dans la section commentaires ou configurez une demande de tirage (pull request) pour cette page avec GitHub.

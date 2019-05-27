---
title: À propos des conteneurs Windows
description: En savoir plus sur les conteneurs Windows.
keywords: docker, conteneurs
author: taylorb-microsoft
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 80514884b4c95657f63cf585ece6aa8c8b23cc44
ms.sourcegitcommit: daf1d2b5879c382404fc4d59f1c35c88650e20f7
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/23/2019
ms.locfileid: "9674724"
---
# <a name="about-windows-containers"></a>À propos des conteneurs Windows

Prenons l’exemple d’une cuisine. Tout ce dont vous avez besoin au sein de la même pièce, c’est tout ce dont vous avez besoin pour cuisiner une farine: four, panoramique, lavabo, etc. Voici notre conteneur.

![Illustration d’une cuisine entièrement apportée avec un papier peint jaune dans un carré noir.](media/box1.png)

Imaginez à présent que vous avez créé une cuisine dans un bâtiment aussi facilement que de faire glisser un livre dans une bibliothèque virtuelle. Dans la mesure où tout ce dont vous avez besoin pour la cuisine est déjà là, tout ce dont nous avons besoin pour commencer à cuire consiste à relier l’électricité et la plomberie.

![Bâtiment d’une cloison constituée de deux piles de zones noires. Quatre de ces cases sont les mêmes que celles utilisées dans l’exemple de cuisine et sont dans des endroits aléatoires tout au long du bâtiment, tandis que les autres sont des salles de vie multicolores ou sont vides et grisées.](media/apartment.png)

Pourquoi ne plus être là? Vous pouvez personnaliser votre bâtiment comme bon vous semble. Remplissez-la avec de nombreux types de pièces, remplissez-les à l’aide de pièces identiques ou combinez les deux.

Les conteneurs agissent comme cette salle en exécutant une application comme nous le voulons dans notre cuisine. Un conteneur place une application et tout ce dont l’application doit s’exécuter dans sa propre zone isolée. Par conséquent, l’application isolée n’a aucune connaissance d’autres applications ou processus qui existent en dehors de son conteneur. Dans la mesure où le conteneur dispose de tout ce dont l’application doit s’exécuter, le conteneur peut être déplacé en tout lieu, à l’aide des seules ressources dont son hébergement s’est tenu sans toucher aux ressources mises en service pour d’autres conteneurs.

La vidéo suivante vous informera plus sur ce que les conteneurs Windows peuvent faire pour vous, ainsi que sur la façon dont Microsoft s’est associé à l’amarrage pour créer un environnement sans friction pour le développement de conteneurs Open Source:

<iframe width="800" height="450" src="https://www.youtube.com/embed/Ryx3o0rD5lY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## <a name="container-fundamentals"></a>Notions de base des conteneurs

Voici quelques termes utiles lorsque vous commencez à travailler avec des conteneurs Windows:.

- Hôte de conteneur: système d’ordinateur physique ou virtuel configuré à l’aide de la fonctionnalité conteneur Windows. L’hôte de conteneur exécute un ou plusieurs conteneurs Windows.
- Sandbox: couche qui capture toutes les modifications que vous apportez au conteneur lors de son exécution (par exemple, modifications du système de fichiers, modifications du registre ou installations logicielles).
- Image de base: première couche dans les couches d’image d’un conteneur qui fournit l’environnement du système d’exploitation du conteneur. Une image de base ne peut pas être modifiée.
- Image de conteneur: modèle en lecture seule d’instructions pour la création d’un conteneur. Les images peuvent être basées sur un environnement de système d’exploitation basique et non modifié, mais peuvent également être créées à partir du bac à sable d’un conteneur modifié. Ces images modifiées calquent leurs modifications en haut du calque d’image de base, et ces couches peuvent être copiées puis réappliquées à d’autres images de base pour créer une nouvelle image avec ces mêmes modifications.
- Référentiel de conteneurs: référentiel local qui stocke l’image de votre conteneur et ses dépendances chaque fois que vous créez une nouvelle image. Vous pouvez réutiliser les images stockées autant de fois que vous le souhaitez sur l’hôte de conteneur. Vous pouvez également stocker les images du conteneur dans un registre public ou privé, tel qu’un hub d’ancrage, afin qu’ils puissent être utilisés dans de nombreux hôtes de conteneur différents.
- Conteneur Orchestrator: processus qui automatise et gère un grand nombre de conteneurs et la manière dont ils interagissent entre eux. Pour en savoir plus, voir [à propos des orchestraistes des conteneurs Windows](overview-container-orchestrators.md).
- Docker: processus automatisé qui empaquet et fournit des images de conteneur. Pour en savoir plus, voir [vue d’ensemble](docker-overview.md)de l’arrimateur, [moteur de l’ancrage sur Windows](../manage-docker/configure-docker-daemon.md) ou visitez le [site Web](https://www.docker.com)de l’ancrage.

![Organigramme qui montre comment créer des conteneurs. Les images d’application et de base servent à créer un bac à sable (sandbox) et une nouvelle image d’application, qui sont superposées en haut de l’image de base pour créer un nouveau conteneur.](media/containerfund.png)

Les conteneurs et machines virtuelles peuvent paraître similaires pour une personne qui connaît les machines virtuelles. Un conteneur exécute un système d’exploitation, dispose d’un système de fichiers, et est accessible via un réseau comme un système d’informatique physique ou virtuel. Ceci dit, la technologie et les concepts derrière les conteneurs sont très différents de ceux des machines virtuelles. Pour plus d’informations sur ces concepts, voir le [billet de blog](https://azure.microsoft.com/blog/containers-docker-windows-and-trends/) de Mark Russinovich qui explique les différences plus en détail.

### <a name="windows-container-types"></a>Types de conteneurs Windows

Une autre chose que vous devez savoir est qu’il existe deux types de conteneur différents, également appelés runtimes.

Les conteneurs Windows Server fournissent l’isolement de l’application par le biais de la technologie d’isolation de processus et d’espaces de noms, ce qui explique pourquoi ces conteneurs sont également désignés sous le nom de conteneurs isolés de processus. Un conteneur Windows Server partage un noyau avec l’hôte de conteneur et tous les conteneurs exécutés sur l’hôte. Ces conteneurs isolés du processus ne fournissent pas de limite de sécurité hostile et ne doivent pas être utilisés pour isoler du code non fiable. En raison de l’espace de noyau partagé, ces conteneurs requièrent la même version et configuration de noyau.

L’isolement Hyper-V s’étend sur l’isolement fourni par les conteneurs Windows Server en exécutant chaque conteneur dans une machine virtuelle hautement optimisée. Dans cette configuration, l’hôte de conteneur ne partage pas son noyau avec d’autres conteneurs sur le même hôte. Ces conteneurs sont conçus pour un hébergement mutualisé hostile avec les mêmes garanties de sécurité qu’une machine virtuelle. Dans la mesure où ces conteneurs ne partagent pas le noyau avec l’hôte ou d’autres conteneurs sur l’hôte, ils peuvent exécuter des noyaux avec différentes versions et configurations (dans les versions prises en charge). Par exemple, tous les conteneurs Windows sur Windows 10 utilisent l’isolation Hyper-V pour utiliser la version et la configuration du noyau Windows Server.

L’exécution d’un conteneur sur Windows avec ou sans l’isolation Hyper-V est une décision d’exécution. Vous pouvez commencer par créer le conteneur avec l’isolation Hyper-V, puis ultérieurement lors de l’exécution, choisir de l’exécuter en tant que conteneur Windows Server.

## <a name="container-users"></a>Utilisateurs de conteneurs

### <a name="containers-for-developers"></a>Conteneurs pour les développeurs

Les conteneurs permettent aux développeurs de créer et d’expédier des applications de meilleure qualité plus rapidement. Les développeurs peuvent créer une image d’amarrage qui s’déploiera de manière identique sur tous les environnements en quelques secondes. Il y a un écosystème énorme et croissant d’applications conditionnées dans des conteneurs d’amarrage. DockerHub, un registre de l’application publique géré par l’arrimeur, a publié plus de 180 000 applications dans son référentiel de la communauté publique et ce numéro reste en augmentation.

Lorsqu’un développeur containerizes une application, seule l’application et les composants qu’elle doit exécuter sont combinés dans une image. Les conteneurs sont ensuite créés à partir de cette image si vous en avez besoin. Vous pouvez également utiliser une image comme référence pour créer une autre image, ce qui accélère encore davantage la création d’images. Les conteneurs multiples peuvent partager la même image, ce qui signifie que les conteneurs démarrent très rapidement et utilisent moins de ressources. Par exemple, un développeur peut utiliser des conteneurs pour faire tourner les composants d’application léger et léger, également appelés microservices, pour les applications distribuées et mettre rapidement à l’échelle chaque service séparément.

Les conteneurs sont portables et polyvalents, peuvent être rédigés dans n’importe quelle langue et sont compatibles avec les ordinateurs exécutant Windows Server 2016. Les développeurs peuvent créer et tester un conteneur localement sur leur ordinateur portable ou de bureau, puis déployer ce même type d’image sur le cloud privé de leur entreprise, le cloud public ou le fournisseur de services. La souplesse naturelle des conteneurs prend en charge les modèles de développement d’applications modernes dans les environnements Cloud virtualisés à grande échelle.

### <a name="containers-for-it-professionals"></a>Conteneurs pour les professionnels de l’informatique

Les conteneurs permettent aux administrateurs de créer une infrastructure plus facile à mettre à jour et à mettre à jour. Les professionnels de l’informatique peuvent utiliser des conteneurs pour fournir des environnements normalisés pour leurs équipes de développement, de QA et de production. Ils n’ont plus à se soucier des procédures d’installation et de configuration complexes. L’utilisation de conteneurs permet aux administrateurs de systèmes de soustraire les différences dans les installations du système d’exploitation et dans l’infrastructure sous-jacente.

## <a name="containers-101-video-presentation"></a>Présentation vidéo de conteneurs 101

La présentation vidéo suivante vous offre une vue d’ensemble approfondie de l’historique et de l’implémentation des conteneurs Windows.

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## <a name="try-windows-server-containers"></a>Essayer les conteneurs Windows Server

Vous êtes prêt à exploiter les impressionnantes fonctionnalités des conteneurs? Les articles suivants vous aideront à commencer:

Pour configurer un conteneur sur Windows Server, voir [démarrage rapide de Windows Server](../quick-start/quick-start-windows-server.md).

Pour configurer un conteneur sur Windows 10, voir l’aide au [démarrage de Windows 10](../quick-start/quick-start-windows-10.md).
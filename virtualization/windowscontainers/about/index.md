---
title: À propos des conteneurs Windows
description: Les conteneurs constituent une technologie de création de packages et d’exécution d’applications, y compris des applications Windows, dans différents environnements locaux et dans le Cloud. Cette rubrique vous explique comment Microsoft, Windows et Azure vous permettent de développer et de déployer des applications dans des conteneurs, notamment à l’aide de docker et d’Azure Kubernetes service.
keywords: docker, conteneurs
author: taylorb-microsoft
ms.date: 10/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: acce214cc8991f20c979b6dbe636590416841cb9
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254289"
---
# <a name="windows-and-containers"></a>Windows et conteneurs

Les conteneurs constituent une technologie permettant d’empaqueter et d’exécuter des applications Windows et Linux dans divers environnements locaux et dans le Cloud. Les conteneurs fournissent un environnement léger et isolé qui simplifie le développement, le déploiement et la gestion des applications. Les conteneurs débutent et arrêtent rapidement, ce qui les rend idéaux pour les applications qui doivent s’adapter rapidement à la modification de la demande. Le caractère léger des conteneurs rend également un outil utile pour améliorer la densité et l’utilisation de votre infrastructure.

![Graphique illustrant la façon dont les conteneurs peuvent s’exécuter dans le Cloud ou sur site, prenant en charge des applications ou des microservices monolithiques écrits en presque n’importe quelle langue.](media/about-3-box.png)

## <a name="the-microsoft-container-ecosystem"></a>Écosystème de conteneurs Microsoft

Microsoft fournit un certain nombre d’outils et de plateformes pour vous aider à développer et déployer des applications dans des conteneurs:

- <strong>Exécutez des conteneurs Windows ou Linux sur Windows 10 pour le</strong> développement et les tests à l’aide de la version de bureau de l' [amarrage](https://store.docker.com/editions/community/docker-ce-desktop-windows), qui utilise la fonctionnalité conteneurs intégrée à Windows. Vous pouvez également [exécuter des conteneurs en mode natif sur Windows Server](../quick-start/set-up-environment.md?tabs=Windows-Server).
- <strong>Développez, testez, publiez et déployez des conteneurs Windows</strong> à l’aide de la [puissante prise en charge des conteneurs dans Visual Studio](https://docs.microsoft.com/visualstudio/containers/overview) et [Visual Studio](https://code.visualstudio.com/docs/azure/docker), qui incluent la prise en charge de l’arrimeur, de la composition du Dock, de Kubernetes, de Helm et d’autres fonctionnalités utiles. disponibles.
- <strong>Publiez vos applications en tant qu’images de conteneurs</strong> sur le DockerHub public pour permettre à d’autres personnes de les utiliser, ou à un [Registre de conteneur Azure](https://azure.microsoft.com/services/container-registry/) privé pour le développement et le déploiement de votre organisation, en envoyant et en tirant directement à partir de Visual Studio et du code Visual Studio .
- <strong>Déployer des conteneurs à l’échelle sur Azure ou sur d'</strong> autres nuages:

  - Tirez votre application (image de conteneur) à partir d’un registre de conteneurs, tel que le registre de conteneur Azure, puis déployez et gérez-la à l’aide d’un système d’évaluation tel qu’Azure [Kubernetes service (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes) (en Preview pour les applications Windows) ou [Azure service. Fabrics](https://docs.microsoft.com/azure/service-fabric/).
  - Le service Azure Kubernetes déploie des conteneurs sur les machines virtuelles Azure, et les gère à l’échelle, qu’il s’agisse de des conteneurs, des centaines ou même des milliers. Les machines virtuelles Azure exécutent une image Windows Server personnalisée (si vous déployez une application Windows) ou une image Linux Ubuntu personnalisée (si vous déployez une application basée sur Linux).
- <strong>Déployez les conteneurs en local</strong> à l’aide de [la pile Azure avec le moteur AKS](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) (en preview avec les conteneurs Linux) ou [Azure Stack with OpenShift](https://docs.microsoft.com/azure/virtual-machines/linux/openshift-azure-stack). Vous pouvez également configurer Kubernetes vous-même sur Windows Server (voir [Kubernetes sur Windows](../kubernetes/getting-started-kubernetes-windows.md)), et nous travaillons à la prise en charge de l’exécution de [conteneurs Windows sur la plateforme de conteneur OpenShift RedHat](https://techcommunity.microsoft.com/t5/Networking-Blog/Managing-Windows-containers-with-Red-Hat-OpenShift-Container/ba-p/339821) .

## <a name="how-containers-work"></a>Fonctionnement des conteneurs

Un conteneur est un silo léger isolé d’exécution d’une application sur le système d’exploitation hôte. Les conteneurs se créent au-dessus du noyau du système d’exploitation hôte (qui peut être pensé comme la plomberie du système d’exploitation), comme illustré dans ce diagramme.

![Diagramme architectural montrant la façon dont les conteneurs s’exécutent au-dessus du noyau](media/container-diagram.svg)

Tant qu’un conteneur partage le noyau du système d’exploitation hôte, le conteneur ne dispose pas d’un accès sans entrave. Au lieu de cela, le conteneur obtient un isolement, et dans certains cas, un affichage du système. Par exemple, un conteneur peut accéder à une version virtualisée du système de fichiers et du Registre, mais les modifications affectent uniquement le conteneur et sont supprimées lorsqu’il s’arrête. Pour enregistrer des données, le conteneur peut monter un stockage persistant tel qu’un [disque Azure](https://azure.microsoft.com/services/storage/disks/) ou un partage de fichiers (y compris les [fichiers Azure](https://azure.microsoft.com/services/storage/files/)).

Un conteneur repose sur le noyau, mais le noyau ne fournit pas toutes les API et services qu’une application doit exécuter: la plupart de ces fonctions sont fournies par des fichiers système (bibliothèques) qui s’exécutent au-dessus du noyau dans le mode utilisateur. Dans la mesure où un conteneur est isolé de l’environnement du mode utilisateur de l’hôte, il a besoin de sa propre copie de ces fichiers système en mode utilisateur, qui sont empaquetés dans une image de base. L’image de base sert de calque de base sur lequel votre conteneur est créé, et fournit des services de système d’exploitation non fournis par le noyau. Néanmoins, nous vous parlerons plus tard.

## <a name="containers-vs-virtual-machines"></a>Conteneurs et machines virtuelles

Contrairement à un conteneur, une machine virtuelle (VM) exécute un système d’exploitation complet, y compris son propre noyau, comme le montre ce diagramme.

![Diagramme architectural montrant comment les ordinateurs virtuels exécutent un système d’exploitation complet en regard du système d’exploitation hôte](media/virtual-machine-diagram.svg)

Les conteneurs et machines virtuelles ont chacun de leurs usages: en fait, de nombreux déploiements de conteneurs utilisent des machines virtuelles comme système d’exploitation hôte plutôt que de s’exécuter directement sur le matériel, en particulier lors de l’exécution de conteneurs dans le Cloud.

Pour plus d’informations sur les similitudes et les différences entre ces technologies complémentaires, voir [conteneurs et machines virtuelles](containers-vs-vm.md).

## <a name="container-images"></a>Images de conteneurs

Tous les conteneurs sont créés à partir d’images de conteneurs. Les images de conteneur sont un ensemble de fichiers organisés en une pile de couches se trouvant sur votre ordinateur local ou dans un registre de conteneur distant. L’image du conteneur se compose des fichiers du système d’exploitation du mode utilisateur nécessaires à la prise en charge de votre application, de votre application, de tout Runtime ou dépendance de votre application, ainsi que de tout autre fichier de configuration divers dont votre application a besoin pour fonctionner correctement.

Microsoft propose plusieurs images (appelées images de base) que vous pouvez utiliser comme point de départ pour créer votre propre image de conteneur:

* <strong>Windows</strong> : contient l’ensemble complet d’API Windows et de services système (moins de rôles de serveur).
* <strong>Windows Server Core</strong> : image plus petite contenant un sous-ensemble des API du serveur Windows, à savoir le .NET Framework complet. Il inclut également la plupart des rôles de serveur, même s’il s’agit de personnes qui ne sont pas des serveurs de télécopie.
* <strong>Nano Server</strong> : l’image la plus petite du serveur Windows, avec la prise en charge des API .net Core et de certains rôles de serveur.
* <strong>Windows 10 IOT</strong> standard-version de Windows utilisée par des fabricants de matériel informatique pour les appareils mobiles qui utilisent des processeurs ARM ou x86/x64.

Comme indiqué précédemment, les images de conteneur sont composées d’une série de couches. Chaque couche contient un ensemble de fichiers qui, lors de leur suivi, représentent l’image de votre conteneur. En raison de la nature stratifiée des conteneurs, vous ne devez pas nécessairement cibler une image de base pour créer un conteneur Windows. Au lieu de cela, vous pouvez cibler une autre image qui porte déjà l’infrastructure de votre choix. Par exemple, l’équipe .NET publie une [image .net Core](https://hub.docker.com/_/microsoft-dotnet-core) qui comporte .net Core Runtime. Il évite aux utilisateurs de devoir dupliquer le processus d’installation de .NET Core: il peut réutiliser les couches de cette image de conteneur. L’image principale de .NET est basée sur nano Server.

Pour plus d’informations, consultez [images de base du conteneur](../manage-containers/container-base-images.md).

## <a name="container-users"></a>Utilisateurs de conteneurs

### <a name="containers-for-developers"></a>Conteneurs pour les développeurs

Les conteneurs permettent aux développeurs de créer et d’expédier des applications de meilleure qualité plus rapidement. Les conteneurs permettent aux développeurs de créer une image de conteneur qui s’déploie en quelques secondes, de la même façon que dans les environnements. Les conteneurs permettent de partager du code au sein des équipes et d’amorcer un environnement de développement sans influer sur le système de fichiers de votre hôte.

Les conteneurs sont portables et polyvalents, peuvent exécuter des applications écrites dans n’importe quel langage, et sont compatibles avec les ordinateurs exécutant Windows 10, version 1607 ou ultérieure, ou Windows 2016 ou une version ultérieure. Les développeurs peuvent créer et tester un conteneur localement sur leur ordinateur portable ou de bureau, puis déployer ce même type d’image dans le cloud privé de leur entreprise, le cloud public ou le fournisseur de services. La souplesse naturelle des conteneurs prend en charge les modèles de développement d’applications modernes dans les environnements Cloud virtualisés à grande échelle.

### <a name="containers-for-it-professionals"></a>Conteneurs pour les professionnels de l’informatique

Les conteneurs permettent aux administrateurs de créer une infrastructure plus facile à mettre à jour et à mettre à jour, et ce qui utilise davantage de ressources matérielles. Les professionnels de l’informatique peuvent utiliser des conteneurs pour fournir des environnements normalisés pour leurs équipes de développement, de QA et de production. L’utilisation de conteneurs permet aux administrateurs de systèmes de soustraire les différences dans les installations du système d’exploitation et dans l’infrastructure sous-jacente.

## <a name="container-orchestration"></a>Orchestration de conteneur

Les orchestreurs constituent un élément essentiel d’infrastructure lors de la configuration d’un environnement basé sur un conteneur. Même si vous pouvez gérer quelques conteneurs manuellement à l’aide de l’arrimeur et de Windows, les applications utilisent souvent cinq, dix ou même des centaines de conteneurs, qui sont les uns des autres.

Les administrateurs de conteneurs ont été conçus pour aider à gérer les conteneurs à l’échelle et en production. Les orchestrarons fournissent des fonctionnalités pour:

- Déploiement à l’échelle
- Planification de charge de travail
- Analyse du fonctionnement
- Basculement en cas d’échec d’un nœud
- Mise à l’échelle vers le haut ou vers le bas
- Réseaux
- Découverte de service
- Coordination des mises à niveau d’applications
- Affinité de nœud de cluster

Il existe de nombreux orchestrauses que vous pouvez utiliser avec des conteneurs Windows. Voici les options proposées par Microsoft:
- [Service Azure Kubernetes (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes) -utilisez un service Azure Kubernetes géré
- [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/) -utiliser un service géré
- [Pile Azure avec le moteur AKS](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) -utilisez le service Azure Kubernetes en local
- [Kubernetes sur Windows](../kubernetes/getting-started-kubernetes-windows.md) -configurez vous-même Kubernetes sur Windows

## <a name="try-containers-on-windows"></a>Utiliser des conteneurs sur Windows

Pour commencer à utiliser les conteneurs sur Windows Server ou Windows 10, voir les rubriques suivantes:
> [!div class="nextstepaction"]
> [Commencer: configurer votre environnement pour les conteneurs](../quick-start/set-up-environment.md)

Pour vous aider à choisir les services Azure qui conviennent le mieux à votre scénario, voir [services de conteneur Azure](https://azure.microsoft.com/product-categories/containers/) et [sélectionnez les services Azure à utiliser pour héberger votre application](https://docs.microsoft.com/azure/architecture/guide/technology-choices/compute-decision-tree).

---
title: Conteneurs Linux sur Windows
description: En savoir plus sur les différentes façons, que vous pouvez utiliser Hyper-V pour exécuter des conteneurs Linux sur Windows, comme si elles sont natives.
keywords: LCOW, des conteneurs linux, docker, conteneurs
author: scooley
ms.date: 11/02/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 7db0135e5d5079d3b8cce815d051ecd6a7cb896b
ms.sourcegitcommit: 614e3ca3e6f4373b999a501a2829adbaa61de4c4
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2018
ms.locfileid: "7277533"
---
# <a name="linux-containers-on-windows"></a>Conteneurs Linux sur Windows

Conteneurs Linux constituent un pourcentage considérable de l’écosystème de conteneurs globale et sont essentiels à la fois des expériences de développeur et des environnements de production.  Dans la mesure où les conteneurs partagent un noyau avec l’hôte de conteneur, toutefois, en cours d’exécution des conteneurs Linux directement sur Windows n’est pas une option[*](linux-containers.md#other-options-we-considered).  Il s’agit là qu’intervient la virtualisation dans l’image.

Maintenant, il existe deux façons d’exécuter des conteneurs Linux avec Docker pour Windows et Hyper-v:

1. Exécuter des conteneurs Linux dans une VM Linux complet - c’est ce que Docker en général dès aujourd'hui.
1. Exécuter des conteneurs Linux avec [Isolation Hyper-V](../manage-containers/hyperv-container.md) (LCOW) - Ceci est une nouvelle option de Docker pour Windows.

Cet article explique comment chaque approche fonctionne, fournit des indications sur quand opter pour que la solution et des partages de travail en cours.

## <a name="linux-containers-in-a-moby-vm"></a>Conteneurs Linux sur un ordinateur virtuel Moby

Pour exécuter des conteneurs Linux dans une VM de Linux, suivez les instructions du [guide de get-démarré de Docker](https://docs.docker.com/docker-for-windows/).

Docker a été en mesure d’exécuter des conteneurs Linux sur Windows desktop dans la mesure où il a été relâché tout d’abord dans 2016 (avant l’isolation Hyper-V ou LCOW étaient disponibles) à l’aide d’un [LinuxKit](https://github.com/linuxkit/linuxkit) basé machine virtuelle en cours d’exécution sur Hyper-V.

Dans ce modèle, le Client Docker s’exécute sur le bureau Windows, mais les appels dans le démon Docker sur les VM Linux.

![VM Moby en tant que l’hôte de conteneur](media/MobyVM.png)

Dans ce modèle, tous les conteneurs Linux partagent un hôte de conteneur basé sur Linux unique et tous les conteneurs Linux:

* Partagent un noyau entre eux et les VM Moby, mais pas avec l’hôte Windows.
* Disposer de stockage cohérent et mise en réseau des propriétés avec des conteneurs Linux en cours d’exécution sur Linux (dans la mesure où ils s’exécutent sur un VM Linux).

Cela signifie également que l’hôte de conteneur Linux (Moby VM) doit exécuter démon Docker et toutes les dépendances du démon Docker.

Pour voir si vous exécutez Moby VM, vérifiez le Gestionnaire Hyper-V pour les VM Moby l’IU Gestionnaire Hyper-V ou en exécutant `Get-VM` dans une fenêtre PowerShell avec élévation de privilèges.

## <a name="linux-containers-with-hyper-v-isolation"></a>Conteneurs Linux avec isolation Hyper-V

Pour essayer LCOW, suivez les instructions de conteneur de Linux dans [ce guide get-démarré](../quick-start/quick-start-windows-10.md)

Conteneurs Linux avec isolation Hyper-V s’exécutent à chaque conteneur Linux (LCOW) dans un optimisé VM Linux avec juste assez du système d’exploitation pour exécuter des conteneurs.  Contrairement à l’approche Moby VM, chaque LCOW possède son propre noyau et son propre bac à sable de la machine virtuelle.  Ils sont également gérés directement par Docker sur Windows.

![Conteneurs Linux avec isolation Hyper-V (LCOW)](media/lcow-approach.png)

Examiner de plus près à la gestion des conteneurs diffère entre l’approche de Moby VM et les LCOW, dans le LCOW gestion des conteneurs de modèle reste à Windows et chaque gestion LCOW se produit via GRPC et containerd.  Cela signifie que les conteneurs de distro Linux utilisent pour LCOW peut avoir un inventaire de plus petit quantité.  Droit à présent, nous utilisons LinuxKit pour utilisent les conteneurs distro optimisé, mais les autres projets comme Kata de génération similaire hautement optimisées Linux exactes (clair Linux) ainsi.

Voici un Examinons de près de chaque LCOW:

![Architecture LCOW](media/lcow.png)

Pour voir si vous exécutez LCOW, accédez à `C:\Program Files\Linux Containers`.  Si Docker est configuré pour utiliser LCOW, il y aura quelques fichiers ici contenant le distro LinuxKit minimal qui s’exécute dans chaque conteneur Hyper-V.  Notez que les composants d’ordinateur virtuel optimisés sont moins de 100 Mo, beaucoup plus petite que l’image de LinuxKit dans Moby VM.

### <a name="work-in-progress"></a>Travail en cours

LCOW est en cours de développement actif.  Suivre la progression en cours dans le projet Moby sur [GitHub](https://github.com/moby/moby/issues/33850)

#### <a name="bind-mounts"></a>Lier des montages

La liaison des volumes de montage avec `docker run -v ...` stocke les fichiers sur le système de fichiers NTFS de Windows. Par conséquent, une traduction est nécessaire pour les opérations POSIX. À l’heure actuelle, certaines opérations de système de fichiers ne sont que partiellement implémentées ou ne le sont pas du tout, ce qui peut entraîner des incompatibilités pour certaines applications.

Ces opérations ne fonctionnent actuellement pas pour les volumes montés par liaison:

* MkNod
* XAttrWalk
* XAttrCreate
* Lock
* Getlock
* Auth
* Flush
* INotify

Certaines ne sont pas totalement implémentés:

* GetAttr: le nombre Nlink est toujours signalé comme étant2
* Open: seuls les indicateurs ReadWrite, WriteOnly et ReadOnly sont implémentés

Ces applications requièrent tous mappage de volume et ne seront pas démarrer ou s’exécuter correctement.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>Informations supplémentaires

[Blog de docker décrivant LCOW](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Vidéo de conteneur Linux](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[Le noyau LCOW LinuxKit, ainsi que des instructions de génération](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Quand utiliser Visual Studio de Moby VM LCOW

### <a name="when-to-use-moby-vm"></a>Quand utiliser Moby VM

Droit à présent, nous vous recommandons la méthode Moby VM de l’exécution de conteneurs Linux à des personnes qui:

1. Choix d’un environnement de conteneur stable.  Il s’agit de la valeur par défaut de Docker pour Windows.
1. Exécuter des conteneurs Windows ou Linux, mais rarement les deux en même temps.
1. Sont complexes ou personnalisée mise en réseau besoins entre les conteneurs Linux.
1. N’ont pas besoin d’isolation de noyau (isolation Hyper-V) entre les conteneurs Linux.

### <a name="when-to-use-lcow"></a>Quand utiliser LCOW

Droit à présent, nous vous recommandons LCOW aux personnes qui:

1. Vous souhaitez tester les dernières technologies.
1. Exécuter des conteneurs Windows et Linux en même temps.
1. Besoin d’isolation de noyau (isolation Hyper-V) entre les conteneurs Linux.

## <a name="other-options-we-considered"></a>Autres options que nous avons considéré

Lorsque nous avons regardant façons d’exécuter des conteneurs Linux sur Windows, nous avons considéré WSL. Pour finir, nous avons choisi une approche basée sur la virtualisation afin que les conteneurs Linux sur Windows sont cohérentes avec les conteneurs Linux sur Linux. L’utilisation d’Hyper-V permet également LCOW plus sécurisée. Nous pouvons réévaluer à l’avenir, mais pour l’instant, les LCOW vont continuer à utiliser Hyper-V.

Si vous avez des idées, veuillez envoyer des commentaires par le biais de GitHub ou UserVoice.  En particulier, nous apprécions vos commentaires sur l’expérience spécifique que vous souhaitez voir.

---
title: Conteneurs Linux sur Windows
description: Découvrez les différentes façons dont vous pouvez utiliser Hyper-V pour exécuter des conteneurs Linux sur Windows comme s’ils étaient en natif.
keywords: LCOW, conteneurs Linux, ancrage, conteneurs
author: scooley
ms.date: 11/02/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 0426b14c423c06a0f12ea91529ce794f7a972f47
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998476"
---
# <a name="linux-containers-on-windows"></a>Conteneurs Linux sur Windows

Les conteneurs Linux constituent un pourcentage considérable de l’écosystème global du conteneur et sont fondamentaux pour les expériences des développeurs et les environnements de production.  Étant donné que les conteneurs partagent un noyau avec l’hôte de conteneur, il n’est pas possible[*](linux-containers.md#other-options-we-considered)d’exécuter des conteneurs Linux directement sur Windows.  C’est ici que la virtualisation est proposée dans l’image.

Il existe deux façons d’exécuter des conteneurs Linux avec le Dockr pour Windows et Hyper-V:

1. Exécutez des conteneurs Linux dans une station virtuelle Linux complète, c’est la version d’arrimeur qui est la plus courante.
1. Exécuter des conteneurs Linux avec l' [isolation Hyper-V](../manage-containers/hyperv-container.md) (LCOW)-il s’agit d’une nouvelle option dans docker pour Windows.

Cet article décrit le fonctionnement de chaque approche, fournit des recommandations sur le moment où choisir la solution et partager le travail en cours.

## <a name="linux-containers-in-a-moby-vm"></a>Conteneurs Linux dans un VM Moby

Pour exécuter les conteneurs Linux dans un VM Linux, suivez les instructions du [Guide de démarrage de l'](https://docs.docker.com/docker-for-windows/)outil de connexion.

L’arrimeur a pu exécuter des conteneurs Linux sur le bureau Windows, car il a été lancé pour la première fois dans 2016 (avant l’isolement Hyper-V ou LCOW étaient disponibles) à l’aide d’une machine virtuelle [LinuxKit](https://github.com/linuxkit/linuxkit) exécutée sur Hyper-v.

Dans ce modèle, le client d’amarrage s’exécute sur un ordinateur de bureau Windows, mais il appelle le démon de l’amarrage de l’ordinateur virtuel Linux.

![Moby VM en tant qu’hôte de conteneur](media/MobyVM.png)

Dans ce modèle, tous les conteneurs Linux partagent un hôte de conteneurs Linux unique et tous les conteneurs Linux:

* Partagez un noyau entre eux et l’ordinateur virtuel Moby, mais pas avec l’hôte Windows.
* Disposent de propriétés d’espace de stockage et de réseau cohérentes avec les conteneurs Linux s’exécutant sur Linux (dans la mesure où ils s’exécutent sur un VM Linux).

Par ailleurs, le processus de l’hôte de conteneur Linux (VM Moby) doit exécuter le démon de la station d’accueil et toutes les dépendances du démon de l’ancrage.

Pour savoir si vous utilisez la version VM de Moby, activez le Gestionnaire Hyper-V pour Moby VM à l’aide de l’interface utilisateur du gestionnaire `Get-VM` Hyper-v ou en exécutant une fenêtre PowerShell avec élévation de privilèges.

## <a name="linux-containers-with-hyper-v-isolation"></a>Conteneurs Linux avec isolation Hyper-V

Pour essayer LCOW, suivez les instructions du conteneur Linux dans [ce guide](../quick-start/quick-start-windows-10.md) de mise en route

Les conteneurs Linux disposant d’une isolation Hyper-V exécutent chaque conteneur Linux (LCOW) dans un ordinateur virtuel Linux optimisé doté d’un système d’exploitation suffisant pour exécuter des conteneurs.  Par contraste de l’approche de Moby VM, chaque LCOW dispose de son propre noyau et de son propre sandbox d’ordinateur virtuel.  Ils sont également gérés par l’arrimeur sur Windows directement.

![Conteneurs Linux avec isolation Hyper-V (LCOW)](media/lcow-approach.png)

Pour plus d’informations sur le fonctionnement de la gestion des conteneurs entre l’approche de la machine virtuelle Moby et LCOW, voir la gestion des conteneurs de modèles LCOW reste sur Windows et chaque gestion LCOW intervient via GRPC et conteneur.  Cela signifie que les conteneurs distro Linux utilisés pour LCOW peuvent présenter un inventaire plus petit.  Pour l’instant, nous utilisons LinuxKit pour l’utilisation des conteneurs distro optimisés, mais d’autres projets comme Kata développent des distros Linux similaires (en clair Linux).

Voici un exemple d’étude plus approfondie de chaque LCOW:

![Architecture LCOW](media/lcow.png)

Pour savoir si vous utilisez LCOW, accédez à `C:\Program Files\Linux Containers`. S’il est configuré pour utiliser LCOW, il y aura quelques fichiers contenant le minimum LinuxKit distro qui s’exécute dans chaque conteneur exécuté sous l’isolation Hyper-V.  Notez que les composants d’ordinateur virtuel optimisés sont inférieurs à 100 Mo, beaucoup plus petits que l’image LinuxKit dans l’ordinateur virtuel Moby.

### <a name="work-in-progress"></a>Travail en cours

LCOW est sous développement actif. Suivre l’avancement en cours dans le projet Moby sur [GitHub](https://github.com/moby/moby/issues/33850)

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

Ces applications nécessitent tout le mappage de volume et ne démarrent pas ou ne s’exécutent pas correctement.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>Informations supplémentaires

[Blog de l’amarrage décrivant LCOW](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Vidéo conteneur Linux](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-kernel plus instructions de génération](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Quand utiliser Moby VM ou LCOW

### <a name="when-to-use-moby-vm"></a>Quand utiliser Moby VM

Pour l’instant, nous vous recommandons d’utiliser la méthode VM Moby pour exécuter des conteneurs Linux aux personnes qui:

- Souhaitez un environnement de conteneur stable.  Il s’agit de l’option d’amarrage pour Windows par défaut.
- Exécutez des conteneurs Windows ou Linux, mais rarement les deux en même temps.
- Présentent des exigences de réseau complexes ou personnalisées entre les conteneurs Linux.
- Vous n’avez pas besoin d’isolement du noyau (isolation Hyper-V) entre les conteneurs Linux.

### <a name="when-to-use-lcow"></a>Utilisation de LCOW

Pour l’instant, nous vous recommandons de LCOWer aux personnes qui:

- Vous souhaitez tester notre technologie la plus récente.
- Exécutez des conteneurs Windows et Linux en même temps.
- Nécessite l’isolement du noyau (isolation Hyper-V) entre les conteneurs Linux.

## <a name="other-options-we-considered"></a>Autres options prises en considération

Pour exécuter les conteneurs Linux sur Windows, nous avons considéré WSL. Pour finir, nous avons opté pour une approche basée sur la virtualisation pour les conteneurs Linux sur Windows, avec les conteneurs Linux sur Linux. L’utilisation de Hyper-V rend LCOW plus sûr. Nous sommes susceptibles de réévaluer à l’avenir, mais pour l’instant, LCOW continuera à utiliser Hyper-V.

Si vous avez des réflexions, envoyez des commentaires via GitHub ou UserVoice.  Nous aimerions particulièrement nous faire part de vos commentaires concernant l’interface que vous aimeriez voir.

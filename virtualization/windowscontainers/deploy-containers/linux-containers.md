---
title: Conteneurs Linux sur Windows 10
description: Découvrez les différentes façons dont vous pouvez utiliser Hyper-V pour exécuter des conteneurs Linux sur Windows 10 comme s’ils étaient natifs.
keywords: LCOW, conteneurs Linux, arrimeur, conteneurs, Windows 10
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 843bd0ab7ccf3a227482ba3a3d2677e36b395b29
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78854013"
---
# <a name="linux-containers-on-windows-10"></a>Conteneurs Linux sur Windows 10

Les conteneurs Linux constituent un pourcentage énorme de l’écosystème de conteneurs global et sont fondamentaux pour les environnements de développement et les environnements de production.  Étant donné que les conteneurs partagent un noyau avec l’hôte de conteneur, toutefois, l’exécution de conteneurs Linux directement sur Windows n’est pas une option[*](linux-containers.md#other-options-we-considered).  C’est là qu’intervient la virtualisation dans l’image.

À ce stade, il existe deux façons d’exécuter des conteneurs Linux avec Docker pour Windows et Hyper-V :

- Exécuter des conteneurs Linux sur une machine virtuelle Linux complète : c’est ce que l’arrimeur effectue généralement aujourd’hui.
- Exécuter des conteneurs Linux avec l' [isolation Hyper-V](../manage-containers/hyperv-container.md) (LCOW) : il s’agit d’une nouvelle option dans docker pour Windows.

> _L’exécution de conteneurs Linux sur un système d’exploitation Windows Server est actuellement toujours à l’État expérimental. Des licences supplémentaires pour le programme Dockr EE seront nécessaires pour essayer cela. **Le reste de cet article s’applique uniquement à Windows 10**._

Cet article décrit le fonctionnement de chaque approche, fournit des conseils sur la manière de choisir la solution et les partages de travail en cours.

## <a name="linux-containers-in-a-moby-vm"></a>Conteneurs Linux dans une machine virtuelle Moby

Pour exécuter des conteneurs Linux sur une machine virtuelle Linux, suivez les instructions du [Guide de prise en main](https://docs.docker.com/docker-for-windows/)de l’ancrage.

L’arrimeur a pu exécuter des conteneurs Linux sur le bureau Windows, car il a été publié pour la première fois dans 2016 (avant que l’isolation Hyper-V ou les conteneurs Linux sur Windows étaient disponibles) à l’aide d’une machine virtuelle [LinuxKit](https://github.com/linuxkit/linuxkit) s’exécutant sur Hyper-v.

Dans ce modèle, le client docker s’exécute sur le bureau Windows, mais appelle le démon Dockr sur la machine virtuelle Linux.

![Machine virtuelle Moby en tant qu’hôte de conteneur](media/MobyVM.png)

Dans ce modèle, tous les conteneurs Linux partagent un seul hôte de conteneur basé sur Linux et tous les conteneurs Linux :

* Partager un noyau entre eux et la machine virtuelle Moby, mais pas avec l’hôte Windows.
* Utilisez des propriétés de réseau et de stockage cohérentes avec des conteneurs Linux exécutés sur Linux (car ils s’exécutent sur une machine virtuelle Linux).

Cela signifie également que l’hôte de conteneur Linux (machine virtuelle Moby) doit exécuter le démon de station d’accueil et toutes les dépendances du démon d’ancrage.

Pour voir si vous utilisez une machine virtuelle Moby, vérifiez le Gestionnaire Hyper-V pour la machine virtuelle Moby à l’aide de l’interface utilisateur du Gestionnaire Hyper-V ou en exécutant `Get-VM` dans une fenêtre PowerShell avec élévation de privilèges.

## <a name="linux-containers-with-hyper-v-isolation"></a>Conteneurs Linux avec isolation Hyper-V

Pour tester les conteneurs Linux sur Windows 10 (LCOW10), suivez les instructions du conteneur Linux dans [conteneurs Linux sur Windows 10](../quick-start/quick-start-windows-10-linux.md). 

Les conteneurs Linux avec isolation Hyper-V exécutent chaque conteneur Linux sur une machine virtuelle Linux optimisée avec juste assez de système d’exploitation pour exécuter des conteneurs. Contrairement à l’approche de machine virtuelle Moby, chaque conteneur Linux a son propre noyau et son propre bac à sable (sandbox) de machine virtuelle. Elles sont également gérées directement par l’arrimeur sur Windows.

![Conteneurs Linux avec isolation Hyper-V (LCOW)](media/lcow-approach.png)

En examinant de plus près la façon dont la gestion des conteneurs diffère entre l’approche de machine virtuelle Moby et LCOW, dans la gestion des conteneurs de modèles LCOW reste sur Windows et chaque gestion LCOW s’effectue via GRPC et conteneur.  Cela signifie que les conteneurs Linux distribution utilisés pour LCOW peuvent avoir un inventaire plus petit.  Pour l’instant, nous utilisons LinuxKit pour l’utilisation des conteneurs distribution optimisés, mais d’autres projets tels que Kata créent également des distributions Linux hautement réglés (Clear Linux).

Voici une présentation détaillée de chaque LCOW :

![Architecture LCOW](media/lcow.png)

Pour voir si vous exécutez LCOW, accédez à `C:\Program Files\Linux Containers`. Si l’Ancreur est configuré pour utiliser LCOW, il y aura quelques fichiers contenant le distribution minimal LinuxKit qui s’exécute dans chaque conteneur s’exécutant sous l’isolation Hyper-V.  Notez que les composants de machine virtuelle optimisés sont inférieurs à 100 Mo, bien plus petits que l’image LinuxKit de la machine virtuelle Moby.

### <a name="work-in-progress"></a>Travail en cours

LCOW est en cours de développement actif. Suivre la progression en cours dans le projet Moby sur [GitHub](https://github.com/moby/moby/issues/33850)

#### <a name="bind-mounts"></a>Montages liés

La liaison des volumes de montage avec `docker run -v ...` stocke les fichiers sur le système de fichiers NTFS de Windows. Par conséquent, une traduction est nécessaire pour les opérations POSIX. À l’heure actuelle, certaines opérations de système de fichiers ne sont que partiellement implémentées ou ne le sont pas du tout, ce qui peut entraîner des incompatibilités pour certaines applications.

Ces opérations ne fonctionnent actuellement pas pour les volumes montés par liaison :

* MkNod
* XAttrWalk
* XAttrCreate
* Verrouiller
* Getlock
* Auth
* Purge
* INotify

Certaines ne sont pas totalement implémentés :

* GetAttr : le nombre Nlink est toujours signalé comme étant 2
* Open : seuls les indicateurs ReadWrite, WriteOnly et ReadOnly sont implémentés

Ces applications nécessitent toutes un mappage de volume et ne démarrent pas ou ne s’exécutent pas correctement.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>Informations supplémentaires

[Blog de l’arrimeur décrivant LCOW](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Vidéo sur les conteneurs Linux](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[Instructions de génération LinuxKit LCOW-kernel plus](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Quand utiliser la machine virtuelle Moby et LCOW

### <a name="when-to-use-moby-vm"></a>Quand utiliser la machine virtuelle Moby

Pour l’instant, nous vous recommandons d’utiliser la méthode de machine virtuelle Moby pour exécuter des conteneurs Linux pour les personnes qui :

- Voulez un environnement de conteneur stable.  Il s’agit de la Docker pour Windows par défaut.
- Exécutez des conteneurs Windows ou Linux, mais rarement les deux à la fois.
- Avoir des exigences de mise en réseau complexes ou personnalisées entre les conteneurs Linux.
- Il n’est pas nécessaire d’isoler le noyau (isolation Hyper-V) entre les conteneurs Linux.

### <a name="when-to-use-lcow"></a>Quand utiliser LCOW

Pour le moment, nous vous recommandons de LCOW pour les personnes qui :

- Vous souhaitez tester notre technologie la plus récente.
- Exécutez des conteneurs Windows et Linux en même temps.
- Isolation du noyau requise (isolation Hyper-V) entre les conteneurs Linux.

## <a name="other-options-we-considered"></a>Autres options que nous avons envisagées

Lorsque nous avons vu comment exécuter des conteneurs Linux sur Windows, nous avons considéré WSL. Au final, nous avons choisi une approche basée sur la virtualisation afin que les conteneurs Linux sur Windows soient cohérents avec les conteneurs Linux sur Linux. L’utilisation d’Hyper-V rend également LCOW plus sécurisée. Nous pouvons le réévaluer à l’avenir, mais pour le moment, LCOW continuera à utiliser Hyper-V.

Si vous avez des réflexions, envoyez vos commentaires via GitHub ou UserVoice.  Nous apprécions particulièrement vos commentaires sur l’expérience spécifique que vous aimeriez voir.

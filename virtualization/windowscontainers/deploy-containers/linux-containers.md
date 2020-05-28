---
title: Conteneurs Linux sur Windows 10
description: Découvrez les différentes façons dont vous pouvez utiliser Hyper-V pour exécuter des conteneurs Linux sur Windows 10 comme s’ils étaient natifs.
keywords: conteneurs linux, docker, conteneurs, windows 10
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 843bd0ab7ccf3a227482ba3a3d2677e36b395b29
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78854013"
---
# <a name="linux-containers-on-windows-10"></a>Conteneurs Linux sur Windows 10

Les conteneurs Linux constituent un pourcentage énorme de l’écosystème global de conteneurs et sont fondamentaux pour les expériences de développement et les environnements de production.  Toutefois, étant donné que les conteneurs partagent un noyau avec l’hôte de conteneur, l’exécution de conteneurs Linux directement sur Windows n’est pas envisageable[*](linux-containers.md#other-options-we-considered).  C’est là qu’intervient la virtualisation.

À ce stade, il existe deux façons d’exécuter des conteneurs Linux avec Docker pour Windows et Hyper-V :

- Exécuter des conteneurs Linux sur une machine virtuelle Linux complète : c’est ce que fait généralement Docker aujourd’hui.
- Exécuter des conteneurs Linux avec une [isolation Hyper-V](../manage-containers/hyperv-container.md) (conteneur Linux sur Windows) : il s’agit d’une nouvelle option dans Docker pour Windows.

> _L’exécution de conteneurs Linux sur un système d’exploitation Windows Server est actuellement toujours à un stade expérimental. Des licences supplémentaires pour le programme Docker EE seront nécessaires pour essayer cela. **Le reste de cet article a trait exclusivement à Windows 10**._

Cet article décrit le fonctionnement de chaque approche, fournit des conseils sur le choix de la solution appropriée et partage le travail en cours.

## <a name="linux-containers-in-a-moby-vm"></a>Conteneurs Linux sur une machine virtuelle Moby

Pour exécuter des conteneurs Linux sur une machine virtuelle Linux, suivez les instructions du [Guide de prise en main de Docker](https://docs.docker.com/docker-for-windows/).

Docker a pu exécuter des conteneurs Linux sur un ordinateur de bureau Windows dès sa première publication en 2016 (avant que l’isolation Hyper-V ou les conteneurs Linux sur Windows soient disponibles) en utilisant une machine virtuelle basée sur [LinuxKit](https://github.com/linuxkit/linuxkit) s’exécutant sur Hyper-V.

Dans ce modèle, le client docker s’exécute sur le bureau Windows, mais appelle le démon Docker sur la machine virtuelle Linux.

![Machine virtuelle Moby en tant qu’hôte de conteneur](media/MobyVM.png)

Dans ce modèle, tous les conteneurs Linux partagent un hôte de conteneur basé Linux. Par ailleurs, les conteneurs Linux :

* Partagent un noyau entre chacun d’eux et la machine virtuelle Moby, mais pas avec l’hôte Windows.
* Utilisent des propriétés de stockage et de mise en réseau cohérentes avec les conteneurs Linux s’exécutant sur Linux (puisqu’ils s’exécutent sur une machine virtuelle Linux).

Cela signifie également que l’hôte de conteneur Linux (machine virtuelle Moby) doit exécuter le démon Docker et toutes les dépendances de celui-ci.

Pour voir si vous opérez avec une machine virtuelle Moby, vérifiez le Gestionnaire Hyper-V pour machine virtuelle Moby soit en utilisant l’interface utilisateur du Gestionnaire Hyper-V, soit en exécutant `Get-VM` dans une fenêtre PowerShell avec élévation de privilèges.

## <a name="linux-containers-with-hyper-v-isolation"></a>Conteneurs Linux avec isolation Hyper-V

Pour tester des conteneurs Linux sur Windows 10 (LCOW10), suivez les instructions fournies dans [Conteneurs Linux sur Windows 10](../quick-start/quick-start-windows-10-linux.md). 

Les conteneurs Linux avec isolation Hyper-V exécutent chaque conteneur Linux sur une machine virtuelle Linux optimisée avec juste assez de système d’exploitation pour exécuter les conteneurs. Contrairement à l’approche de machine virtuelle Moby, chaque conteneur Linux a son propre noyau et son propre bac à sable de machine virtuelle. Les conteneurs sont également gérés directement par Docker sur Windows.

![Conteneurs Linux avec isolation Hyper-V (conteneur Linux sur Windows)](media/lcow-approach.png)

En examinant de plus près la façon dont la gestion des conteneurs diffère entre l’approche de la machine virtuelle Moby et du conteneur Linux sur Windows, on constate que, dans le modèle de conteneur Linux sur Windows, la gestion des conteneurs reste sur Windows et que la gestion de chaque conteneur Linux sur Windows s’effectue via GRPC et containerd.  Cela signifie que les conteneurs de la distribution Linux utilisés comme conteneurs Linux sur Windows peuvent avoir un inventaire beaucoup plus petit.  Pour l’instant, nous utilisons LinuxKit pour les conteneurs de la distribution optimisée, mais d’autres projets tels que Kata créent également des distributions Linux de pointe (projet Clear Linux).

Voici une présentation détaillée de chaque conteneur Linux sur Windows :

![Architecture de conteneur Linux sur Windows](media/lcow.png)

Pour voir si vous exécutez un conteneur Linux sur Windows, accédez à `C:\Program Files\Linux Containers`. Si Docker est configuré pour utiliser un conteneur Linux sur Windows, il y aura ici quelques fichiers contenant la distribution LinuxKit minimale qui s’exécute sur chaque conteneur opérant sous isolation Hyper-V.  Notez que les composants de machine virtuelle optimisée sont d’une taille inférieure à 100 Mo, bien plus petits que l’image LinuxKit dans la machine virtuelle Moby.

### <a name="work-in-progress"></a>Travail en cours

Le conteneur Linux sur Windows est en cours de développement. Suivez la progression en cours dans le projet Moby sur [GitHub](https://github.com/moby/moby/issues/33850).

#### <a name="bind-mounts"></a>Montages liés

La liaison des volumes de montage avec `docker run -v ...` stocke les fichiers sur le système de fichiers NTFS de Windows. Par conséquent, une traduction est nécessaire pour les opérations POSIX. À l’heure actuelle, certaines opérations de système de fichiers ne sont que partiellement implémentées ou ne le sont pas du tout, ce qui peut entraîner des incompatibilités pour certaines applications.

Ces opérations ne fonctionnent actuellement pas pour les volumes montés par liaison :

* MkNod
* XAttrWalk
* XAttrCreate
* Verrou
* Getlock
* Auth
* Purge
* INotify

Certaines ne sont pas totalement implémentées :

* GetAttr : le nombre Nlink est toujours signalé comme étant 2
* Open : seuls les indicateurs ReadWrite, WriteOnly et ReadOnly sont implémentés

Ces applications nécessitent toutes un mappage de volume et ne démarrent ni ne s’exécutent pas correctement.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>Informations supplémentaires

[Blog Docker décrivant un conteneur Linux sur Windows](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Vidéo sur les conteneurs Linux](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[Instructions pour la création de conteneur Linux sur Windows avec LinuxKit et noyau](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Quand utiliser une machine virtuelle Moby ou un conteneur Linux sur Windows

### <a name="when-to-use-moby-vm"></a>Quand utiliser une machine virtuelle Moby

Actuellement, nous recommandons l’approche machine virtuelle Moby de la méthode d’exécution de conteneurs Linux pour les personnes qui :

- Veulent un environnement de conteneur stable.  Il s’agit du Docker pour Windows par défaut.
- Exécutent des conteneurs Windows ou Linux, mais rarement les deux à la fois.
- Ont des exigences de mise en réseau complexe ou personnalisée entre les conteneurs Linux.
- N’ont pas besoin d’isolation de noyau (isolation Hyper-V) entre les conteneurs Linux.

### <a name="when-to-use-lcow"></a>Quand l’utiliser un conteneur Linux sur Windows

Actuellement, nous vous recommandons l’approche conteneur Linux sur Windows pour les personnes qui :

- Souhaitent tester notre technologie la plus récente.
- Exécutent des conteneurs Windows et Linux en même temps.
- Ont besoin d’une isolation de noyau (isolation Hyper-V) entre les conteneurs Linux.

## <a name="other-options-we-considered"></a>Autres options envisagées

Lors de l’examen de la manière d’exécuter des conteneurs Linux sur Windows, nous avons envisagé WSL. Au final, nous avons choisi une approche basée sur la virtualisation afin que les conteneurs Linux sur Windows soient cohérents avec les conteneurs Linux sur Linux. L’utilisation d’Hyper-V contribue également à sécuriser davantage le conteneur Linux sur Windows. Nous pourrions reconsidérer ce choix à l’avenir mais, pour le moment, le conteneur Linux sur Windows continuera d’utiliser Hyper-V.

Si vous avez des idées à partager, faites-nous part de vos commentaires via GitHub ou UserVoice.  Nous apprécions aussi beaucoup vos commentaires sur l’expérience spécifique dont vous aimeriez pouvoir bénéficier.

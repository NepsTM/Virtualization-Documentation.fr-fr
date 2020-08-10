---
title: Conteneurs Linux sur Windows 10
description: Découvrez les différentes façons dont vous pouvez utiliser Hyper-V pour exécuter des conteneurs Linux sur Windows 10 comme s’ils étaient natifs.
keywords: conteneurs linux, docker, conteneurs, windows 10
author: scooley
ms.date: 09/17/2019
ms.topic: overview
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 107b81fb17fdaee10e901bda8adf5a58ceba1188
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985353"
---
# <a name="linux-containers-on-windows-10"></a>Conteneurs Linux sur Windows 10

Les conteneurs Linux constituent un pourcentage énorme de l’écosystème global de conteneurs et sont fondamentaux pour les expériences de développement et les environnements de production.  Toutefois, étant donné que les conteneurs partagent un noyau avec l’hôte de conteneur, l’exécution de conteneurs Linux directement sur Windows n’est pas envisageable. C’est là qu’intervient la virtualisation.

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

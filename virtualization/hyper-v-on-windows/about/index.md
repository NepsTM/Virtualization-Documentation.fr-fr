---
title: Introduction à Hyper-V sur Windows10
description: Introduction à Hyper-V, à la virtualisation et aux technologies connexes.
keywords: windows10, hyper-v
author: scooley
ms.date: 06/25/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
ms.openlocfilehash: aab6a285e9c1ed9918b39cb1de88e3e2243fb3a9
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/28/2019
ms.locfileid: "9680959"
---
# <a name="introduction-to-hyper-v-on-windows-10"></a>Introduction à Hyper-V sur Windows10

> Hyper-V remplace MicrosoftVirtual PC.

Nombreux sont les développeurs de logiciels, professionnels de l’informatique ou passionnés de technologies qui doivent exécuter plusieurs systèmes d’exploitation. Hyper-V permet d’exécuter plusieurs systèmes d’exploitation en tant qu’ordinateurs virtuels sur Windows.

![Ordinateur virtuel exécutant Windows](media/HyperVNesting.png)

Hyper-V fournit spécifiquement des capacités de virtualisation matérielle.  Cela signifie que chaque machine virtuelle s’exécute sur du matériel virtuel.  Hyper-V vous permet de créer des disques durs virtuels, des commutateurs virtuels et un certain nombre d’autres périphériques virtuels qui peuvent être ajoutés aux machines virtuelles.

## <a name="reasons-to-use-virtualization"></a>Raisons justifiant la virtualisation

La virtualisation vous permet ce qui suit:

* Exécuter un logiciel nécessitant une version antérieure de Windows ou des systèmes d’exploitation autres que Windows.

* Se familiariser avec d’autres systèmes d’exploitation. Hyper-V permet de créer et de supprimer très facilement différents systèmes d’exploitation.

* Tester des logiciels sur plusieurs systèmes d’exploitation à l’aide de plusieurs machines virtuelles. Grâce à Hyper-V, vous pouvez les exécuter sur un seul ordinateur de bureau ou portable. Ces machines virtuelles peuvent être exportées, puis importées dans un autre système Hyper-V, notamment Azure.

## <a name="system-requirements"></a>Configuration requise

Hyper-V est disponible sur les versions 64 bits de Windows 10 professionnel, entreprise et éducation. Elle n’est pas disponible dans l’édition familiale.

> Effectuez la mise à niveau de Windows 10 édition familiale vers Windows **** > 10 professionnel en ouvrant la**mise à jour des paramètres et** > **l’activation**de la sécurité. Vous pouvez alors visiter le magasin et acheter la mise à niveau.

La plupart des ordinateurs exécutent Hyper-V, mais chaque machine virtuelle exécute un système d’exploitation entièrement différent.  Vous pouvez généralement exécuter une ou plusieurs machines virtuelles sur un ordinateur équipé de 4Go de RAM. Vous aurez cependant besoin de plus de ressources pour des machines virtuelles supplémentaires ou pour installer et exécuter des logiciels gourmands en ressources tels que des jeux, des logiciels de montage vidéo ou de conception d'ingénierie.

Pour en savoir plus sur la configuration requise pour Hyper-V et la procédure à suivre pour vérifier que Hyper-V s’exécute sur votre machine, consultez les [Références de la configuration requise pour Hyper-V](../reference/hyper-v-requirements.md).

## <a name="operating-systems-you-can-run-in-a-virtual-machine"></a>Systèmes d’exploitation que vous pouvez exécuter dans une machine virtuelle

Hyper-V sur Windows prend en charge de nombreux systèmes d’exploitation dans un ordinateur virtuel, y compris plusieurs versions de Linux, FreeBSD et Windows.

À titre de rappel, vous devez avoir une licence valide pour les systèmes d’exploitation que vous utilisez dans les machines virtuelles.

Pour en savoir plus sur les systèmes d’exploitation pris en charge comme invités dans Hyper-V sur Windows, voir [Systèmes d’exploitation invités Windows pris en charge](supported-guest-os.md) et [Systèmes d’exploitation invités Linux pris en charge](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows).

## <a name="differences-between-hyper-v-on-windows-and-hyper-v-on-windows-server"></a>Différences entre Hyper-V sur Windows et Hyper-V sur Windows Server

Certaines fonctionnalités se comportent différemment dans Hyper-V sur Windows et Hyper-V sur Windows Server.

Fonctionnalités Hyper-V disponibles uniquement sous WindowsServer:

* Migration dynamique des machines virtuelles d’un hôte vers un autre
* Réplication Hyper-V
* Fibre Channel virtuel
* Mise en réseau SR-IOV
* .VHDX partagé

Fonctionnalités Hyper-V disponibles uniquement sous Windows10:

* Création rapide et bibliothèque d’ordinateur virtuel
* Réseau par défaut (commutateur NAT)

Le modèle de gestion de mémoire est différent pour Hyper-V sur Windows. Sur un serveur, la mémoire Hyper-V est gérée en partant du principe que seules les machines virtuelles sont exécutées sur le serveur. Dans Hyper-V sur Windows, la mémoire est gérée en prévision du fait que la plupart des machines clientes exécutent des logiciels sur l’hôte en plus des machines virtuelles.

## <a name="limitations"></a>Limitations

Les programmes qui dépendent d’un matériel spécifique ne fonctionnent pas correctement dans une machine virtuelle. Par exemple, les jeux ou applications qui nécessitent un traitement avec des GPU risquent de ne pas fonctionner correctement. Par ailleurs, les applications basées sur les minuteurs sous 10ms, notamment les applications de mixage de concert et les minuteurs de haute précision, peuvent être confrontées à des problèmes d’exécution dans une machine virtuelle.

En outre, si Hyper-V est activé, les applications de haute précision sensibles à la latence peuvent également être confrontées à des problèmes d’exécution dans l’hôte.  En effet, quand la virtualisation est activée, le système d’exploitation hôte est également exécuté sur la couche de virtualisation Hyper-V, tout comme les systèmes d’exploitation invités. Toutefois, contrairement aux invités, le système d’exploitation hôte présente la particularité d’avoir un accès direct à l’ensemble du matériel, ce qui signifie que les applications avec une configuration matérielle requise spéciale continuent de fonctionner sans problème dans le système d’exploitation hôte.

## <a name="next-step"></a>Étape suivante

[Installer Hyper-V sur Windows10](../quick-start/enable-hyper-v.md)

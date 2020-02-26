---
title: Invités Windows pris en charge
description: Invités Windows pris en charge.
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
ms.openlocfilehash: 25c72b910af15fc0b498a5b2abce72d32e6d1efd
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439596"
---
# <a name="supported-windows-guests"></a>Invités Windows pris en charge

Cet article répertorie les combinaisons de systèmes d’exploitation prises en charge dans Hyper-V sur Windows.  Il sert également à présenter les services d’intégration et d’autres facteurs de prise en charge.

Microsoft a testé ces combinaisons hôte/invité.  Les problèmes liés à ces combinaisons peuvent être traités par les services PSS (Product Support Services).

Microsoft assure la prise en charge de la manière suivante :

* Les problèmes détectés dans les services d’intégration et systèmes d’exploitation Microsoft sont pris en charge par le support technique Microsoft.

* En ce qui concerne les problèmes dans d’autres systèmes d’exploitation ayant été certifiés par le fournisseur de système d’exploitation comme fonctionnant sur Hyper-V, la prise en charge est assurée par le fournisseur.

* En ce qui concerne les problèmes détectés dans d’autres systèmes d’exploitation, Microsoft soumet le problème à la communauté de support multifournisseur, [TASNet](http://www.tsanet.org/).

Pour pouvoir être pris en charge, tous les systèmes d’exploitation (invité et hôte) doit être mis à jour.  Recherchez les mises à jour critiques dans Windows Update.

## <a name="supported-guest-operating-systems"></a>Systèmes d’exploitation invités pris en charge

| Système d’exploitation invité |  Nombre maximal de processeurs virtuels | Remarques |
|:-----|:-----|:-----|
| Windows 10 | 32 |Le mode de session étendu ne fonctionne pas sur l'édition Windows 10 Famille |
| Windows 8.1 | 32 | |
| Windows 8 | 32 ||
| Windows 7 avec Service Pack 1 (SP1) | 4 | Édition Intégrale, Édition Entreprise et Édition Professionnel (32 bits et 64 bits). |
| Windows 7 | 4 | Édition Intégrale, Édition Entreprise et Édition Professionnel (32 bits et 64 bits). |
| Windows Vista avec Service Pack 2 (SP2) | 2 | Professionnel, Entreprise et Édition Intégrale, notamment les éditions N et KN. |
| - | | |
| [Canal semi-annuel Windows Server](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview) | 64 | |
| Windows Server 2019 | 64 | |
| Windows Server 2016 | 64 | |
| Windows Server 2012 R2 | 64 | |
| Windows Server 2012 | 64 | |
| Windows Server 2008 R2 avec Service Pack 1 (SP1) | 64 | Éditions Datacenter, Entreprise, Standard et Web. |
| Windows Server 2008 avec Service Pack 2 (SP2) | 4 | Éditions Datacenter, Entreprise, Standard et Web (32 bits et 64 bits). |
| Windows Home Server 2011 | 4 | |
| Windows Small Business Server 2011 | Édition Essentials - 2, édition Standard - 4 | |

> Windows 10 peut s’exécuter en tant que système d’exploitation invité sur les hôtes Hyper-V Windows 8.1 et Windows Server 2012 R2.

## <a name="supported-linux-and-free-bsd"></a>Linux et FreeBSD pris en charge

| Système d’exploitation invité |  |
|:-----|:------|
| [CentOS et Red Hat Enterprise Linux](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-CentOS-and-Red-Hat-Enterprise-Linux-virtual-machines-on-Hyper-V) | |
| [Machines virtuelles Debian sur Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Debian-virtual-machines-on-Hyper-V) | |
| [SUSE](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-SUSE-virtual-machines-on-Hyper-V) | |
| [Oracle Linux](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Oracle-Linux-virtual-machines-on-Hyper-V)  | |
| [Ubuntu](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Ubuntu-virtual-machines-on-Hyper-V) | |
| [FreeBSD](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-FreeBSD-virtual-machines-on-Hyper-V) | |

Pour plus d’informations, notamment concernant la prise en charge sur des versions précédentes d’Hyper-V, voir [Machines virtuelles Linux et FreeBSD sur Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows).

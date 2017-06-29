---
title: "Invités Windows pris en charge"
description: "Invités Windows pris en charge."
keywords: Windows10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
ms.openlocfilehash: 8b764cb06b94465f516f9e5e8f06860ced8b39bb
ms.sourcegitcommit: d5f30aa1bdfb34dd9e1909d73b5bd9f4153d6b46
ms.translationtype: HT
ms.contentlocale: fr-FR
---
# <a name="supported-windows-guests"></a>Invités Windows pris en charge 

Cet article répertorie les combinaisons de systèmes d’exploitation prises en charge dans Hyper-V sur Windows.  Il sert également à présenter les services d’intégration et d’autres facteurs de prise en charge.

## <a name="what-does-support-mean"></a>Que signifie la prise en charge? 
La prise en charge signifie que Microsoft a testé ces combinaisons hôte/invité.  Les problèmes liés à ces combinaisons peuvent être traités par les services PSS (Product Support Services).
 
Microsoft assure la prise en charge des systèmes d’exploitation invités de la façon suivante:
* Les problèmes détectés dans les services d’intégration et systèmes d’exploitation Microsoft sont pris en charge par le support technique Microsoft.
* En ce qui concerne les problèmes dans d’autres systèmes d’exploitation ayant été certifiés par le fournisseur de système d’exploitation comme fonctionnant sur Hyper-V, la prise en charge est assurée par le fournisseur.
* En ce qui concerne les problèmes détectés dans d’autres systèmes d’exploitation, Microsoft soumet le problème à la communauté de support multifournisseur, [TSANet](http://www.tsanet.org/).

Pour pouvoir être pris en charge, l’hôte Hyper-V et l’invité doivent être mis à jour avec toutes les mises à jour critiques disponibles dans Windows Update.

## <a name="supported-guest-operating-systems"></a>Systèmes d’exploitation invités pris en charge

Pour bénéficier de la prise en charge, les systèmes d’exploitation invités Windows et le système d’exploitation hôte doivent être à jour avec toutes les mises à jour critiques disponibles dans Windows Update.

| Système d’exploitation invité |  Nombre maximal de processeurs virtuels | Remarques | 
|:-----|:-----|:-----|
| Windows10 | 32 | |
| Windows8.1 | 32 | |
| Windows8 | 32 |    |
| Windows7 avec Service Pack1 (SP1) | 4 | Édition Intégrale, Édition Entreprise et Édition Professionnel (32bits et 64bits). |
| Windows7 | 4 | Édition Intégrale, Édition Entreprise et Édition Professionnel (32bits et 64bits). |
| WindowsVista avec Service Pack2 (SP2) | 2 | Professionnel, Entreprise et Édition Intégrale, notamment les éditions N et KN. | 
| - | | |
| Windows Server 2012 R2 | 64 | |
| Windows Server2012 | 64 | |
| Windows Server2008R2 avec Service Pack1 (SP1) | 64 | Éditions Datacenter, Entreprise, Standard et Web. |
| Windows Server2008 avec Service Pack2 (SP2) | 4 | Éditions Datacenter, Entreprise, Standard et Web (32bits et 64bits). |
| Windows Home Server2011 | 4 | |
| Windows Small Business Server2011 | Édition Essentials - 2, édition Standard - 4 | |
  
 > Windows10 peut s’exécuter en tant que système d’exploitation invité sur les hôtes Hyper-V Windows8.1 et Windows Server2012 R2.

## <a name="supported-linux-and-free-bsd"></a>Linux et FreeBSD pris en charge

| Système d’exploitation invité |  |
|:-----|:------|
| [CentOS et Red Hat Enterprise Linux ](https://technet.microsoft.com/library/dn531026.aspx) | |
| [Machines virtuelles Debian sur Hyper-V](https://technet.microsoft.com/library/dn614985.aspx) | |
| [SUSE](https://technet.microsoft.com/en-us/library/dn531027.aspx) | |
| [Oracle Linux](https://technet.microsoft.com/en-us/library/dn609828.aspx)  | |
| [Ubuntu](https://technet.microsoft.com/en-us/library/dn531029.aspx) | |
| [FreeBSD](https://technet.microsoft.com/library/dn848318.aspx) | |

Pour plus d’informations, notamment concernant la prise en charge sur des versions précédentes d’Hyper-V, voir [Machines virtuelles Linux et FreeBSD sur Hyper-V](https://technet.microsoft.com/library/dn531030.aspx).

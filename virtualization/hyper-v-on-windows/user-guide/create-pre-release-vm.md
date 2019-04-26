---
title: Essayer les fonctionnalités de la préversion pour Hyper-V
description: Essayer les fonctionnalités de la préversion pour Hyper-V
keywords: Windows10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 426c87cc-fa50-4b8d-934e-0b653d7dea7d
ms.openlocfilehash: ea91ea0ffca5479cb0593ef9961625f7b7ab1f42
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9577420"
---
# <a name="try-pre-release-features-for-hyper-v"></a>Essayer les fonctionnalités de la préversion pour Hyper-V

> Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.  
  Les machines virtuelles en préversion sont destinées à des environnements de développement ou de test uniquement, car Microsoft n’en assure pas le support.

Accédez en avant-première aux fonctionnalités de la préversion pour Hyper-V sur Windows Server2016 Technical Preview pour les tester dans vos environnements de développement ou de test. Vous pouvez être le premier à découvrir les dernières fonctionnalités Hyper-V et à participer à l’élaboration du produit en fournissant vos commentaires.

Les machines virtuelles que vous créez en préversion n’ont aucune compatibilité d’une build à l’autre, ni aucun support.  Ne les utilisez pas dans un environnement de production.

Voici d’autres raisons pour lesquelles elles sont réservées à des environnements de non-production:

* Il n’existe aucune compatibilité ascendante pour les machines virtuelles en préversion. Vous ne pouvez pas mettre à niveau ces machines virtuelles vers une nouvelle version de configuration.
* Les machines virtuelles en préversion n’ont pas de définition cohérente entre les builds. Si vous mettez à jour le système d’exploitation hôte, les machines virtuelles en préversion existantes peuvent être incompatibles avec l’hôte. Ces machines virtuelles peuvent ne pas démarrer ou peuvent sembler fonctionner au début pour ensuite rencontrer des problèmes de compatibilité importants.
* Si vous importez une machine virtuelle en préversion vers un hôte avec une build différente, les résultats sont imprévisibles. Vous pouvez déplacer une machine virtuelle en préversion vers un autre hôte. Toutefois, ce scénario ne fonctionne vraisemblablement que si les deux hôtes exécutent la même build.

## <a name="create-a-pre-release-virtual-machine"></a>Créer une machine virtuelle en préversion

Vous pouvez créer une machine virtuelle en préversion sur des hôtes Hyper-V qui exécutent Windows Server2016 Technical Preview.

1. Sur le Bureau Windows, cliquez sur le bouton Démarrer et tapez une partie du nom **Windows PowerShell**.
2. Cliquez avec le bouton droit sur **Windows PowerShell**, puis sélectionnez **Exécuter en tant qu’administrateur**.
3. Utilisez l’applet de commande [New-VM](https://technet.microsoft.com/library/hh848537.aspx) avec l’indicateur -Prerelease pour créer la machine virtuelle en préversion. Par exemple, exécutez la commande suivante, où VM Name est le nom de la machine virtuelle que vous souhaitez créer.

``` PowerShell
New-VM -Name <VM Name> -Prerelease
```
Autres exemples d’utilisation de l’indicateur -Prerelease:
 - Pour créer une machine virtuelle qui utilise un disque dur virtuel existant ou un nouveau disque dur, consultez les exemples PowerShell dans [Créer une machine virtuelle dans Hyper-V sur Windows Server2016 Technical Preview](https://technet.microsoft.com/library/mt126140.aspx#BKMK_PowerShell).
 - Pour créer un disque dur virtuel qui démarre sur une image du système d’exploitation, consultez l’exemple PowerShell dans [Déployer une machine virtuelle Windows dans Hyper-V sur Windows10](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_create_vm).

 Les exemples présentés dans ces articles fonctionnent pour les hôtes Hyper-V qui exécutent Windows10 ou Windows Server2016 Technical Preview. Pour l’instant, vous ne pouvez utiliser l’indicateur -Prerelease que pour créer une machine virtuelle en préversion sur des hôtes Hyper-V qui exécutent Windows Server2016 Technical Preview.

## <a name="see-also"></a>Voir aussi
-  [Blog de virtualisation](https://blogs.technet.microsoft.com/virtualization/): Découvrez les fonctionnalités de la préversion qui sont disponibles et comment les tester.
- [Versions de configuration de machines virtuelles prises en charge](https://technet.microsoft.com/library/mt695898.aspx#BKMK_SupportedConfigVersions): Apprenez à vérifier la version de configuration de la machine virtuelle et les versions prises en charge par Microsoft.

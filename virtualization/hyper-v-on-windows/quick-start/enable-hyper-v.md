---
title: Activer Hyper-V sur Windows10
description: Installer Hyper-V sur Windows10
keywords: Windows10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 752dc760-a33c-41bb-902c-3bb2ecd9ac86
ms.openlocfilehash: 034792fb65d890588f1c7edc89a075deab12628d
ms.sourcegitcommit: b7f37f3d385042ca8455b3e7d1fa887ac26989de
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/16/2018
---
# <a name="install-hyper-v-on-windows-10"></a>Installer Hyper-V sur Windows10

Activez Hyper-V pour créer des machines virtuelles sur Windows10.  
Hyper-V peut être activé de nombreuses manières, y compris à l’aide du Panneau de configuration de Windows10, de PowerShell (mon favori) ou de l’outil Gestion et maintenance des images de déploiement (DISM). Ce document présente chacune de ces options.

> **Remarque:** Hyper-V est intégré à Windows en tant que fonctionnalité facultative; il n’est pas disponible en téléchargement ni sous la forme de composant installable. 

## <a name="check-requirements"></a>Vérifier la configuration requise

* Windows10 Entreprise, Professionnel ou Éducation
* Processeur 64bits avec traduction d’adresse de second niveau (SLAT).
* Processeur prenant en charge les extensions de mode du moniteur de machine virtuelle (VT-c sur les processeurs Intel).
* Au minimum 4Go de mémoire.

Le rôle Hyper-V **ne peut pas** être installé sur Windows10 Famille.  
Mettez à niveau l’édition Windows10 Famille vers Windows10 Professionnel en ouvrant **Paramètres** > **Mise à jour et sécurité** > **Activation**.

Pour plus d’informations et pour connaître les étapes de résolution des problèmes, voir [Configuration requise pour Hyper-V sur Windows10](../reference/hyper-v-requirements.md).


## <a name="install-hyper-v"></a>InstallerHyper-V 
Hyper-V est intégré à Windows en tant que fonctionnalité facultative; il n’est pas disponible en téléchargement ni sous la forme de composant installable.  Le rôle Hyper-V intégré peut être activé de diverses façons.

### <a name="enable-hyper-v-using-powershell"></a>Activer Hyper-V à l’aide de PowerShell

1. Ouvrez une console PowerShell en tant qu’administrateur.

2. Exécutez la commande suivante:
  ```powershell
  Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
  ```  

  Si la commande est introuvable, assurez-vous que vous exécutez PowerShell en tant qu’administrateur.  

Une fois l’installation terminée, vous devez redémarrer votre ordinateur.  

### <a name="enable-hyper-v-with-cmd-and-dism"></a>Activer Hyper-V avec CMD et DISM

L’outil Gestion et maintenance des images de déploiement (DISM) vous aide à configurer Windows et les images Windows.  DSIM permet, entre autres, d’activer des fonctionnalités Windows pendant que le système d’exploitation est en cours d’exécution.  

Pour activer le rôle Hyper-V à l’aide de DISM:
1. Ouvrez une session PowerShell ou CMD en tant qu’administrateur.

2. Tapez la commande suivante:  
  ```powershell
  DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V
  ```  
  ![](media/dism_upd.png)

Pour plus d’informations sur DSIM, voir [DISM- Informations techniques de référence sur l’outil Gestion et maintenance des images de déploiement](https://technet.microsoft.com/en-us/library/hh824821.aspx).

### <a name="manually-enable-the-hyper-v-role"></a>Activer manuellement le rôle Hyper-V

1. Cliquez avec le bouton droit sur le bouton Windows et sélectionnez Applications et fonctionnalités.

2. Sélectionnez **Activer ou désactiver des fonctionnalités Windows**.

3. Sélectionnez **Hyper-V**, puis cliquez sur **OK**.  

![](media/enable_role_upd.png)

Une fois l’installation terminée, vous êtes invité à redémarrer votre ordinateur.

![](media/restart_upd.png)


## <a name="make-virtual-machines"></a>Créer des machines virtuelles
[Créez votre première machine virtuelle](quick-create-virtual-machine.md)

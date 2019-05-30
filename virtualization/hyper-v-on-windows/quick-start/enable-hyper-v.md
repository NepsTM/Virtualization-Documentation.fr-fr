---
title: Activer Hyper-V sur Windows10
description: Installer Hyper-V sur Windows10
keywords: Windows10, Hyper-V
author: scooley
ms.date: 02/15/2019
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 752dc760-a33c-41bb-902c-3bb2ecd9ac86
ms.openlocfilehash: e1b6b55b2e17ac4f0883078748d75f6d4b9fcafa
ms.sourcegitcommit: 3c06d6b5cebeacc4c0185dca5c20680ad67655bd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681900"
---
# <a name="install-hyper-v-on-windows-10"></a>Installer Hyper-V sur Windows10

Activez Hyper-V pour créer des machines virtuelles sur Windows10.  
Hyper-V peut être activé de diverses manières, notamment à l’aide du panneau de configuration Windows 10, PowerShell ou à l’aide de l’outil de gestion des services d’imagerie de déploiement (DISM). Ce document présente chacune de ces options.

> **Remarque:** Hyper-V est intégré à Windows en tant que fonctionnalité facultative; il n’est pas disponible en téléchargement.

## <a name="check-requirements"></a>Vérifier la configuration requise

* Windows 10 entreprise, professionnel ou éducation
* Processeur 64bits avec traduction d’adresse de second niveau (SLAT).
* Prise en charge de l’UC pour l’extension du mode moniteur VM (VT-c sur les processeurs Intel).
* Au minimum 4Go de mémoire.

Le rôle Hyper-V **ne peut pas** être installé sur Windows10 Famille.

Effectuez la mise à niveau de Windows 10 édition familiale vers Windows 10 professionnel en ouvrant la**mise à jour** > des **paramètres** > et**l’activation**de la sécurité.

Pour plus d’informations et pour connaître les étapes de résolution des problèmes, voir [Configuration requise pour Hyper-V sur Windows10](../reference/hyper-v-requirements.md).

## <a name="enable-hyper-v-using-powershell"></a>Activer Hyper-V à l’aide de PowerShell

1. Ouvrez une console PowerShell en tant qu’administrateur.

2. Exécutez la commande suivante:

  ```powershell
  Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
  ```

  Si la commande est introuvable, assurez-vous que vous exécutez PowerShell en tant qu’administrateur.

Une fois l’installation terminée, redémarrez.

## <a name="enable-hyper-v-with-cmd-and-dism"></a>Activer Hyper-V avec CMD et DISM

L’outil Gestion et maintenance des images de déploiement (DISM) vous aide à configurer Windows et les images Windows.  DSIM permet, entre autres, d’activer des fonctionnalités Windows pendant que le système d’exploitation est en cours d’exécution.

Pour activer le rôle Hyper-V à l’aide de DISM:

1. Ouvrez une session PowerShell ou CMD en tant qu’administrateur.

1. Tapez la commande suivante:

  ```powershell
  DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V
  ```

  ![Fenêtre de console montrant Hyper-V en cours d’activation.](media/dism_upd.png)

Pour plus d’informations sur DISM, voir [DISM- Informations techniques de référence sur l’outil Gestion et maintenance des images de déploiement](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-8.1-and-8/hh824821(v=win.10)>).

## <a name="enable-the-hyper-v-role-through-settings"></a>Activer le rôle Hyper-V via les paramètres

1. Cliquez avec le bouton droit sur le bouton Windows et sélectionnez Applications et fonctionnalités.

2. Sélectionnez **programmes et fonctionnalités** dans la partie droite sous paramètres liés. 

3. Sélectionnez **Activer ou désactiver des fonctionnalités Windows**.

4. Sélectionnez **Hyper-V**, puis cliquez sur **OK**.

![Boîte de dialogue des programmes et fonctionnalités Windows](media/enable_role_upd.png)

Une fois l’installation terminée, vous êtes invité à redémarrer votre ordinateur.

## <a name="make-virtual-machines"></a>Créer des machines virtuelles

[Créez votre première machine virtuelle](quick-create-virtual-machine.md)

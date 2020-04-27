---
title: Configuration requise pour Hyper-V sur Windows 10
description: Configuration requise pour Hyper-V sur Windows 10
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6e5e6b01-7a9d-4123-8cc7-f986e10cd372
ms.openlocfilehash: ebc9be132f05c20eb8daf9b5e6713b9258012305
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/24/2020
ms.locfileid: "78853986"
---
# <a name="windows-10-hyper-v-system-requirements"></a>Configuration requise pour Hyper-V sur Windows 10

Hyper-V est disponible dans les versions 64 bits des éditions Professionnel, Entreprise et Éducation de Windows 10. Hyper-V nécessite la technologie SLAT (Second Level Address Translation), présente dans la génération actuelle de processeurs 64 bits Intel et AMD.

Vous pouvez exécuter trois ou quatre machines virtuelles de base sur un hôte disposant de 4 Go de RAM. Pour exécuter plus de machines virtuelles, davantage de ressources sont nécessaires. En revanche, vous pouvez créer des machines virtuelles de grande taille avec 32 processeurs et 512 Go de RAM, selon votre matériel physique.

## <a name="operating-system-requirements"></a>Systèmes d'exploitation requis

Le rôle Hyper-V peut être activé sur ces versions de Windows 10 :

- Windows 10 Entreprise
- Windows 10 Professionnel
- Windows 10 Éducation

Le rôle Hyper-V **ne peut pas** être installé sur :

- Windows 10 Famille
- Windows 10 Mobile
- Windows 10 Mobile Entreprise

>Windows 10 Famille peut être mis à niveau vers Windows 10 Professionnel. Pour cela, ouvrez **Paramètres** > **Mise à jour et sécurité** > **Activation**. Vous pouvez alors visiter le magasin et acheter la mise à niveau.

## <a name="hardware-requirements"></a>Configuration matérielle requise

Bien que ce document ne fournisse pas la liste complète du matériel compatible avec Hyper-V, vous devez disposer des éléments suivants :

- Processeur 64 bits avec traduction d’adresse de second niveau (SLAT).
- Processeur prenant en charge les extensions de mode du moniteur de machine virtuelle (VT-x sur les processeurs Intel).
- Au minimum 4 Go de mémoire. Étant donné que les machines virtuelles partagent la mémoire avec l’hôte Hyper-V, vous devez fournir suffisamment de mémoire pour gérer la charge de travail virtuelle attendue.

Les éléments suivants doivent être activés dans le BIOS du système :
- Technologie de virtualisation (cette appellation peut varier selon le fabricant de la carte mère).
- Prévention de l’exécution des données appliquée par le matériel.

## <a name="verify-hardware-compatibility"></a>Vérifier la compatibilité matérielle

Après avoir pris connaissance des conditions requises en matière de système d'exploitation et de matériel, vérifiez la compatibilité matérielle dans Windows. Pour ce faire, ouvrez une session PowerShell ou une fenêtre d'invite de commandes (cmd.exe), entrez **systeminfo**, puis consultez la section Configuration requise pour Hyper-V. Si tous les éléments de la configuration requise pour Hyper-V ont la valeur **Oui**, votre système peut exécuter le rôle Hyper-V. Si l’un des éléments a la valeur **Non**, passez en revue la configuration requise figurant dans ce document et apportez des ajustements dans la mesure du possible.

![](media/SystemInfo-upd.png)

## <a name="final-check"></a>Vérification finale

Si les conditions requises sont toutes respectées en matière de système d'exploitation, de matériel et de compatibilité, **Hyper-V** apparaît dans **Panneau de configuration : Activer ou désactiver des fonctionnalités Windows**, avec 2 options.

1. Plateforme Hyper-V
1. Outils de gestion Hyper-V

![](media/hyper_v_feature_screenshot.png)

> [!NOTE] Si vous voyez **Plateforme de l'hyperviseur Windows** au lieu de **Hyper-V** dans **Panneau de configuration : Activer ou désactiver des fonctionnalités Windows**, cela signifie que votre système n'est peut-être pas compatible avec Hyper-V. Revérifiez les conditions requises ci-dessus.
>
>Si vous exécutez **systeminfo** sur un hôte Hyper-V existant, la section Configuration requise pour Hyper-V indique ce qui suit :
>
>```
>Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V will not be displayed.
>```

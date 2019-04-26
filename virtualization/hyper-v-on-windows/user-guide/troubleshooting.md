---
title: Résoudre les problèmes liés à Hyper-V sur Windows10
description: Résoudre les problèmes liés à Hyper-V sur Windows10
keywords: Windows10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: f0ec8eb4-ffc4-4bf1-9a19-7a8c3975b359
ms.openlocfilehash: 4d1b7b310d0df7c198d5446b339a9c38279c72db
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9575140"
---
# <a name="troubleshoot-hyper-v-on-windows-10"></a>Résoudre les problèmes liés à Hyper-V sur Windows10

## <a name="i-updated-to-windows-10-and-now-i-cant-connect-to-my-downlevel-windows-81-or-server-2012-r2-host"></a>Je viens d’effectuer la mise à jour vers Windows10 et je ne peux plus me connecter à mon hôte de niveau inférieur (Windows8.1 ou Windows Server2012R2)
Dans Windows10, le Gestionnaire Hyper-V se trouve dans WinRM pour la gestion à distance.  Cela signifie que vous devez à présent activer la gestion à distance sur l’hôte distant afin d’utiliser le Gestionnaire Hyper-V pour le gérer.

Pour plus d’informations, voir [Gestion d’hôtes Hyper-V distants](https://technet.microsoft.com/windows-server-docs/compute/hyper-v/manage/Remotely-manage-Hyper-V-hosts)

## <a name="i-changed-the-checkpoint-type-but-it-is-still-taking-the-wrong-type-of-checkpoint"></a>J’ai modifié le type de point de contrôle, mais c’est encore le mauvais type de point de contrôle qui est pris
Si vous prenez le point de contrôle de VMConnect et que vous modifiez le type de point de contrôle dans le Gestionnaire Hyper-V, le type du point de contrôle pris est celui qui a été spécifié lors de l’ouverture de VMConnect.

Fermez et rouvrez VMConnect pour que le bon type de point de contrôle soit pris.

## <a name="when-i-try-to-create-a-virtual-hard-disk-on-a-flash-drive-an-error-message-is-displayed"></a>Quand j’essaie de créer un disque dur virtuel sur un lecteur flash, un message d’erreur s’affiche
Hyper-V ne prend pas en charge les lecteurs de disque au format FAT/FAT32, car ces systèmes de fichiers ne fournissent pas de listes de contrôle d’accès et ne prennent pas en charge les fichiers de plus de 4Go. Les disques au format ExFAT fournissent uniquement des listes de contrôle d’accès avec des fonctionnalités limitées. Ils ne sont donc pas non plus pris en charge pour des raisons de sécurité.
Le message d’erreur affiché dans PowerShell est le suivant: «Le système n’a pas pu créer ’\[chemin du disque dur virtuel\]’. Impossible de terminer l’opération demandée du fait d’une limitation du système de fichiers (0x80070299).»

Utilisez plutôt un lecteur au format NTFS. 

## <a name="i-get-this-message-when-i-try-to-install-hyper-v-cannot-be-installed-the-processor-does-not-support-second-level-address-translation-slat"></a>J’obtiens le message suivant pendant l’installation: «Impossible d’installer Hyper-V: le processeur ne prend pas en charge la traduction d’adresse de second niveau (SLAT).»
Hyper-V a besoin de SLAT pour exécuter des machines virtuelles. Si votre ordinateur ne prend pas en charge SLAT, vous ne pouvez pas l’utiliser en tant qu’hôte pour des machines virtuelles.

Si vous essayez uniquement d’installer les outils de gestion, désélectionnez **Plateforme Hyper-V** dans **Programmes et fonctionnalités** > **Activer ou désactiver des fonctionnalités Windows**.

---
title: Créer une machine virtuelle avec Hyper-V
description: Créer une machine virtuelle avec Hyper-V sur Windows10Creators Update
keywords: windows10, hyper-v, ordinateur virtuel
author: scooley
ms.date: 04/07/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: f1e75efa-8745-4389-b8dc-91ca931fe5ae
ms.openlocfilehash: 970e92def02e5386d38a2e72d5ef921aa8321fdf
ms.sourcegitcommit: 08cc38955faad26f075b912a64b8ffb6b36f190c
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578682"
---
# <a name="create-a-virtual-machine-with-hyper-v"></a>Créer une machine virtuelle avec Hyper-V

Créer une machine virtuelle et installer son système d’exploitation.

Nous avons été à créer de nouveaux outils pour la création d’ordinateurs virtuels et les instructions ont considérablement changé au fil des trois dernières versions.

Choisissez votre système d’exploitation pour le bon jeu d’instructions:

* [Windows 10 Fall Creators Update (v1709) et versions ultérieures](quick-create-virtual-machine.md#windows-10-fall-creators-update-windows-10-version-1709)
* [Windows 10 Creators Update (v1703)](quick-create-virtual-machine.md#windows-10-creators-update-windows-10-version-1703)
* [Mise à jour anniversaire Windows 10 (v1607) et versions antérieures](quick-create-virtual-machine.md#before-windows-10-creators-update-windows-10-version-1607-and-earlier)

Commençons.

## <a name="windows-10-fall-creators-update-windows-10-version-1709"></a>Windows 10 Fall Creators Update (Windows 10 version 1709)

Dans Fall Creators Update, Création rapide a été étendu pour inclure une bibliothèque d’ordinateurs virtuels qui peut être lancée indépendamment du Gestionnaire Hyper-V.

Pour créer un nouvel ordinateur virtuel dans Fall Creators Update:

1. Ouvrez Création rapide Hyper-V à partir du menu Démarrer.

    ![Créer rapidement une bibliothèque dans le menu Démarrer de Windows](media/quick-create-start-menu.png)

1. Sélectionnez un système d’exploitation ou choisissez le vôtre un utilisant une source d’installation locale.

    ![Affichage de la bibliothèque](media/vmgallery.png)

    1. Si vous souhaitez utiliser votre propre image pour créer l’ordinateur virtuel, sélectionnez **Source d’installation locale **.
    1. Sélectionnez **Change Installation Source**.
      ![Bouton pour utiliser une source d’installation locale](media/change-source.png)
    1. Choisissez le fichier .iso ou .vhdx que vous souhaitez convertir en un nouvel ordinateur virtuel.
    1. Si l’image est une image Linux, désélectionnez l’option de démarrage sécurisé.
      ![Bouton pour utiliser une source d’installation locale](media/toggle-secure-boot.png)

1. Sélectionner «Créer un ordinateur virtuel»

Et voilà!  Création rapide se charge du reste.

## <a name="windows-10-creators-update-windows-10-version-1703"></a>Windows 10 Creators Update (Windows 10 version 1703)

![Capture d’écran de l’interface utilisation de Création rapide](media/quickcreatesteps_inked.jpg)

1. Ouvrez le gestionnaire Hyper-V à partir du menu Démarrer.

1. Dans le Gestionnaire Hyper-V, trouvez **Création rapide** sur le menu **Actions** de droite.

1. Personnalisez votre ordinateur virtuel.

    * (facultatif) Nommez la machine virtuelle.
    * Sélectionnez le support d’installation de la machine virtuelle. Vous pouvez l'installer à partir d’un fichier .iso ou .vhdx.
    Si vous installez Windows sur la machine virtuelle, vous pouvez activer le démarrage sécurisé de Windows. Sinon, laissez-le non sélectionné.
    * Configurer un réseau.
    Si vous disposez d'un commutateur virtuel existant, vous pouvez le sélectionner sur la liste déroulante du réseau. Si vous ne disposez d’aucun commutateur existant, vous verrez un bouton permettant de configurer un réseau automatique, ce qui configurera automatiquement un réseau virtuel.

1. Cliquez sur **Connecter** pour démarrer votre ordinateur virtuel. Ne vous inquiétez pas de la modification des paramètres, vous pouvez revenir en arrière et les modifier à tout moment.

    Vous pouvez être invité à appuyer sur une touche quelconque pour démarrer à partir d’un CD ou d’un DVD. Dans ce cas, appuyez sur une touche pour continuer.  Pour lui, vous effectuez l'installation depuis un CD.

Félicitations, vous avez une nouvelle machine virtuelle.  Vous êtes maintenant prêt à installer le système d’exploitation.

Votre ordinateur virtuel doit ressembler plus ou moins à ceci:

![Écran de démarrage de l’ordinateur virtuel](media/OSDeploy_upd.png)

> **Remarque:** si vous n’exécutez pas une version de Windows sous licence en volume, vous devez posséder une licence distincte pour Windows qui s’exécute sur un ordinateur virtuel. Le système d’exploitation de l’ordinateur virtuel est indépendant du système d’exploitation de l’hôte.

## <a name="before-windows-10-creators-update-windows-10-version-1607-and-earlier"></a>Avant Windows 10 Creators Update (Windows 10 version 1607 et versions antérieure)

Si vous n’exécutez pas Windows10Creators Update ou une version ultérieure, suivez plutôt les instructions à l’aide de l’Assistant Nouvel ordinateur virtuel:

1. [Créer un réseau virtuel](connect-to-network.md)
1. [Créer un ordinateur virtuel](create-virtual-machine.md)

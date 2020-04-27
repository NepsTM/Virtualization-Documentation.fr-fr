---
title: Créer une machine virtuelle avec Hyper-V
description: Créer une machine virtuelle avec Hyper-V sur Windows 10 Creators Update
keywords: windows 10, hyper-v, machine virtuelle
author: scooley
ms.date: 04/07/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: f1e75efa-8745-4389-b8dc-91ca931fe5ae
ms.openlocfilehash: 6035143bc1449bc4a8e9bb7a4484b4c5329e6d3c
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/24/2020
ms.locfileid: "77439626"
---
# <a name="create-a-virtual-machine-with-hyper-v"></a>Créer une machine virtuelle avec Hyper-V

Créer une machine virtuelle et installer son système d’exploitation.

Nous avons mis au point de nouveaux outils pour la création de machines virtuelles et les instructions ont considérablement changé au fil des trois dernières versions.

Choisissez votre système d’exploitation pour le bon jeu d’instructions :

* [Windows 10 Fall Creators Update (v1709) et versions ultérieures](quick-create-virtual-machine.md#windows-10-fall-creators-update-windows-10-version-1709)
* [Windows 10 Creators Update (v1703)](quick-create-virtual-machine.md#windows-10-creators-update-windows-10-version-1703)
* [Mise à jour anniversaire Windows 10 (v1607) et versions antérieures](quick-create-virtual-machine.md#before-windows-10-creators-update-windows-10-version-1607-and-earlier)

Commençons.

## <a name="windows-10-fall-creators-update-windows-10-version-1709"></a>Windows 10 Fall Creators Update (Windows 10 version 1709)

Dans Fall Creators Update, Création rapide a été étendu pour inclure une galerie de machines virtuelles qui peut être lancée indépendamment du Gestionnaire Hyper-V.

Pour créer une nouvelle machine virtuelle dans Fall Creators Update :

1. Ouvrez Création rapide Hyper-V à partir du menu Démarrer.

    ![Créer rapidement une bibliothèque dans le menu Démarrer de Windows](media/quick-create-start-menu.png)

1. Sélectionnez un système d’exploitation ou choisissez le vôtre un utilisant une source d’installation locale.

    ![Affichage de la bibliothèque](media/vmgallery.png)

    1. Si vous souhaitez utiliser votre propre image pour créer la machine virtuelle, sélectionnez **Source d’installation locale**.
    1. Sélectionnez **Modifier la source d'installation**.
      ![Bouton pour utiliser une source d’installation locale](media/change-source.png)
    1. Choisissez le fichier .iso ou .vhdx que vous souhaitez transformer en nouvelle machine virtuelle.
    1. Si l’image correspond à une image Linux, désélectionnez l’option de démarrage sécurisé.
      ![Bouton pour utiliser une source d’installation locale](media/toggle-secure-boot.png)

1. Sélectionner « Créer une machine virtuelle »

C’est tout !  Création rapide se charge du reste.

## <a name="windows-10-creators-update-windows-10-version-1703"></a>Windows 10 Creators Update (Windows 10 version 1703)

![Capture d’écran de l’interface utilisateur de Création rapide](media/quickcreatesteps_inked.jpg)

1. Ouvrez le Gestionnaire Hyper-V à partir du menu Démarrer.

1. Dans le Gestionnaire Hyper-V, trouvez **Création rapide** sur le menu **Actions** de droite.

1. Personnalisez votre machine virtuelle.

    * (facultatif) Nommez la machine virtuelle.
    * Sélectionnez le support d’installation de la machine virtuelle. Vous pouvez l'installer à partir d’un fichier .iso ou .vhdx.
    Si vous installez Windows sur la machine virtuelle, vous pouvez activer le démarrage sécurisé de Windows. Sinon, laissez-le non sélectionné.
    * Configurer un réseau.
    Si vous disposez d'un commutateur virtuel existant, vous pouvez le sélectionner sur la liste déroulante du réseau. Si vous ne disposez d’aucun commutateur existant, vous verrez un bouton permettant de configurer un réseau automatique, ce qui configurera automatiquement un réseau virtuel.

1. Cliquez sur **Connecter** pour démarrer votre machine virtuelle. Ne vous inquiétez pas de la modification des paramètres, vous pouvez revenir en arrière et les modifier à tout moment.

    Vous pouvez être invité à appuyer sur une touche quelconque pour démarrer à partir d’un CD ou d’un DVD. Dans ce cas, appuyez sur une touche pour continuer.  Pour lui, vous effectuez l'installation depuis un CD.

Félicitations, vous avez une nouvelle machine virtuelle.  Vous êtes maintenant prêt à installer le système d’exploitation.

Votre machine virtuelle doit ressembler plus ou moins à ceci :

![Écran de démarrage de la machine virtuelle](media/OSDeploy_upd.png)

> **Remarque :** si vous n’exécutez pas une version de Windows sous licence en volume, vous devez posséder une licence distincte pour l'exemplaire de Windows qui s’exécute sur une machine virtuelle. Le système d’exploitation de la machine virtuelle est indépendant du système d’exploitation de l’hôte.

## <a name="before-windows-10-creators-update-windows-10-version-1607-and-earlier"></a>Avant Windows 10 Creators Update (Windows 10 version 1607 et versions antérieures)

Si vous n’exécutez pas Windows 10 Creators Update ou une version ultérieure, suivez plutôt les instructions à l’aide de l’Assistant Nouvel ordinateur virtuel :

1. [Créer un réseau virtuel](connect-to-network.md)
1. [Créer une machine virtuelle](create-virtual-machine.md)

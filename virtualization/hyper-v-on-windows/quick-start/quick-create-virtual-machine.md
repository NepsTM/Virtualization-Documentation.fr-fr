---
title: "Créer une machine virtuelle avec Hyper-V"
description: "Créer une machine virtuelle avec Hyper-V sur Windows10Creators Update"
keywords: Windows10, Hyper-V
author: aoatkinson
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: f1e75efa-8745-4389-b8dc-91ca931fe5ae
ms.openlocfilehash: 1b2b778e882b413d29f52adf3e46e12e8aceede1
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/21/2017
---
# Créer une machine virtuelle avec Hyper-V

Créer une machine virtuelle et installer son système d’exploitation.  

Vous aurez besoin d'un fichier .iso du système d’exploitation que vous souhaitez exécuter. Si vous n’en avez pas à portée de main, prenez une copie d’évaluation de Windows dans le [centre d’évaluation TechNet](http://www.microsoft.com/en-us/evalcenter/).


> Windows10 Creators Update introduit un nouvel outil de **création rapide** pour simplifier la création de nouvelles machines virtuelles.  
  Si vous n’exécutez pas Windows10Creators Update ou une version ultérieure, suivez plutôt les instructions à l’aide de l’Assistant Nouvel ordinateur virtuel:  
  [Créer une machine virtuelle](create-virtual-machine.md)  
  [Créer un réseau virtuel](connect-to-network.md)

Commençons.

![](media/quickcreatesteps_inked.jpg)

1. **Ouvrez le Gestionnaire Hyper-V**  
  Appuyez sur la touche Windows et tapez «Gestionnaire Hyper-V» pour rechercher des applications pour le Gestionnaire Hyper-V ou faites défiler les applications dans votre menu Démarrer jusqu'à ce que vous trouviez le Gestionnaire Hyper-V.

2. **Ouvrez Création rapide**  
  Dans le Gestionnaire Hyper-V, trouvez **Création rapide** sur le menu **Actions** de droite.

3. **Personnaliser votre machine virtuelle**
  * (facultatif) Nommez la machine virtuelle.  
    Il s’agit du nom utilisé par Hyper-V pour la machine virtuelle, et non pas du nom d’ordinateur attribué au système d’exploitation invité qui sera déployé dans la machine virtuelle.
  * Sélectionnez le support d’installation de la machine virtuelle. Vous pouvez l'installer à partir d’un fichier .iso ou .vhdx.  
    Si vous installez Windows sur la machine virtuelle, vous pouvez activer le démarrage sécurisé de Windows. Sinon, laissez-le non sélectionné.
  * Configurer un réseau.  
    Si vous disposez d'un commutateur virtuel existant, vous pouvez le sélectionner sur la liste déroulante du réseau. Si vous ne disposez d’aucun commutateur existant, vous verrez un bouton permettant de configurer un réseau automatique, ce qui configurera automatiquement un commutateur externe.

4. **Se connecter à la machine virtuelle**  
  Sélectionner **Connexion** lance la connexion à une machine virtuelle et démarre celle-ci.     
  Ne vous inquiétez pas de la modification des paramètres, vous pouvez revenir en arrière et les modifier à tout moment.  
  
    Vous pouvez être invité à appuyer sur une touche quelconque pour démarrer à partir d’un CD ou d’un DVD. Dans ce cas, appuyez sur une touche pour continuer.  Pour lui, vous effectuez l'installation depuis un CD.

Félicitations, vous avez une nouvelle machine virtuelle.  Vous êtes maintenant prêt à installer le système d’exploitation.  

Votre machine virtuelle doit ressembler plus ou moins à ceci:  
![](media/OSDeploy_upd.png) 

> **Remarque:** si vous n’exécutez pas une version de Windows sous licence en volume, vous devez posséder une licence distincte pour l'exemplaire de Windows qui s’exécute sur une machine virtuelle. Le système d’exploitation de la machine virtuelle est indépendant du système d’exploitation de l’hôte.

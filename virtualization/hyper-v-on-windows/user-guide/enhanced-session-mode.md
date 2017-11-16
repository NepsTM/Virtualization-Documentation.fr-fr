---
title: "Partage de périphériques avec des ordinateurs virtuels Windows"
description: "Décrit le partage de périphériques avec des ordinateurs virtuels Hyper-V (USB, audio, microphone et lecteurs montés)"
keywords: windows10, hyper-v
ms.author: scooley
ms.date: 10/20/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d1aeb9cb-b18f-43cb-a568-46b33346a188
ms.openlocfilehash: 52d51fca03f454a311a123f20e5aeda9376fdc3d
ms.sourcegitcommit: 3ec9917c456875f68180323eb9e0470d5115325a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2017
---
# <a name="share-devices-with-your-virtual-machine"></a>Partager des périphériques avec votre ordinateur virtuel

> Disponible uniquement pour les ordinateurs virtuels Windows.

Le mode de session étendu permet à Hyper-V de se connecter à des ordinateurs virtuels à l’aide du protocole RDP (Remote Desktop Protocol).  Non seulement cela améliore-t-il d’une manière générale l’affichage de votre ordinateur virtuel, mais la connexion à l’aide du protocole RDP permet également à l’ordinateur virtuel de partager des périphériques avec votre ordinateur.  Dans la mesure où ce mode est activé par défaut dans Windows10, vous utilisez probablement déjà le protocole RDP pour vous connecter à vos ordinateurs virtuels Windows.  Cet article met en avant certains des avantages et quelques-unes des options masquées dans la boîte de dialogue des paramètres de connexion.

Le protocole RDP/mode de session étendu:

* Rend les ordinateurs virtuels redimensionnables et leur permet de reconnaître les résolutions élevées.
* Améliore l’intégration des ordinateurs virtuels
  * Presse-papiers partagé
  * Partage de fichiers par glisser-déplacer et copier-coller
* Permet le partage de périphériques
  * Microphone/haut-parleurs
  * PériphériquesUSB
  * Disques de données (y compris C:)
  * Imprimantes

Cet article décrit comment découvrir votre type de session, entrer en mode de session étendu et configurer les paramètres de votre session.

## <a name="share-drives-and-devices"></a>Partager des lecteurs et des périphériques

Les capacités de partage de périphériques du mode de session étendu sont cachées dans cette discrète fenêtre de connexion, qui s’affiche lorsque vous vous connectez à un ordinateur virtuel:

![](media/esm-default-view.png)

Par défaut, les ordinateurs virtuels qui utilisent le mode de session étendu partagent le Presse-papiers et les imprimantes.  Ils sont également configurés par défaut pour transmettre l’audio de l’ordinateur virtuel aux haut-parleurs de votre ordinateur.

Pour partager des périphériques avec votre ordinateur virtuel ou pour modifier ces paramètres par défaut:

1. Affichez davantage d’options

  ![](media/esm-show-options.png)

1. Affichez les ressources locales.

  ![](media/esm-local-resources.png)

### <a name="share-storage-and-usb-devices"></a>Partager des périphériques de stockage etUSB

Par défaut, les ordinateurs virtuels qui utilisent le mode de session étendu partagent des imprimantes, le Presse-papiers, et transmettent les cartes à puce ainsi que les autres périphériques de sécurité aux ordinateurs virtuels afin que vous puissiez utiliser des outils de connexion plus sécurisés à partir de votre ordinateur virtuel.

Pour partager d’autres appareils, tels que des périphériquesUSB ou votre lecteurC:, sélectionnez le menu «Plus...»:  
![](media/esm-more-devices.png)

À partir de cette boîte de dialogue, vous pouvez sélectionner les périphériques que vous souhaitez partager avec l’ordinateur virtuel.  Le lecteur système ( WindowsC:) est particulièrement utile pour le partage de fichiers.  
![](media/esm-drives-usb.png)

### <a name="share-audio-devices-speakers-and-microphones"></a>Partager des périphériques audio (haut-parleurs et microphones)

Par défaut, les ordinateurs virtuels qui utilisent le mode de session étendu transmettent l’audio de l’ordinateur virtuel afin que vous puissiez l’entendre.  L’ordinateur virtuel utilise le périphérique audio actuellement sélectionné sur l’ordinateur hôte.

Pour modifier ces paramètres ou pour ajouter la transmission par microphone (de sorte que vous puissiez enregistrer de l’audio sur un ordinateur virtuel):

Sélectionnez le menu «Paramètres...» pour la configuration des paramètres audio à distance  
![](media/esm-audio.png)

Configurez ensuite les paramètres relatifs à l’audio et au microphone  
![](media/esm-audio-settings.png)

Étant donné que votre ordinateur virtuel s’exécute sans doute localement, les options «Lire sur cet ordinateur» et «Lire sur l’ordinateur distant» produisent des résultats identiques.

## <a name="re-launching-the-connection-settings"></a>Lancer à nouveau les paramètres de connexion

Si la boîte de dialogue de résolution et de partage de périphériques ne s’affiche pas, essayez de lancer VMConnect indépendamment, à partir du menu Windows ou à partir de la ligne de commande en tant qu’administrateur.  

``` Powershell
vmconnect.exe
```

## <a name="check-session-type"></a>Vérifier le type de session

Vous pouvez vérifier le type de connexion dont vous bénéficiez grâce à l’icône du mode de session étendu située en haut de l’outil Virtual Machine Connect (VMConnect).  Ce bouton vous permet également de basculer entre une session simple et le mode de session étendu.

![](media/esm-button-location.png)

| icône | état de la connexion |
|:-----|:---------|
|![](media/esm-basic.png)| Vous êtes actuellement en mode de session étendu.  Cliquez sur cette icône pour vous reconnecter à votre ordinateur virtuel en mode simple. |
|![](media/esm-connect.png)| Vous êtes actuellement en mode de session simple, mais le mode de session étendu est disponible.  Cliquez sur cette icône pour vous reconnecter à votre ordinateur virtuel en mode de session étendu.  |
|![](media/esm-stop.png)| Vous êtes actuellement en mode simple.  Le mode de session étendu n’est pas disponible pour cet ordinateur virtuel. |
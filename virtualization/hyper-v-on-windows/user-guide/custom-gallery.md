---
title: Créer une bibliothèque d’ordinateurs virtuels personnalisée
description: Créez vos propres entrées dans la bibliothèque d’ordinateurs virtuels dans Windows10 Creators Update et versions ultérieures.
keywords: windows10, hyper-v, création rapide, ordinateur virtuel, bibliothèque
ms.author: scooley
ms.date: 05/04/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d9238389-7028-4015-8140-27253b156f37
ms.openlocfilehash: 2235201a56a238cbd5a75b0a6cae64cdb26108a2
ms.sourcegitcommit: edc153ffef01094c2324a0da2f9a301b31015a58
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/23/2018
ms.locfileid: "1928376"
---
# <a name="create-a-custom-virtual-machine-gallery"></a>Créer une bibliothèque d’ordinateurs virtuels personnalisée

> Windows10FallCreatorsUpdate et versions ultérieures.

Dans Fall Creators Update, Création rapide étendue pour inclure une bibliothèque d’ordinateurs virtuels.

![Création rapide d’une bibliothèque d’ordinateurs virtuels avec des images personnalisées](media/vmgallery.png)

Bien qu’il existe un ensemble d’images fournies parMicrosoft et les partenaires de Microsoft, la bibliothèque peut également contenir vos propres images.

Cet article détaille:

* la création d’ordinateurs virtuels compatibles avec la bibliothèque;
* la création d’une nouvelle source de bibliothèque;
* l’ajout d’une nouvelle source de bibliothèque personnalisée dans la bibliothèque.

## <a name="gallery-architecture"></a>Architecture de bibliothèque

La bibliothèque d’ordinateurs virtuels est une représentation graphique d’un ensemble de sources d’ordinateurs virtuels défini dans le Registre Windows.  Chaque source d’ordinateur virtuel est un chemin (chemin d’accès local ou URI) vers un fichier JSON contenant les ordinateurs virtuels sous forme de liste.

La liste des ordinateurs virtuels que vous voyez dans la bibliothèque correspond au contenu complet de la première source, suivi du contenu de la deuxième source, et ainsi de suite jusqu’à ce que tous les ordinateurs virtuels disponibles soient répertoriés.  La liste est créée dynamiquement chaque fois que vous lancez la bibliothèque.

![architecture de bibliothèque](media/vmgallery-architecture.png)

Clé de Registre: `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization`

Nom de valeur: `GalleryLocations`

Tapez: `REG_MULTI_SZ`

## <a name="create-gallery-compatible-virtual-machines"></a>Créer des ordinateurs virtuels compatibles avec une bibliothèque

Les ordinateurs virtuels de la bibliothèque peuvent être une image de disque (.iso) ou un disque dur virtuel (.vhdx).

Les ordinateurs virtuels créés à partir d’un disque dur virtuel ont quelques exigences de configuration:

1. Conçus pour prendre en charge le microprogramme UEFI. S’ils sont créés à l’aide d’Hyper-V, qui est un ordinateur virtuel deuxième génération.
1. Le disque dur virtuel doit contenir au moins 20Go - n’oubliez pas que c’est la taille maximale.  Hyper-V ne prend pas l’espace que l’ordinateur virtuel n’utilise pas activement.

### <a name="testing-a-new-vm-image"></a>Test d’une nouvelle image d’ordinateur virtuelle

La bibliothèque d’ordinateurs virtuels crée des ordinateurs virtuels à l’aide du même mécanisme que l’installation à partir d’une source d’installation locale.

Pour valider qu’une image d’ordinateur virtuel va démarrer et s’exécuter:

1. Ouvrez la bibliothèque d’ordinateurs virtuels (Création rapide Hyper-V) et sélectionnez **Source d’installation locale**.
  ![Bouton pour utiliser une source d’installation locale](media/use-local-source.png)
1. Sélectionnez **Change Installation Source**.
  ![Bouton pour utiliser une source d’installation locale](media/change-source.png)
1. Choisissez le fichier .iso ou .vhdx qui sera utilisé dans la bibliothèque.
1. Si l’image est une image Linux, désélectionnez l’option de démarrage sécurisé.
  ![Bouton pour utiliser une source d’installation locale](media/toggle-secure-boot.png)
1. Créez un ordinateur virtuel.  Si l’ordinateur virtuel démarre correctement, il est prêt pour la bibliothèque.

## <a name="build-a-new-gallery-source"></a>Créer une nouvelle source de bibliothèque

L’étape suivante consiste à créer une nouvelle source de bibliothèque.  Il s’agit du fichier JSON qui répertorie vos ordinateurs virtuels et qui ajoute toutes les informations supplémentaires que vous voyez dans la bibliothèque.

Informations textuelles:

![Emplacements de texte de bibliothèque étiquetés](media/gallery-text.png)

* **name** - obligatoire - c’est le nom qui apparaît dans la colonne de gauche et en haut de l’affichage de l’ordinateur virtuel.
* **publisher** - obligatoire
* **description** - obligatoire - liste de chaînes qui décrivent l’ordinateur virtuel.
* **version** - obligatoire
* lastUpdated - par défaut, lundi, 1erjanvier0001.

  Le format doit être: aaaa-mm-jj-Thh:mm:ssZ

  La commande PowerShell suivante fournit la date du jour au format correct et la place dans le Presse-papiers:

  ``` PowerShell
  Get-Date -UFormat "%Y-%m-%dT%TZ" | clip.exe
  ```

* locale - vide par défaut.

Images:

![Emplacements d’images de bibliothèque étiquetés](media/gallery-pictures.png)

* **logo** - obligatoire
* symbol
* thumbnail

Et, bien entendu, votre ordinateur virtuel (.iso ou .vhdx).

Le modèle JSON ci-après comporte des éléments de démarrage et de schéma de la bibliothèque.  Si vous le modifiez en VSCode, il fournit automatiquement IntelliSense.

[!code-json[main](../../../hyperv-tools/vmgallery/vm-gallery-template.json)]

## <a name="connect-your-gallery-to-the-vm-gallery-ui"></a>Connectez votre bibliothèque à l’interface utilisateur de bibliothèque d’ordinateur virtuel

La meilleure façon d’ajouter la source de votre bibliothèque personnalisée à la bibliothèque de l’ordinateur virtuel consiste à l’ajouter à regedit.

1. Ouvrez **regedit.exe**
1. Accédez à `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\`
1. Recherchez l’élément `GalleryLocations`.

    S’il existe déjà, accédez au menu **Edition** et **Modifier **.

    S’il n’existe pas déjà, accédez au menu **Edition**, naviguez de **New** à **Valeur de chaînes multiples**

1. Ajoutez votre bibliothèque à la clé de Registre `GalleryLocations`.

    ![Clé de Registre de bibliothèque avec le nouvel élément](media/new-gallery-uri.png)

## <a name="troubleshooting"></a>Résolution des problèmes

### <a name="check-for-errors-loading-gallery"></a>Recherchez des erreurs de chargement de la bibliothèque

La bibliothèque d’ordinateurs virtuels ne signale pas les erreurs dans l’Observateur d’événements Windows.  Pour rechercher des erreurs:

1. Ouvrez l’observateur d’événements
1. Naviguez vers **Journaux Windows** -> **Application**
1. Recherchez des événements à partir du fichier VMCreate source.

## <a name="resources"></a>Ressources

Il existe un petit nombre de scripts de bibliothèques et d’applications d’assistance dans GitHub [lien ](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/live/hyperv-tools/vmgallery).

Consultez un exemple d’entrée de bibliothèque [ici](https://go.microsoft.com/fwlink/?linkid=851584).  Il s’agit du fichier JSON qui définit la bibliothèque intégrée.

---
title: Créer une galerie de machines virtuelles personnalisée
description: Créez vos propres entrées dans la galerie de machines virtuelles dans Windows 10 Creators Update et versions ultérieures.
keywords: windows 10, hyper-v, création rapide, machine virtuelle, galerie
ms.author: scooley
ms.date: 05/04/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d9238389-7028-4015-8140-27253b156f37
ms.openlocfilehash: 1348b9923d9de1314818f13414abdacee2cb9735
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439710"
---
# <a name="create-a-custom-virtual-machine-gallery"></a>Créer une galerie de machines virtuelles personnalisée

> Windows 10 Fall Creators Update et versions ultérieures.

Dans Fall Creators Update, Création rapide étendue pour inclure une galerie de machines virtuelles.

![Création rapide d’une galerie de machines virtuelles avec des images personnalisées](media/vmgallery.png)

Bien qu’il existe un ensemble d’images fournies par Microsoft et les partenaires de Microsoft, la galerie peut également contenir vos propres images.

Cet article détaille :

* la création de machines virtuelles compatibles avec la galerie ;
* la création d’une nouvelle source de galerie ;
* l’ajout d’une nouvelle source de galerie personnalisée à la galerie.

## <a name="gallery-architecture"></a>Architecture de galerie

La galerie de machines virtuelles est une représentation graphique d’un ensemble de sources de machines virtuelles défini dans le Registre Windows.  Chaque source de machine virtuelle est un chemin (chemin d’accès local ou URI) vers un fichier JSON contenant les machines virtuelles sous forme de liste.

La liste des machines virtuelles que vous voyez dans la bibliothèque correspond au contenu complet de la première source, suivi du contenu de la deuxième source, et ainsi de suite jusqu’à ce que toutes les machines virtuelles disponibles soient répertoriées.  La liste est créée dynamiquement chaque fois que vous lancez la galerie.

![architecture de galerie](media/vmgallery-architecture.png)

Clé de Registre : `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization`

Nom de la valeur : `GalleryLocations`

Type : `REG_MULTI_SZ`

## <a name="create-gallery-compatible-virtual-machines"></a>Créer des machines virtuelles compatibles avec une galerie

Les machines virtuelles de la galerie peuvent être une image de disque (.iso) ou un disque dur virtuel (.vhdx).

Les machines virtuelles créées à partir d’un disque dur virtuel ont quelques exigences de configuration :

1. Conçues pour prendre en charge le microprogramme UEFI. Si elles sont créées avec Hyper-V, qui est une machine virtuelle deuxième génération.
1. Le disque dur virtuel doit contenir au moins 20 Go - n’oubliez pas que c’est la taille maximale.  Hyper-V ne prend pas l’espace que la machine virtuelle n’utilise pas activement.

### <a name="testing-a-new-vm-image"></a>Test d’une nouvelle image de machine virtuelle

La galerie de machines virtuelles crée des machines virtuelles à l’aide du même mécanisme que l’installation à partir d’une source d’installation locale.

Pour vérifier qu’une image de machine virtuelle va démarrer et s’exécuter :

1. Ouvrez la galerie de machines virtuelles (Création rapide Hyper-V) et sélectionnez **Source d’installation locale**.
  ![Bouton pour utiliser une source d’installation locale](media/use-local-source.png)
1. Sélectionnez **Modifier la source d'installation**.
  ![Bouton pour utiliser une source d’installation locale](media/change-source.png)
1. Choisissez le fichier .iso ou .vhdx qui sera utilisé dans la galerie.
1. Si l’image correspond à une image Linux, désélectionnez l’option de démarrage sécurisé.
  ![Bouton pour utiliser une source d’installation locale](media/toggle-secure-boot.png)
1. Créez une machine virtuelle.  Si la machine virtuelle démarre correctement, elle est prête pour la galerie.

## <a name="build-a-new-gallery-source"></a>Créer une nouvelle source de galerie

L’étape suivante consiste à créer une nouvelle source de galerie.  Il s’agit du fichier JSON qui répertorie vos machines virtuelles et qui ajoute toutes les informations supplémentaires que vous voyez dans la galerie.

Informations textuelles :

![Emplacements de texte de galerie étiquetés](media/gallery-text.png)

* **name** - obligatoire - c’est le nom qui apparaît dans la colonne de gauche et en haut de l’affichage de la machine virtuelle.
* **publisher** - obligatoire
* **description** - obligatoire - liste de chaînes qui décrivent la machine virtuelle.
* **version** - obligatoire
* lastUpdated - par défaut, lundi, 1er janvier 0001.

  Le format doit être : aaaa-mm-jj-Thh:mm:ssZ

  La commande PowerShell suivante fournit la date du jour au format qui convient et la place dans le Presse-papiers :

  ``` PowerShell
  Get-Date -UFormat "%Y-%m-%dT%TZ" | clip.exe
  ```

* locale - vide par défaut.

Images :

![Emplacements d’images de galerie étiquetés](media/gallery-pictures.png)

* **logo** - obligatoire
* symbol
* thumbnail

Et, bien entendu, votre machine virtuelle (.iso ou .vhdx).

Pour générer les hachages, vous pouvez utiliser la commande PowerShell suivante :

  ``` PowerShell
  Get-FileHash -Path .\TMLogo.jpg -Algorithm SHA256
  ```

Le modèle JSON ci-après comporte des éléments de démarrage et de schéma de la galerie.  Si vous le modifiez en VSCode, il fournit automatiquement IntelliSense.

[!code-json[main](../../../hyperv-tools/vmgallery/vm-gallery-template.json)]

## <a name="connect-your-gallery-to-the-vm-gallery-ui"></a>Connecter votre galerie à l’interface utilisateur de la galerie de machines virtuelles

La meilleure façon d’ajouter la source de votre galerie personnalisée à la galerie de machines virtuelles consiste à l’ajouter à regedit.

1. Ouvrez **regedit.exe**
1. Accédez à `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\`
1. Recherchez l’élément `GalleryLocations`.

    S’il existe déjà, accédez au menu **Édition** et **Modifier** .

    S’il n’existe pas déjà, accédez au menu **Édition**, naviguez de **Nouveau** à **Valeur de chaînes multiples**

1. Ajoutez votre galerie à la clé de Registre `GalleryLocations`.

    ![Clé de Registre de galerie avec le nouvel élément](media/new-gallery-uri.png)

## <a name="troubleshooting"></a>Résolution des problèmes

### <a name="check-for-errors-loading-gallery"></a>Rechercher des erreurs de chargement de la galerie

La galerie de machines virtuelles ne signale pas les erreurs dans l’Observateur d’événements Windows.  Pour rechercher des erreurs :

1. Ouvrez l’observateur d’événements
1. Accédez à **Journaux Windows** -> **Application**
1. Recherchez des événements à partir du fichier VMCreate source.

## <a name="resources"></a>Ressources

Il existe un certain nombre de scripts de galeries et d’applications d’assistance dans GitHub [lien](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/live/hyperv-tools/vmgallery).

Consultez un exemple d’entrée de galerie [ici](https://go.microsoft.com/fwlink/?linkid=851584).  Il s’agit du fichier JSON qui définit la galerie intégrée.

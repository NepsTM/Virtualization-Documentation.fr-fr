---
title: Configuration requise pour un conteneur Windows
description: Configuration requise pour un conteneur Windows.
keywords: "métadonnées, conteneurs"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: d4594bd5efe3e1852f6a47b6474bf9ea56196c14
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/21/2017
---
# Configuration requise pour un conteneur Windows

Ces guides fournissent la configuration requise pour un hôte de conteneur Windows.

## Configuration requise du système d’exploitation

- La fonctionnalité de conteneur Windows est disponible uniquement sur Windows Server2016 (Core et avec expérience utilisateur), Nano Server ainsi que Windows10 Professionnel et Entreprise (Édition anniversaire).
- Le rôle Hyper-V doit être installé avant d’exécuter des conteneurs Hyper-V.
- Sur les hôtes de conteneur Windows Server, Windows doit être installé sur C:\. Cette restriction ne s’applique pas si seuls les conteneurs Hyper-V sont déployés.

## Hôtes de conteneurs virtualisés

Si un hôte de conteneur Windows est exécuté sur une machine virtuelle Hyper-V et héberge également des conteneurs Hyper-V, la virtualisation imbriquée doit être activée. La configuration requise pour la virtualisation imbriquée est la suivante:

- Au moins 4 Go de RAM disponible pour l’hôte Hyper-V virtualisé.
- Windows Server2016 ou Windows10 sur le système hôte, et Windows Server (Full, Core) ou Nano Server sur la machine virtuelle.
- Un processeur Intel VT-x (cette fonctionnalité est actuellement disponible pour les processeurs Intel uniquement).
- La machine virtuelle de l’hôte de conteneur doit aussi disposer d’au moins 2processeurs virtuels.

## Images de base prises en charge

Les conteneurs Windows fournissent deux images de base de conteneur (Windows Server Core et Nano Server). Les configurations ne prennent pas toutes en charge les deux images de système d’exploitation. Ce tableau détaille les configurations prises en charge.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Système d’exploitation invité</center></th>
<th><center>Conteneur Windows Server</center></th>
<th><center>Conteneur Hyper-V</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server2016 (Standard ou Datacenter)</center></td>
<td><center>Server Core / Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server</center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows10 Professionnel / Entreprise</center></td>
<td><center>Non disponible</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
</tbody>
</table>

## Correspondance de la version d’hôte de conteneur avec les versions des images de conteneur
### Conteneurs de serveur Windows
Étant donné que les conteneurs Windows Server et l’hôte sous-jacent partagent un noyau unique, l’image de base du conteneur doit correspondre à celle de l’hôte.  Si les versions sont différentes, le conteneur peut démarrer, mais les fonctionnalités ne peuvent pas toutes être garanties. Par conséquent, des versions qui diffèrent ne sont pas prises en charge.  Le système d’exploitation Windows dispose de quatre niveaux de gestion de version: Majeure, Mineure, Build et Révision (par exemple, 10.0.14393.0). Le numéro de version change uniquement lorsque de nouvelles versions du système d’exploitation sont publiées. Le numéro de révision est mis à jour quand des mises à jour Windows sont appliquées. Le démarrage des conteneurs Windows Server est bloqué quand le numéro de build est différent: par exemple, 10.0.14300.1030 (Technical Preview5) et 10.0.14393 (Windows Server2016 RTM). Si le numéro de build correspond, mais que le numéro de révision est différent, il n’est pas bloqué: par exemple, 10.0.14393 (Windows Server2016 RTM) et 10.0.14393.206 (Windows Server2016 GA). Même si techniquement il n’est pas bloqué, cette configuration peut ne pas fonctionner correctement dans tous les cas et, par conséquent, ne peut pas être prise en charge pour les environnements de production. 

Pour vérifier quelle version un hôte Windows a installée, vous pouvez interroger HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion.  Pour vérifier quelle version votre image de base utilise, vous pouvez consulter les balises sur le hub Docker ou la table de hachage de l’image fournies dans la description de l’image.  La page [Historique des mises à jour de Windows10](https://support.microsoft.com/en-us/help/12387/windows-10-update-history) répertorie la date de publication de chaque build et de chaque révision.

Dans cet exemple, 14393 est le numéro de version majeure et 321 est le numéro de révision.
```none
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

### Isolation Hyper-V pour les conteneurs
Les conteneurs Windows peuvent être exécutés avec ou sans isolation Hyper-V.  L’isolation Hyper-V crée une limite sécurisée autour du conteneur avec un ordinateur virtuel optimisé.  Contrairement aux conteneurs Windows standard qui partagent le noyau entre les conteneurs et l’ordinateur hôte, chaque conteneur isolé Hyper-V dispose de sa propre instance du noyau Windows.  Pour cette raison, vous pouvez avoir différentes versions de système d’exploitation dans l’image d’hôte et l’image de conteneur (voir la matrice de compatibilité ci-dessous).  

Pour exécuter un conteneur ayant une isolation Hyper-V, ajoutez simplement la balise «--isolation=hyper-v»" à votre commande docker run.

### Matrice de compatibilité
Les builds WindowsServer avec un numéro de build supérieur à 2016GA (10.0.14393.206) peuvent exécuter les images Windows Server2016GA de WindowsServerCore ou NanoServer dans une configuration prise en charge, quel que soit le numéro de révision.    

Il est important de comprendre que pour disposer de toutes les fonctionnalités, de la fiabilité et des garanties de sécurité fournies avec les mises à jour Windows, vous devez toujours avoir les versions les plus récentes sur tous les systèmes.  

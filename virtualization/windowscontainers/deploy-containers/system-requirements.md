---
title: Configuration requise pour un conteneur Windows
description: Configuration requise pour un conteneur Windows.
keywords: "métadonnées, conteneurs"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 6ae690ff6592198bc16cbaf60489d3ed5aceeeb0
ms.sourcegitcommit: 64f5f8d838f72ea8e0e66a72eeb4ab78d982b715
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2017
---
# <a name="windows-container-requirements"></a>Configuration requise pour un conteneur Windows

Ces guides fournissent la configuration requise pour un hôte de conteneur Windows.

## <a name="os-requirements"></a>Configuration requise du système d’exploitation

- La fonctionnalité de conteneur Windows est disponible uniquement sur Windows Server build1709, Windows Server2016 (Core et avec expérience utilisateur), ainsi que Windows10 Professionnel et Entreprise (Édition anniversaire).
- Le rôle Hyper-V doit être installé avant d’exécuter des conteneurs Hyper-V.
- Sur les hôtes de conteneur Windows Server, Windows doit être installé sur C:\. Cette restriction ne s’applique pas si seuls les conteneurs Hyper-V sont déployés.

## <a name="virtualized-container-hosts"></a>Hôtes de conteneurs virtualisés

Si un hôte de conteneur Windows est exécuté sur une machine virtuelle Hyper-V et héberge également des conteneurs Hyper-V, la virtualisation imbriquée doit être activée. La configuration requise pour la virtualisation imbriquée est la suivante:

- Au moins 4 Go de RAM disponible pour l’hôte Hyper-V virtualisé.
- Windows Server build1709, Windows Server2016 ou Windows10 sur le système hôte, et Windows Server (Full, Core) sur l’ordinateur virtuel.
- Un processeur Intel VT-x (cette fonctionnalité est actuellement disponible pour les processeurs Intel uniquement).
- La machine virtuelle de l’hôte de conteneur doit aussi disposer d’au moins 2processeurs virtuels.

## <a name="supported-base-images"></a>Images de base prises en charge

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
<td><center>NanoServer*</center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows10 Professionnel / Entreprise</center></td>
<td><center>Non disponible</center></td>
<td><center>ServerCore / NanoServer</center></td>
</tr>
</tbody>
</table>
* À compter de WindowsServerversion1709, NanoServer n’est plus disponible en tant qu’hôte de conteneur.

### <a name="memory-requirments"></a>Mémoire requise
Les restrictions applicables à la mémoire disponible pour les conteneurs peuvent être configurées via [les contrôles de ressources](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls) ou la surcharge d’un hôte de conteneur.  La quantité minimale de mémoire requise pour lancer un conteneur et exécuter les commandes de base (ipconfig dir, etc.) est répertoriée ci-dessous.  Notez que ces valeurs ne tiennent pas compte du partage de ressources entre les conteneurs ni de la configuration requise par l’application en cours d’exécution dans le conteneur.

#### <a name="windows-server-2016"></a>WindowsServer2016
| Image de base  | Conteneur WindowsServer | Isolation Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| NanoServer | 40Mo                     | 130Mo + fichier d’échange 1Go |
| ServerCore | 50Mo                     | 325Mo + fichier d’échange 1Go |

#### <a name="windows-server-version-1709"></a>WindowsServer version1709
| Image de base  | Conteneur WindowsServer | Isolation Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| NanoServer | 30Mo                     | 110Mo + fichier d’échange 1Go |
| ServerCore | 45Mo                     | 360Mo + fichier d’échange 1Go |


### <a name="nano-server-vs-windows-server-core"></a>NanoServer et WindowsServerCore

Comment choisir entre WindowsServerCore et NanoServer? Bien qu’il soit possible de créer une application avec n’importe quelle configuration, si votre application doit être parfaitement compatible avec .NETFramework, vous devez utiliser [WindowsServerCore](https://hub.docker.com/r/microsoft/windowsservercore/). À l’inverse, si votre application est destinée au cloud et si elle utilise .NETCore, vous devez utiliser [NanoServer](https://hub.docker.com/r/microsoft/nanoserver/). En effet, NanoServer a été conçu de manière à avoir un très faible encombrement et, de ce fait, plusieurs bibliothèques non essentielles ont été supprimées. Il est important de garder ces points à l’esprit lorsque vous envisagez de créer une application basée sur NanoServer:

- La pile de traitements a été supprimée
- .NETCore n’est pas inclus (mais vous pouvez utiliser l’[image NanoServer de .NETCore](https://hub.docker.com/r/microsoft/dotnet/))
- PowerShell a été supprimé
- WMI a été supprimé

Il s’agit là des principales différences, mais cette liste n’est pas exhaustive. D’autres composants non mentionnés sont également absents. N’oubliez pas que vous pouvez ajouter autant de couches que vous le souhaitez sur NanoServer. Pour obtenir un exemple, consultez le [fichier Dockerfile NanoServer .NETCore](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile).

## <a name="matching-container-host-version-with-container-image-versions"></a>Correspondance de la version d’hôte de conteneur avec les versions des images de conteneur
### <a name="windows-server-containers"></a>Conteneurs de serveur Windows
Étant donné que les conteneurs Windows Server et l’hôte sous-jacent partagent un noyau unique, l’image de base du conteneur doit correspondre à celle de l’hôte.  Si les versions sont différentes, le conteneur peut démarrer, mais les fonctionnalités ne peuvent pas toutes être garanties. Par conséquent, des versions qui diffèrent ne sont pas prises en charge.  Le système d’exploitation Windows dispose de quatre niveaux de gestion de version: Majeure, Mineure, Build et Révision (par exemple, 10.0.14393.0). Le numéro de version change uniquement lorsque de nouvelles versions du système d’exploitation sont publiées. Le numéro de révision est mis à jour quand des mises à jour Windows sont appliquées. Le démarrage des conteneurs Windows Server est bloqué quand le numéro de build est différent: par exemple, 10.0.14300.1030 (Technical Preview5) et 10.0.14393 (Windows Server2016 RTM). Si le numéro de build correspond, mais que le numéro de révision est différent, il n’est pas bloqué: par exemple, 10.0.14393 (Windows Server2016 RTM) et 10.0.14393.206 (Windows Server2016 GA). Même si techniquement il n’est pas bloqué, cette configuration peut ne pas fonctionner correctement dans tous les cas et, par conséquent, ne peut pas être prise en charge pour les environnements de production. 

Pour vérifier quelle version un hôte Windows a installée, vous pouvez interroger HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion.  Pour vérifier quelle version votre image de base utilise, vous pouvez consulter les balises sur le hub Docker ou la table de hachage de l’image fournies dans la description de l’image.  La page [Historique des mises à jour de Windows10](https://support.microsoft.com/en-us/help/12387/windows-10-update-history) répertorie la date de publication de chaque build et de chaque révision.

Dans cet exemple, 14393 est le numéro de version majeure et 321 est le numéro de révision.
```
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

### <a name="hyper-v-isolation-for-containers"></a>Isolation Hyper-V pour les conteneurs
Les conteneurs Windows peuvent être exécutés avec ou sans isolation Hyper-V.  L’isolation Hyper-V crée une limite sécurisée autour du conteneur avec un ordinateur virtuel optimisé.  Contrairement aux conteneurs Windows standard qui partagent le noyau entre les conteneurs et l’ordinateur hôte, chaque conteneur isolé Hyper-V dispose de sa propre instance du noyau Windows.  Pour cette raison, vous pouvez avoir différentes versions de système d’exploitation dans l’image d’hôte et l’image de conteneur (voir la matrice de compatibilité ci-dessous).  

Pour exécuter un conteneur ayant une isolation Hyper-V, ajoutez simplement la balise «--isolation=hyper-v» à votre commande docker run.

### <a name="compatibility-matrix"></a>Matrice de compatibilité
Les builds WindowsServer avec un numéro de build supérieur à 2016GA (10.0.14393.206) peuvent exécuter les images Windows Server2016GA de WindowsServerCore ou NanoServer dans une configuration prise en charge, quel que soit le numéro de révision.    

Il est important de comprendre que pour disposer de toutes les fonctionnalités, de la fiabilité et des garanties de sécurité fournies avec les mises à jour Windows, vous devez toujours avoir les versions les plus récentes sur tous les systèmes.  

---
title: Configuration requise pour un conteneur Windows
description: Configuration requise pour un conteneur Windows.
keywords: métadonnées, conteneurs
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 4706ea24da1d5ca61b94dfd141883aa2d04ad906
ms.sourcegitcommit: 7c3af076eb8bad98e1c3de0af63dacd842efcfa3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/07/2018
ms.locfileid: "1844048"
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
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
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

> [!Warning]  
> <span id="warn-1">À compter de WindowsServerversion1709, NanoServer n’est plus disponible en tant qu’hôte de conteneur.</span>


### <a name="memory-requirements"></a>Mémoire requise
Les restrictions applicables à la mémoire disponible pour les conteneurs peuvent être configurées via [les contrôles de ressources](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls) ou la surcharge d’un hôte de conteneur.  La quantité minimale de mémoire requise pour lancer un conteneur et exécuter les commandes de base (ipconfig dir, etc.) est répertoriée ci-dessous.  __Notez que ces valeurs ne tiennent pas compte du partage de ressources entre les conteneurs ni de la configuration requise par l’application en cours d’exécution dans le conteneur.  Par exemple, un hôte avec 512Mo de mémoire disponible peut exécuter plusieurs conteneurs Server Core sous isolation Hyper-V, car ces conteneurs partagent des ressources.__

#### <a name="windows-server-2016"></a>Windows Server2016
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
- À partir de Windows Server version1709,les applications s’exécutent dans un contexte utilisateur. Ainsi, les commandes qui nécessitent des privilèges d’administrateur échouent. Vous pouvez spécifier le compte de l’administrateur de conteneur via l’indicateur d’utilisateur (autrement dit, docker run--utilisateur ContainerAdministrator). Toutefois à l’avenir, nous envisageons de supprimer complètement les comptes d’administrateur de NanoServer.

Il s’agit là des principales différences, mais cette liste n’est pas exhaustive. D’autres composants non mentionnés sont également absents. N’oubliez pas que vous pouvez ajouter autant de couches que vous le souhaitez sur NanoServer. Pour obtenir un exemple, consultez le [fichier Dockerfile NanoServer .NETCore](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile).


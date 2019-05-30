---
title: Configuration requise pour un conteneur Windows
description: Configuration requise pour un conteneur Windows.
keywords: métadonnées, conteneurs
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: d3df0631a8a61db16ad207f49163a7304c5db717
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681049"
---
# <a name="windows-container-requirements"></a>Configuration requise pour un conteneur Windows

Ce guide présente la configuration requise pour un hôte de conteneur Windows.

## <a name="os-requirements"></a>Configuration requise pour le système d’exploitation

- La fonctionnalité conteneur Windows est uniquement disponible sur Windows Server 2016 (Core et avec expérience de bureau), Windows 10 professionnel et entreprise (édition anniversaire) et versions ultérieures.
<<<<<<< HEAD
- Le rôle Hyper-V doit être installé avant d’exécuter l’isolement Hyper-V
- Sur les hôtes de conteneur Windows Server, Windows doit être installé sur C:\. Cette restriction ne s’applique pas si seuls les conteneurs isolés Hyper-V seront déployés.
=======
- Le rôle Hyper-V doit être installé avant d’exécuter des conteneurs avec l’isolation Hyper-V.
- Sur les hôtes de conteneur Windows Server, Windows doit être installé sur C:\. Cette restriction ne s’applique pas si seuls les conteneurs Hyper-V sont déployés.
>>>>>>> origine/maître

## <a name="virtualized-container-hosts"></a>Hôtes de conteneur virtualisés

<<<<<<< HEAD si un hôte de conteneur Windows sera exécuté à partir d’une machine virtuelle Hyper-V et sera également en hébergement d’isolement Hyper-V, la virtualisation imbriquée doit être activée. La virtualisation imbriquée présente la configuration requise suivante: = = = = = = si un hôte de conteneur Windows sera exécuté à partir d’une machine virtuelle Hyper-V et qu’il héberge également des conteneurs avec l’isolation Hyper-V, la virtualisation imbriquée doit être activée. La configuration requise pour la virtualisation imbriquée est la suivante:
>>>>>>> origine/maître

- Au moins 4 Go de RAM disponible pour l’hôte Hyper-V virtualisé.
- Windows Server 2019, Windows Server version 1803, Windows Server version 1709, Windows Server 2016 ou Windows 10 sur le système hôte et Windows Server (complet, cœur) sur la machine virtuelle.
- Un processeur Intel VT-x (cette fonctionnalité est actuellement disponible pour les processeurs Intel uniquement).
<<<<<<< HEAD
- L’ordinateur virtuel hôte de conteneur doit également disposer de deux processeurs virtuels.

## <a name="supported-base-images"></a>Images de base prises en charge

<a name="windows-containers-are-offered-with-four-container-base-images-windows-server-core-nano-server-windows-and-iot-core-not-all-configurations-support-both-os-images-this-table-details-the-supported-configurations"></a>Les conteneurs Windows sont proposés avec quatre images de base conteneur: Windows Server Core, nano Server, Windows et IoT Core. Les configurations ne prennent pas toutes en charge les deux images de système d’exploitation. Ce tableau détaille les configurations prises en charge.
=======
- La machine virtuelle de l’hôte de conteneur doit aussi disposer d’au moins 2processeurs virtuels.

## <a name="supported-base-images"></a>Images de base prises en charge

Les conteneurs Windows sont proposés avec quatre images de base conteneur: Windows Server Core, nano Server, Windows et IoT Core. Les configurations ne prennent pas toutes en charge les deux images de système d’exploitation. Ce tableau détaille les configurations prises en charge.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Système d’exploitation invité</center></th>
<th><center>Conteneur WindowsServer</center></th>
<th><center>Isolation Hyper-V</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016/2019 (standard ou Datacenter)</center></td>
<td><center>Serveur principal, nano Server, Windows</center></td>
<td><center>Serveur principal, nano Server, Windows</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
<td><center> Nano Server</center></td>
<td><center>Serveur principal, nano Server, Windows</center></td>
</tr>
<tr valign="top">
<td><center>Windows10 Professionnel / Entreprise</center></td>
<td><center>Windows<a href="#warn-2">**</a></center></td>
<td><center>Serveur principal, nano Server, Windows</center></td>
</tr>
<tr valign="top">
<td><center>IoT Standard</center></td>
<td><center>IoT Standard</center></td>
<td><center>Non disponible</center></td>
</tr>
</tbody>
</table>

> [!Warning]  
> <span id="warn-1">* Depuis Windows Server, la version 1709 de nano Server n’est plus disponible en tant qu’hôte de conteneur.</span>

> <span id="warn-2">* * Nécessite la mise à jour de Windows 10 octobre 2018 et l’utilisation directe d’un `--isolation=process` indicateur lors de l’exécution de vos `docker run`conteneurs via.</span>
>>>>>>> origine/maître

|Système d’exploitation hôte|Conteneur Windows|Isolation Hyper-V|
|---------------------|-----------------|-----------------|
|Windows Server 2016 ou Windows Server 2019 (standard ou Datacenter)|Serveur principal, nano Server, Windows|Serveur principal, nano Server, Windows|
|Nano Server|Nano Server|Serveur principal, nano Server, Windows|
|Windows 10 professionnel ou Windows 10 entreprise|Non disponible|Serveur principal, nano Server, Windows|
|IoT Standard|IoT Standard|Non disponible|

> [!WARNING]  
> À partir de Windows Server version 1709, nano Server n’est plus disponible en tant qu’hôte de conteneur.

### <a name="memory-requirements"></a>Mémoire requise

Les restrictions applicables à la mémoire disponible pour les conteneurs peuvent être configurées via [les contrôles de ressources](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls) ou la surcharge d’un hôte de conteneur.  Le volume minimal de mémoire requis pour le lancement d’un conteneur et l’exécution de commandes de base (ipconfig, dir, etc.) sont indiqués ci-dessous.

>[!NOTE]
>Ces valeurs ne prennent pas en compte le partage de ressources entre les conteneurs ou les exigences de l’application qui s’exécutent dans le conteneur.  Par exemple, un hôte avec 512Mo de mémoire disponible peut exécuter plusieurs conteneurs Server Core sous isolation Hyper-V, car ces conteneurs partagent des ressources.

#### <a name="windows-server-2016"></a>Windows Server2016

| Image de base  | Conteneur Windows Server | Isolation Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MO                     | 130 Mo + 1 Go de fichier d’échange |
| ServerCore | 50 MO                     | 325 Mo + 1 Go de fichier d’échange |

#### <a name="windows-server-version-1709"></a>WindowsServer version1709

| Image de base  | Conteneur Windows Server | Isolation Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MO                     | 110 Mo + 1 Go de fichier d’échange |
| ServerCore | 45 MO                     | 360 Mo + 1 Go de fichier d’échange |

### <a name="base-image-differences"></a>Différences d’image de base

Dans quelle mesure l’un choix est-il décidé de l’image de base appropriée? Même si vous êtes libre de générer ce que vous voulez, Voici les recommandations générales pour chaque image:

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): Si votre application a besoin du .NET Framework complet, il s’agit de la meilleure image à utiliser.
- [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): pour les applications qui nécessitent uniquement .net Core, nano Server offrira une image de plus en plus mince.
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): vous risquez de constater que votre application dépend d’un composant ou. dll manquant dans les images serveur ou nano Server, telles que les bibliothèques GDI. Cette image comporte le jeu de dépendances complet de Windows.
- [IOT](https://hub.docker.com/_/microsoft-windows-iotcore)standard: cette image est conçue spécialement pour les [applications IOT](https://developer.microsoft.com/windows/iot). Vous devez utiliser cette image de conteneur lors du ciblage d’un hôte de base IoT.

Pour la plupart des utilisateurs, Windows Server Core ou nano Server sera l’image la plus appropriée à utiliser. Voici quelques éléments à garder à l’esprit lorsque vous pensez à la création sur nano Server:

- La pile de traitements a été supprimée
- .NETCore n’est pas inclus (mais vous pouvez utiliser l’[image NanoServer de .NETCore](https://hub.docker.com/r/microsoft/dotnet/))
- PowerShell a été supprimé
- WMI a été supprimé
- À partir de Windows Server version1709,les applications s’exécutent dans un contexte utilisateur. Ainsi, les commandes qui nécessitent des privilèges d’administrateur échouent. Vous pouvez spécifier le compte d’administrateur de conteneurs par le biais de l’indicateur--User (par exemple, ContainerAdministrator), mais à l’avenir, nous envisageons de supprimer totalement les comptes d’administrateur du serveur.

Il s’agit là des principales différences, mais cette liste n’est pas exhaustive. D’autres composants non mentionnés sont également absents. N’oubliez pas que vous pouvez ajouter autant de couches que vous le souhaitez sur NanoServer. Pour obtenir un exemple, consultez le [fichier Dockerfile NanoServer .NETCore](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).
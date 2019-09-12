---
title: Historique des images de base des conteneurs Windows
description: Liste d’images de conteneurs Windows publiée avec les hachages de couche SHA256
keywords: Docker, conteneurs, hachages
author: patricklang
ms.date: 01/12/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: b2f2d6418fdda2ad0aa0b81c05efad6b99f74375
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107903"
---
# <a name="container-base-images"></a>Images de base du conteneur

## <a name="supported-base-images"></a>Images de base prises en charge

Les conteneurs Windows sont proposés avec quatre images de base conteneur: Windows Server Core, nano Server, Windows et IoT Core. Les configurations ne prennent pas toutes en charge les deux images de système d’exploitation. Ce tableau détaille les configurations prises en charge.

|Système d’exploitation hôte|Conteneur Windows|Isolation Hyper-V|
|---------------------|-----------------|-----------------|
|Windows Server 2016 ou Windows Server 2019 (standard ou Datacenter)|Serveur principal, nano Server, Windows|Serveur principal, nano Server, Windows|
|Nano Server|Nano Server|Serveur principal, nano Server, Windows|
|Windows 10 professionnel ou Windows 10 entreprise|Non disponible|Serveur principal, nano Server, Windows|
|IoT Standard|IoT Standard|Non disponible|

> [!WARNING]  
> À partir de Windows Server version 1709, nano Server n’est plus disponible en tant qu’hôte de conteneur.

## <a name="base-image-differences"></a>Différences d’image de base

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

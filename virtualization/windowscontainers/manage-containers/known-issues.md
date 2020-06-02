---
title: Problèmes connus
description: Problèmes connus sur les conteneurs Windows Server
keywords: métadonnées, conteneurs, version
author: weijuans
ms. author: weijuans
manager: taylob
ms.date: 05/26/2020
ms.openlocfilehash: e1c461a1f28954fb558f0629e0fafd4a7934ca14
ms.sourcegitcommit: 564a9226064077998020bfae721a17a8e0d9142e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/27/2020
ms.locfileid: "84106889"
---
# <a name="known-issues"></a>Problèmes connus

## <a name="know-issues-of-windows-server-version-2004"></a>Problèmes connus de Windows Server, version 2004

### <a name="1-performance-issue-on-server-core-container"></a>1. Problème de performances sur le conteneur Server Core
Lors de la préparation de la disponibilité générale de la mise en production de Windows Server, version 2004, nous avons identifié un problème de performances potentiel avec l’équipe .NET sur l’image conteneur Server Core actuelle dans la mise en production du 27 mai 2020, en comparant les améliorations des performances dont nous avons parlé dans le blog en décembre 2019 [ici](https://techcommunity.microsoft.com/t5/containers/making-windows-server-core-containers-40-smaller/ba-p/1058874) et sur le blog de l’équipe .NET [ici](https://devblogs.microsoft.com/dotnet/we-made-windows-server-core-container-images-40-smaller/). À l’époque, l’analyse des performances avait été effectuée sur une image conteneur Server Core de la mise en production Windows Server, version 2004 Insider Preview. 

Les symptômes que nous avions observés sont les suivants :

Si vous utilisez l’image conteneur Server Core pour créer votre propre image, que vous la chargez sur un registre de conteneurs distant comme Azure Container Registry puis que vous la tirez (pull) du registre et que vous l’exécutez, vous constatez que les performances du conteneur sont plus lentes. Par contre, si vous créez l’image et que vous l’exécutez localement, vous ne constaterez aucune différence de performance.

Étapes suivantes : Nous avons identifié des causes racines possibles et nous travaillons activement à la résolution de ce problème.  


## <a name="know-issues-of-windows-server-version-1909"></a>Problèmes connus de Windows Server, version 1909

## <a name="know-issues-of-windows-server-version-1903"></a>Problèmes connus de Windows Server, version 1903

## <a name="know-issues-of-windows-server-2019windows-server-version-1809"></a>Problèmes connus de Windows Server 2019/Windows Server, version 1809

## <a name="know-issues-of-windows-server-2016"></a>Problèmes connus de Windows Server 2016

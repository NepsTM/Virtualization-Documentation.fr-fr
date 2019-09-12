---
title: Configuration requise pour un conteneur Windows
description: Configuration requise pour un conteneur Windows.
keywords: métadonnées, conteneurs
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: df5d8e17d0d512f7f53fcacf6c2c2a2652f3e7c0
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107863"
---
# <a name="windows-container-requirements"></a>Configuration requise pour un conteneur Windows

Ce guide présente la configuration requise pour un hôte de conteneur Windows.

## <a name="os-requirements"></a>Configuration requise pour le système d’exploitation

- La fonctionnalité conteneur Windows est uniquement disponible sur Windows Server 2016 (Core et avec expérience de bureau), Windows 10 professionnel et entreprise (édition anniversaire) et versions ultérieures.
- Le rôle Hyper-V doit être installé avant d’exécuter l’isolement Hyper-V
- Sur les hôtes de conteneur Windows Server, Windows doit être installé sur C:\. Cette restriction ne s’applique pas si seuls les conteneurs isolés Hyper-V seront déployés.

## <a name="virtualized-container-hosts"></a>Hôtes de conteneur virtualisés

Si un hôte de conteneur Windows sera exécuté à partir d’une machine virtuelle Hyper-V et hébergera également l’isolation Hyper-V, la virtualisation imbriquée doit être activée. La configuration requise pour la virtualisation imbriquée est la suivante:

- Au moins 4 Go de RAM disponible pour l’hôte Hyper-V virtualisé.
- Windows Server 2019, Windows Server version 1803, Windows Server version 1709, Windows Server 2016 ou Windows 10 sur le système hôte et Windows Server (complet, cœur) sur la machine virtuelle.
- Un processeur Intel VT-x (cette fonctionnalité est actuellement disponible pour les processeurs Intel uniquement).
- L’ordinateur virtuel hôte de conteneur doit également disposer de deux processeurs virtuels.

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

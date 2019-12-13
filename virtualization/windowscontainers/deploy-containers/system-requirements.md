---
title: Configuration requise pour un conteneur Windows
description: Configuration requise pour un conteneur Windows.
keywords: métadonnées, conteneurs
author: taylorb-microsoft
ms.author: taylorb
ms.date: 10/22/2019
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 74f501e5efab3a93e60c9d4797464cea283cdc0b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910499"
---
# <a name="windows-container-requirements"></a>Configuration requise pour un conteneur Windows

Ce guide répertorie la configuration requise pour un hôte de conteneur Windows.

## <a name="operating-system-requirements"></a>Configuration requise pour le système d'exploitation

- La fonctionnalité de conteneur Windows est disponible sur Windows Server (canal semi-annuel), Windows Server 2019, Windows Server 2016 et Windows 10 éditions professionnel et entreprise (version 1607 et ultérieure).
- Le rôle Hyper-V doit être installé avant d’exécuter l’isolation Hyper-V
- Sur les hôtes de conteneur Windows Server, Windows doit être installé sur C:\. Cette restriction ne s’applique pas si seuls les conteneurs isolés Hyper-V seront déployés.

## <a name="virtualized-container-hosts"></a>Hôtes de conteneur virtualisés

Si un hôte de conteneur Windows est exécuté à partir d’un ordinateur virtuel Hyper-V et qu’il héberge également l’isolation Hyper-V, la virtualisation imbriquée doit être activée. La configuration requise pour la virtualisation imbriquée est la suivante :

- Au moins 4 Go de RAM disponible pour l’hôte Hyper-V virtualisé.
- Windows Server (canal semi-annuel), Windows Server 2019, Windows Server 2016 ou Windows 10 sur le système hôte ; et Windows Server (complète ou Server Core) sur la machine virtuelle.
- Un processeur Intel VT-x (cette fonctionnalité est actuellement disponible pour les processeurs Intel uniquement).
- La machine virtuelle hôte de conteneur nécessitera également au moins deux processeurs virtuels.

### <a name="memory-requirements"></a>Mémoire requise

Les restrictions applicables à la mémoire disponible pour les conteneurs peuvent être configurées via [les contrôles de ressources](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls) ou la surcharge d’un hôte de conteneur.  La quantité minimale de mémoire requise pour lancer un conteneur et exécuter des commandes de base (ipconfig, dir, etc.) sont répertoriées ci-dessous.

>[!NOTE]
>Ces valeurs ne tiennent pas compte du partage des ressources entre les conteneurs ou les exigences de l’application en cours d’exécution dans le conteneur.  Par exemple, un hôte avec 512 Mo de mémoire disponible peut exécuter plusieurs conteneurs Server Core sous isolation Hyper-V, car ces conteneurs partagent des ressources.

#### <a name="windows-server-2016"></a>Windows Server 2016

| Base image  | Conteneur Windows Server | Isolation Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 Mo                     | 130 Mo + fichier d’échange de 1 Go |
| Server Core | 50 Mo                     | 325 Mo + fichier d’échange de 1 Go |

#### <a name="windows-server-semi-annual-channel"></a>Windows Server (canal semi-annuel)

| Base image  | Conteneur Windows Server | Isolation Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 Mo                     | 110 Mo + fichier d’échange de 1 Go |
| Server Core | 45 MO                     | 360 Mo + fichier d’échange de 1 Go |

## <a name="see-also"></a>Voir également

[Stratégie de support pour les conteneurs et les conteneurs Windows dans des scénarios locaux](https://support.microsoft.com/help/4489234/support-policy-for-windows-containers-and-docker-on-premises)
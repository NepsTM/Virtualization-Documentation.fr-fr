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
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254259"
---
# <a name="windows-container-requirements"></a>Configuration requise pour un conteneur Windows

Ce guide présente la configuration requise pour un hôte de conteneur Windows.

## <a name="operating-system-requirements"></a>Configuration requise pour le système d'exploitation

- La fonctionnalité conteneur Windows est disponible sur Windows Server (canal semi-annuel), Windows Server 2019, Windows Server 2016 et Windows 10 éditions professionnel et entreprise (version 1607 et version ultérieure).
- Le rôle Hyper-V doit être installé avant d’exécuter l’isolement Hyper-V
- Sur les hôtes de conteneur Windows Server, Windows doit être installé sur C:\. Cette restriction ne s’applique pas si seuls les conteneurs isolés Hyper-V seront déployés.

## <a name="virtualized-container-hosts"></a>Hôtes de conteneur virtualisés

Si un hôte de conteneur Windows sera exécuté à partir d’une machine virtuelle Hyper-V et hébergera également l’isolation Hyper-V, la virtualisation imbriquée doit être activée. La configuration requise pour la virtualisation imbriquée est la suivante:

- Au moins 4 Go de RAM disponible pour l’hôte Hyper-V virtualisé.
- Windows Server (canal semi-annuel), Windows Server 2019, Windows Server 2016 ou Windows 10 sur le système hôte; et Windows Server (complet ou serveur principal) sur la machine virtuelle.
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

#### <a name="windows-server-semi-annual-channel"></a>WindowsServer (canal semi-annuel)

| Image de base  | Conteneur Windows Server | Isolation Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MO                     | 110 Mo + 1 Go de fichier d’échange |
| ServerCore | 45 MO                     | 360 Mo + 1 Go de fichier d’échange |

## <a name="see-also"></a>Articles associés

[Stratégie de prise en charge des conteneurs Windows et de l’Arrimateur dans des scénarios locaux](https://support.microsoft.com/help/4489234/support-policy-for-windows-containers-and-docker-on-premises)
---
title: Accélération GPU dans les conteneurs Windows
description: Le niveau d’accélération GPU existe dans les conteneurs Windows
keywords: arrimeur, conteneurs, appareils, matériel
author: cwilhit
ms.openlocfilehash: 6e5010efee10f9b488cbeb57b14bc86f30c1e766
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883272"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Accélération GPU dans les conteneurs Windows

Pour de nombreux réceptacles de charge de travail, les ressources de calcul de l’UC fournissent des performances suffisantes. Toutefois, dans le cas d’une certaine classe de charge de travail, la puissance de calcul massivement parallèle proposée par des GPU (unités de traitement graphique) peut accélérer les opérations en effectuant des commandes de magnitude, ce qui permet de réduire le coût et d’améliorer considérablement le débit.

Les processeurs graphiques sont déjà des outils communs pour de nombreuses charges de travail populaires, allant du rendu traditionnel et de la simulation à la formation et à l’inférence de l’apprentissage automatique. Les conteneurs Windows prennent en charge l’accélération GPU pour DirectX et toutes les infrastructures qu’il contient.

## <a name="requirements"></a>Spécifications

Pour que cette fonctionnalité fonctionne, votre environnement doit présenter la configuration suivante:

- L’hôte de conteneur doit exécuter Windows Server 2019 ou Windows 10 version 1809 ou ultérieure.
- L’image de base Container doit être [MCR.Microsoft.com/Windows:1809](https://hub.docker.com/_/microsoft-windowsfamily-windows) ou une version ultérieure. Les images de conteneur Windows Server Core et nano Server ne sont pas actuellement prises en charge.
- L’hôte de conteneur doit exécuter le moteur d’amarrage 19,03 ou une version ultérieure.
- L’hôte de conteneur doit disposer d’un GPU exécutant les pilotes d’affichage WDDM 2,5 ou version ultérieure.

Pour vérifier la version WDDM de vos pilotes d’affichage, exécutez l’outil de diagnostic DirectX (Dxdiag. exe) sur votre hôte de conteneur. Dans l’onglet «Affichage» de l’outil, accédez à la section «pilotes», comme indiqué ci-dessous.

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>Exécuter un conteneur avec l’accélération GPU

Pour démarrer un conteneur avec l’accélération GPU, exécutez la commande suivante:

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (et tous les infrastructures créées au-dessus) sont les seules API qui peuvent être accélérées avec un GPU dès aujourd’hui. les infrastructures tierces ne sont pas prises en charge.

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-V-prise en charge de conteneur Windows isolés

L’accélération GPU pour les charges de travail dans les conteneurs Windows isolés d’Hyper-V n’est actuellement pas prise en charge.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-V-prise en charge des conteneurs Linux isolés

L’accélération GPU pour les charges de travail dans Hyper-V-les conteneurs Linux isolés n’est actuellement pas prise en charge.
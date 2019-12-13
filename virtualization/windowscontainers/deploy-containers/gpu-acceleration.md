---
title: Accélération du GPU dans les conteneurs Windows
description: Quel est le niveau d’accélération du GPU dans les conteneurs Windows
keywords: station d’accueil, conteneurs, appareils, matériel
author: cwilhit
ms.openlocfilehash: 8f63c74d7839385e21206188263b9e5d08e7eb60
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909909"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Accélération du GPU dans les conteneurs Windows

Pour de nombreuses charges de travail en conteneur, les ressources de calcul de l’UC offrent des performances suffisantes. Toutefois, pour une certaine classe de charge de travail, la puissance de calcul massivement parallèle offerte par les GPU (unités de traitement graphique) peut accélérer les opérations par ordre de grandeur, en réduisant les coûts et en améliorant considérablement le débit.

Les GPU sont déjà un outil courant pour de nombreuses charges de travail populaires, allant du rendu et de la simulation traditionnels à Machine Learning la formation et l’inférence. Les conteneurs Windows prennent en charge l’accélération GPU pour DirectX et tous les frameworks basés sur celui-ci.

> [!NOTE]
> Cette fonctionnalité est disponible dans la version 2,1 et le moteur de l’Ancrable Desktop, version 19,03 ou ultérieure.

## <a name="requirements"></a>Configuration requise

Pour que cette fonctionnalité fonctionne, votre environnement doit répondre aux exigences suivantes :

- L’hôte de conteneur doit exécuter Windows Server 2019 ou Windows 10, version 1809 ou ultérieure.
- L’image de base du conteneur doit être [MCR.Microsoft.com/Windows :1809](https://hub.docker.com/_/microsoft-windows) ou une version plus récente. Les images de conteneur Windows Server Core et nano Server ne sont pas prises en charge actuellement.
- L’hôte de conteneur doit exécuter le moteur d’ancrage 19,03 ou une version plus récente.
- L’hôte de conteneur doit avoir un GPU exécutant des pilotes d’affichage version WDDM 2,5 ou ultérieure.

Pour vérifier la version WDDM de vos pilotes d’affichage, exécutez l’outil de diagnostic DirectX (Dxdiag. exe) sur votre hôte de conteneur. Dans l’onglet « Affichage » de l’outil, recherchez dans la section « pilotes », comme indiqué ci-dessous.

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>Exécuter un conteneur avec l’accélération GPU

Pour démarrer un conteneur avec l’accélération GPU, exécutez la commande suivante :

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (et tous les frameworks basés sur celui-ci) sont les seules API qui peuvent être accélérées avec un GPU dès aujourd’hui. les frameworks tiers ne sont pas pris en charge.

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-V-prise en charge des conteneurs Windows isolés

L’accélération GPU pour les charges de travail dans les conteneurs Windows isolés dans Hyper-V n’est pas prise en charge actuellement.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-V-prise en charge des conteneurs Linux isolés

L’accélération GPU pour les charges de travail dans les conteneurs Linux isolés par Hyper-V n’est pas prise en charge actuellement.

## <a name="more-information"></a>Informations supplémentaires

Pour obtenir un exemple complet d’une application DirectX en conteneur qui tire parti de l’accélération GPU, consultez [exemple de conteneur DirectX](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-container-samples/directx).

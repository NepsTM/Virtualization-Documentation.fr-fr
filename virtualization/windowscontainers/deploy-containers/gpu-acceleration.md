---
title: Accélération GPU dans les conteneurs Windows
description: Le niveau de l’accélération GPU existe dans les conteneurs Windows
keywords: docker, conteneurs, les appareils, matériel
author: cwilhit
ms.openlocfilehash: 281241e07e4bc184e73c4e74a117b44253a775be
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578662"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Accélération GPU dans les conteneurs Windows

De nombreuses charges de travail en conteneur, les ressources de calcul du processeur fournissent des performances suffisantes. Toutefois, pour une certaine classe de charge de travail, la puissance de calcul massivement parallèle offerte par les GPU (unités de traitement graphique) peut accélérer les opérations en ordre de grandeur, générant le coût et améliorer considérablement les débit.

GPU sont déjà un outil commun pour plusieurs charges de travail populaires, à partir de rendu classique et de simulation à la formation d’apprentissage machine et inférence de. Les conteneurs Windows prennent en charge l’accélération GPU pour DirectX et toutes les infrastructures basés sur elle.

> [!IMPORTANT]
> Cette fonctionnalité nécessite une version de Docker qui prend en charge la `--device` une option de ligne de commande pour les conteneurs Windows. Prise en charge de Docker formel est planifiée pour la prochaine version de Docker EE moteur 19.03. En attendant, la [source en amont](https://master.dockerproject.org/) pour Docker contient les bits nécessaires.

## <a name="requirements"></a>Spécifications

Pour cette fonctionnalité fonctionne, votre environnement doit satisfaire les conditions suivantes:

- L’hôte de conteneur doit exécuter Windows Server 2019 ou Windows 10, version 1809 ou une version ultérieure.
- L’image de base du conteneur doit être [mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windowsfamily-windows) ou une version ultérieure. Les images de conteneur Windows Server Core et Nano Server ne sont pas actuellement prises en charge.
- L’hôte de conteneur doit être en cours d’exécution du moteur Docker 19.03 ou une version ultérieure.
- L’hôte de conteneur doit avoir une version en cours d’exécution affichage pilotes GPU WDDM 2.5 ou une version ultérieure.

Pour vérifier la version WDDM de vos pilotes d’affichage, exécutez l’outil de Diagnostic DirectX (dxdiag.exe) sur votre hôte de conteneur. Dans l’onglet l’outil «Affichage», recherchez dans la section «Pilotes» comme indiqué ci-dessous.

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>Exécuter un conteneur avec accélération GPU

Pour démarrer un conteneur avec accélération GPU, exécutez la commande suivante:

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (et toutes les infrastructures basés sur elle) sont les seules API dont peuvent être accélérées dotée d’un GPU aujourd'hui. les infrastructures tierces 3e ne sont pas pris en charge.

## <a name="hyper-v-isolated-windows-container-support"></a>Prise en charge du conteneur Hyper-V-isolé de Windows

L’accélération GPU pour les charges de travail dans les conteneurs Hyper-V-isolé de Windows n’est pas pris en charge dès aujourd'hui.

## <a name="hyper-v-isolated-linux-container-support"></a>Prise en charge des conteneurs Linux Hyper-V-isolé

L’accélération GPU pour les charges de travail dans des conteneurs Linux isolé Hyper-V n’est pas pris en charge dès aujourd'hui.

---
title: Cycles de vie de maintenance des images de base
description: Informations sur le cycle de vie de l’image de base du conteneur Windows.
keywords: conteneurs Windows, conteneurs, cycle de vie, informations sur la version, image de base, image de base du conteneur
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 2dcd228af0984b55162894555fa21f9e02dd1934
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/15/2020
ms.locfileid: "81395742"
---
# <a name="base-image-servicing-lifecycles"></a>Cycles de vie de maintenance des images de base

> [!Note]  
> Microsoft a retardé la fin des dates de prise en charge et de maintenance planifiées pour un certain nombre de produits afin d’aider les personnes et les organisations à concentrer leur attention sur la conservation de la continuité des activités. Pour plus d’informations, consultez [modifications du cycle de vie de la fin de la prise en charge et des dates de maintenance](https://support.microsoft.com/en-us/help/4557164/lifecycle-changes-to-end-of-support-and-servicing-dates) à partir du 14 avril 2020.

Les images de base de conteneur Windows reposent sur des versions de canaux semi-annuelles ou des versions de canaux de maintenance à long terme de Windows Server. Cet article vous indique la durée de la prise en charge pour les différentes versions des images de base des deux canaux.

Le canal semi-annuel est une version de mise à jour de la fonctionnalité à deux fois par an avec des chronologies de maintenance de 18 mois pour chaque version. Cela permet aux clients de tirer parti des nouvelles fonctionnalités du système d’exploitation à un rythme plus rapide, à la fois dans les applications (en particulier celles basées sur les conteneurs et les microservices) et dans le centre de donnes hybrides défini par logiciel. Pour plus d’informations, consultez la [vue d’ensemble du canal semi-annuel Windows Server](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview).

Pour les images Server Core, les clients peuvent également utiliser le canal de maintenance à long terme qui libère une nouvelle version majeure de Windows Server toutes les deux à trois ans. Les versions de canaux de maintenance à long terme reçoivent cinq ans de support standard et cinq ans de support étendu. Ce canal fonctionne avec les systèmes qui nécessitent une option de maintenance plus longue et une stabilité fonctionnelle.

Le tableau suivant répertorie chaque type d’image de base, son canal de maintenance et la durée de sa prise en charge.

|Base image                       |Canal de maintenance|cible|Version du système d’exploitation|Disponibilité|Date de fin du support standard|Date de prise en charge étendue|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core, nano Server, Windows|Semi-annuel      |1909   |18363   |12/11/2019  |11/05/2021                 |N/A                  |
|Server Core, nano Server, Windows|Semi-annuel      |1903   |18362   |05/21/2019  |08/12/2020                 |N/A                  |
|Minimale                      |À long terme        |2019   |17763   |13/11/2018  |09/01/2024                 |09/01/2029           |
|Server Core, nano Server, Windows|Semi-annuel      |1809   |17763   |13/11/2018  |11/10/2020                 |N/A                  |
|Server Core, nano Server         |Semi-annuel      |1803   |17134   |30/04/2018  |12/11/2019                 |N/A                  |
|Server Core, nano Server         |Semi-annuel      |1709   |16299   |17/10/2017  |09/04/2019                 |N/A                  |
|Minimale                      |À long terme        |1607   |14393   |15/10/2016  |11/01/2022                 |11/01/2027           |
|Nano Server                      |Semi-annuel      |1607   |14393   |15/10/2016  |10/09/2018                 |N/A                  |

Pour les exigences de maintenance et d’autres informations supplémentaires, consultez le [Forum aux questions](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products)sur le cycle de vie de Windows, les [informations de version de Windows Server](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info)et les [images du système d’exploitation de base Windows référentiel Hub](https://hub.docker.com/_/microsoft-windows-base-os-images).

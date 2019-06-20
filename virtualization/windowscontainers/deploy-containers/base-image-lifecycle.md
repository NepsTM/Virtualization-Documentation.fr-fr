---
title: Durée de vie des images de base
description: Informations sur le cycle de vie de l’image de base du conteneur Windows.
keywords: conteneurs Windows, conteneurs, cycle de vie, informations de publication, image de base, image de base du conteneur
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: c26f4b225287fbc25566e36376eb8cd604d45a68
ms.sourcegitcommit: 9cd1aa792a417e71192c7aa39e409ae6ca0bc710
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/20/2019
ms.locfileid: "9788548"
---
# <a name="base-image-servicing-lifecycles"></a>Durée de vie des images de base

Les images de base des conteneurs Windows sont basées sur des publications de canal semi-annuel ou de maintenance de canal de maintenance à long terme de Windows Server. Cet article vous indique la durée de prise en charge de différentes versions d’images de base sur les deux canaux.

Le canal semi-annuel est une version de mise à jour de la fonctionnalité à la fois par année avec des plannings de maintenance de 18 mois pour chaque version. Cela permet aux utilisateurs de profiter des nouvelles fonctionnalités du système d’exploitation à un rythme plus rapide, à la fois dans les applications (notamment celles basées sur les conteneurs et les microservices) et dans le centre de donnes hybrides logiciel. Pour plus d’informations, consultez la [vue d’ensemble du canal semi-annuel de Windows Server](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview).

Pour les images principales du serveur, les clients peuvent également utiliser le canal de maintenance à long terme qui libère une nouvelle version majeure de Windows Server toutes les deux à trois ans. Les publications de canaux de maintenance à long terme bénéficient de cinq années d’assistance standard et de cinq ans de support technique étendu. Ce canal fonctionne avec les systèmes qui nécessitent une option de maintenance plus longue et une stabilité fonctionnelle.

Le tableau suivant répertorie chaque type d’image de base, son canal de maintenance et sa durée de prise en charge.

|Image de base                       |Canal de maintenance|Version|Version du système d’exploitation|Disponibilité|Date de fin du support standard|Date du support prolongé|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Serveur principal, nano Server, Windows|Semestriel      |1903   |18362   |05/21/2019  |12/08/2020                 |N/A                  |
|ServerCore                      |Durée de validité        |1809   |17763   |13/11/2018  |01/09/2024                 |01/09/2029           |
|Serveur principal, nano Server, Windows|Semestriel      |1809   |17763   |13/11/2018  |05/12/2020                 |N/A                  |
|Serveur principal, nano Server         |Semestriel      |1803   |17134   |30/04/2018  |12/11/2019                 |N/A                  |
|Serveur principal, nano Server         |Semestriel      |1709   |16299   |17/10/2017  |09/04/2019                 |N/A                  |
|ServerCore                      |Durée de validité        |1607   |14393   |15/10/2016  |11/01/2022                 |11/01/2027           |
|Nano Server                      |Semestriel      |1607   |14393   |15/10/2016  |10/09/2018                 |N/A                  |

Pour plus d’informations sur les exigences de maintenance et d’autres informations supplémentaires, consultez le [Forum aux questions sur le cycle de vie Windows](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products), les [informations de publication de Windows Server](https://docs.microsoft.com/en-us/windows-server/get-started/windows-server-release-info)et les [images du centre de référentiel Samples](https://hub.docker.com/_/microsoft-windows-base-os-images).

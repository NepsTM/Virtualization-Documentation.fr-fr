---
title: PowerShell pour Docker
description: "Comment gérer des conteneurs Docker à l’aide de PowerShell"
keywords: docker, conteneurs, powershell
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 4a0e907d-0d07-42f8-8203-2593391071da
ms.openlocfilehash: bcbc2e4e76c48a3d9a1a9720b09ef366a396bf30
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: fr-FR
---
### <a name="powershell-for-docker"></a>PowerShell pour Docker

En discutant avec vous, avec nos utilisateurs via les forums, sur Twitter, dans GitHub et même en personne, une question revient plus souvent que les autres: pourquoi ne puis-je pas voir les conteneurs Docker depuis PowerShell? 

Après avoir parlé des avantages, des inconvénients et de différentes autres options avec vous, nous sommes arrivés à la conclusion que le module PowerShell de conteneur avait besoin d’une mise à jour... Par conséquent, nous déconseillons le module PowerShell de conteneur qui accompagne les préversions de Windows Server2016 et nous travaillons à le remplacer par un nouveau module PowerShell pour Docker.  Nous avons déjà commencé à développer ce nouveau module, mais avec une approche différente de par le passé: cette fois, nous travaillons au grand jour.  En effet, nous aimerions que ce module soit le fruit d’une collaboration avec la communauté et qu’il devienne une expérience PowerShell réussie pour les conteneurs via le moteur Docker.  Ce nouveau module s’appuie directement sur l’interface REST du moteur Docker permettant à l’utilisateur de choisir entre l’interface de ligne de commande Docker, PowerShell, ou les deux.

La création d’un module PowerShell réussi n’est pas une tâche facile: le code ne doit comporter aucune erreur et il est essentiel de trouver un juste équilibre entre les objets, les ensembles de paramètres et les noms des applets de commande.  C’est pourquoi, au moment de nous lancer dans ce nouveau module, nous comptons sur vous, nos utilisateurs finaux et les grandes communautés PowerShell et Docker, pour participer à l’élaboration de ce module.  Quels ensembles de paramètres sont importants pour vous?  Devons-nous trouver un équivalent à «docker run» ou devez-vous diriger new-container vers start-container? Qu’aimeriez-vous?  Pour en savoir plus sur ce module et participer au développement, visitez notre page GitHub (https://github.com/Microsoft/Docker-PowerShell/) et rejoignez-nous.

À mesure du développement et dès que nous obtiendrons un module de bonne qualité alpha, nous le publierons sur la galerie PowerShell et mettrons à jour cette page avec des instructions sur son utilisation.

---
title: Gestion des ressources de conteneur
description: Gérez les ressources de conteneur avec des conteneurs Windows.
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b2192e64-9d74-474e-8af0-2d8b3ad1deee
---

# Gestion des ressources de conteneur

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

Les conteneurs Windows offrent la possibilité de gérer la quantité de ressources de processeur, d’E/S disque, de réseau et de mémoire que les conteneurs peuvent consommer. En limitant la consommation de ressources du conteneur, vous permettez que les ressources de l’hôte soient utilisées efficacement et vous évitez la surconsommation. Ce document décrit en détail la gestion des ressources de conteneur avec Docker.

## Gérer les ressources avec Docker 

Nous offrons la possibilité de gérer un sous-ensemble des ressources de conteneur avec Docker. Plus précisément, nous permettons aux utilisateurs de spécifier la façon dont le processeur est partagé entre les conteneurs. 

### Processeur

Le pourcentage de processeur réparti entre les conteneurs peut être géré au moment de l’exécution avec l’indicateur --cpu-shares. Par défaut, tous les conteneurs bénéficient d’une même proportion de temps processeur. Pour modifier le pourcentage relatif de processeur que les conteneurs utilisent, exécutez l’indicateur --cpu-shares avec une valeur comprise entre 1 et 10 000. Par défaut, tous les conteneurs reçoivent une pondération de 5 000. Pour plus d’informations sur la contrainte de partage de processeur, voir les [informations de référence sur Docker Run]( https://docs.docker.com/engine/reference/run/#cpu-share-constraint). 

```none 
docker run -it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## Problèmes connus

- Les contrôles de ressources de processeur et d’E/S ne sont pas pris en charge par les conteneurs Hyper-V.
- Les contrôles de ressources d’E/S ne sont pas pris en charge par les volumes de données de conteneur.

<!--HONumber=May16_HO3-->



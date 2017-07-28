---
title: "Démarrage rapide des conteneurs Windows"
description: "Démarrage rapide des conteneurs Windows."
keywords: docker, conteneurs
author: enderb-ms
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-contianers
ms.service: windows-containers
ms.assetid: 4878f5d2-014f-4f3c-9933-97f03348a147
ms.openlocfilehash: fb97f1d0f533b28acfb711e52bd021b29212f66e
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/21/2017
---
# Démarrage rapide des conteneurs Windows

La rubrique Démarrage rapide des conteneurs Windows présente la terminologie propre au produit et aux conteneurs, passe en revue des exemples de déploiement de conteneurs simples et fournit également des informations de référence concernant des rubriques plus avancées. Si vous n’êtes pas familiarisé avec les conteneurs ou les conteneurs Windows, suivez chaque étape de cette rubrique de démarrage rapide pour obtenir une expérience pratique de cette technologie.

## 1. Présentation des conteneurs

Ils constituent un environnement d’exploitation isolé, mobile et contrôlé par les ressources.

En fait, un conteneur est un emplacement isolé dans lequel une application peut s’exécuter sans affecter le reste du système et sans que le système n’affecte l’application. Les conteneurs sont la prochaine étape de la virtualisation.

Si vous étiez à l’intérieur d’un conteneur, vous vous sentiriez comme dans une machine virtuelle ou un ordinateur physique nouvellement installé. Enfin, pour [Docker](https://www.docker.com/), un conteneur Windows Server peut être géré de la même façon que n’importe quel autre conteneur.

## 2. Types de conteneurs Windows

Les conteneurs Windows incluent deux types de conteneurs différents, ou runtimes.

**Conteneurs Windows Server**: Ils assurent l’isolation des applications via une technologie d’isolation des processus et des espaces de noms. Un conteneur Windows Server partage un noyau avec l’hôte de conteneur et tous les conteneurs exécutés sur l’hôte.  Ces conteneurs ne créent pas de frontière de sécurité contre le code hostile et ne doivent pas être utilisés pour isoler du code non fiable.  Ces conteneurs partagent l’espace de noyau avec l’hôte et les autres conteneurs exécutés sur l’hôte. Par conséquent, le noyau doit être cohérent, avec la même version et configuration.

**Isolation Hyper-V**: développe l’isolation fournie par les conteneurs WindowsServer en exécutant chaque conteneur dans une machine virtuelle hautement optimisée. Dans cette configuration, le noyau de l’hôte de conteneur n’est pas partagé avec d’autres conteneurs exécutés sur l’hôte.  Ces conteneurs sont conçus pour un hébergement mutualisé hostile avec les mêmes garanties de sécurité qu’une machine virtuelle. Dans la mesure où ces conteneurs ne partagent pas le noyau avec l’hôte et les autres conteneurs exécutés sur l’hôte, ils peuvent exécuter des noyaux ayant des versions et des configurations différentes (dans les versions prises en charge): par exemple, tous les conteneurs Windows sur Windows10 utilisent l’isolation Hyper-V afin d’utiliser la version et la configuration du noyau WindowsServer.

## 3. Notions de base sur les conteneurs

Quand vous commencez à utiliser des conteneurs, vous remarquez de nombreuses similitudes entre un conteneur et une machine virtuelle. Un conteneur exécute un système d’exploitation, a un système de fichiers et est accessible via un réseau comme s’il s’agissait d’un système d’ordinateur physique ou virtuel. Ceci dit, la technologie et les concepts derrière les conteneurs sont très différents de ceux des machines virtuelles. Les concepts clés suivants peuvent s’avérer utiles quand vous commencez à créer des conteneurs Windows et à les utiliser. 

**Hôte de conteneur:** système informatique physique ou virtuel configuré avec la fonctionnalité de conteneur Windows.

**Image de système d’exploitation de conteneur:** les conteneurs sont déployés à partir d’images. L’image de système d’exploitation de conteneur est la première couche d’un nombre éventuellement important de couches d’images qui constituent un conteneur. Cette image fournit l’environnement du système d’exploitation.

**Image de conteneur:** une image de conteneur contient le système d’exploitation de base, l’application et toutes les dépendances d’application nécessaires pour déployer rapidement un conteneur. 

**Registre de conteneur:** les images de conteneur sont stockées dans un Registre de conteneur et peuvent être téléchargées à la demande. 

**Dockerfile:** les fichiers Dockerfile sont utilisés pour automatiser la création d’images de conteneur.

## Étape suivante:

[Démarrage rapide des conteneurs Windows Server](quick-start-windows-server.md)  

[Démarrage rapide des conteneurs Windows10](quick-start-windows-10.md)


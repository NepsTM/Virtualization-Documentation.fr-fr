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
translationtype: Human Translation
ms.sourcegitcommit: 54eff4bb74ac9f4dc870d6046654bf918eac9bb5
ms.openlocfilehash: d91f53908610ca245b6c8b4c017baed7f044667e

---

# Démarrage rapide des conteneurs Windows

La rubrique Démarrage rapide des conteneurs Windows présente la terminologie propre au produit et aux conteneurs, passe en revue des exemples de déploiement de conteneurs simples et fournit également des informations de référence concernant des rubriques plus avancées. Si vous n’êtes pas familiarisé avec les conteneurs ou les conteneurs Windows, suivez chaque étape de cette rubrique de démarrage rapide pour obtenir une expérience pratique de cette technologie.

## 1. Présentation des conteneurs

Ils constituent un environnement d’exploitation isolé, mobile et contrôlé par les ressources.

En fait, un conteneur est un emplacement isolé dans lequel une application peut s’exécuter sans affecter le reste du système et sans que le système n’affecte l’application. Les conteneurs sont la prochaine étape de la virtualisation.

Si vous étiez à l’intérieur d’un conteneur, vous vous sentiriez comme dans une machine virtuelle ou un ordinateur physique nouvellement installé. Enfin, pour [Docker](https://www.docker.com/), un conteneur Windows Server peut être géré de la même façon que n’importe quel autre conteneur.

## 2. Types de conteneurs Windows

Les conteneurs Windows incluent deux types de conteneurs différents, ou runtimes.

**Conteneurs Windows Server** : Ils assurent l’isolation des applications via une technologie d’isolation des processus et des espaces de noms. Un conteneur Windows Server partage un noyau avec l’hôte de conteneur et tous les conteneurs exécutés sur l’hôte.

**Conteneurs Hyper-V** : Ils développent l’isolation fournie par les conteneurs Windows Server en exécutant chaque conteneur dans une machine virtuelle hautement optimisée. Dans cette configuration, le noyau de l’hôte de conteneur n’est pas partagé avec d’autres conteneurs Hyper-V.

## 3. Notions de base sur les conteneurs

Quand vous commencez à utiliser des conteneurs, vous remarquez de nombreuses similitudes entre un conteneur et une machine virtuelle. Un conteneur exécute un système d’exploitation, a un système de fichiers et est accessible via un réseau comme s’il s’agissait d’un système d’ordinateur physique ou virtuel. Ceci dit, la technologie et les concepts derrière les conteneurs sont très différents de ceux des machines virtuelles. Les concepts clés suivants peuvent s’avérer utiles quand vous commencez à créer des conteneurs Windows et à les utiliser. 

**Hôte de conteneur :** système informatique physique ou virtuel configuré avec la fonctionnalité de conteneur Windows.

**Image de système d’exploitation de conteneur :** les conteneurs sont déployés à partir d’images. L’image de système d’exploitation de conteneur est la première couche d’un nombre éventuellement important de couches d’images qui constituent un conteneur. Cette image fournit l’environnement du système d’exploitation.

**Image de conteneur :** une image de conteneur contient le système d’exploitation de base, l’application et toutes les dépendances d’application nécessaires pour déployer rapidement un conteneur. 

**Registre de conteneur :** les images de conteneur sont stockées dans un Registre de conteneur et peuvent être téléchargées à la demande. 

**Dockerfile :** les fichiers Dockerfile sont utilisés pour automatiser la création d’images de conteneur.

## Étape suivante :

[Démarrage rapide des conteneurs Windows Server](quick-start-windows-server.md)  

[Démarrage rapide des conteneurs Windows 10](quick-start-windows-10.md)




<!--HONumber=Jan17_HO4-->



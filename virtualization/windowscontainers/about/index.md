---
title: About Windows Containers
description: Learn about Windows containers.
keywords: docker, containers
author: taylorb-microsoft
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 2be7a06c7b7b154e392c30981cdf954d2d1b796e
ms.sourcegitcommit: 8e193d8c274a549aef497f16dcdb00d7855e9fa7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/02/2017
---
# Windows Containers

## Présentation des conteneurs

Les conteneurs constituent un moyen d’incorporer une application dans son propre contenant. L’application qui se trouve dans le conteneur n’a aucune connaissance des autres applications ou processus pouvant exister à l’extérieur de ce contenant. Tout ce dont dépend l’application pour s’exécuter correctement se trouve également dans ce conteneur.  Quel que soit l’emplacement de ce dernier, l’application fonctionnera toujours correctement, car le package contient tous les éléments nécessaires à son exécution.

Prenons l’exemple d’une cuisine. Nous y rassemblons tous les appareils ménagers et les meubles, les ustensiles divers et la vaisselle, jusqu’au liquide vaisselle et aux torchons. Il s’agit là de notre conteneur.

<center style="margin: 25px">![](media/box1.png)</center>

Nous pouvons maintenant prendre ce conteneur et le déposer dans n’importe quel appartement et la cuisine sera identique. Il suffit d’effectuer les branchements électriques et les raccordements en eau et nous pouvons commencer à cuisiner (étant donné que nous disposons de tous les appareils dont nous avons besoin!).

<center style="margin: 25px">![](media/apartment.png)</center>

Les conteneurs ressemblent à cette cuisine. Il peut y avoir différents types de pièces et de nombreuses pièces peuvent être identiques. L’essentiel est que les conteneurs incluent tous les éléments nécessaires.

Regardez une présentation rapide ici: [Conteneurs Windows: développement d’applications modernes avec un contrôle de niveau professionnel](https://youtu.be/Ryx3o0rD5lY).

## Notions de base sur les conteneurs

Les conteneurs constituent un environnement d’exploitation isolé, mobile et contrôlé par les ressources, qui s’exécute sur un ordinateur hôte ou un ordinateur virtuel. Une application ou un processus qui s’exécute dans un conteneur inclut la totalité des dépendances et des fichiers de configuration requis. Il lui semble qu’aucun autre processus n’est en cours d’exécution à l’extérieur du conteneur.

L’hôte du conteneur configure un ensemble de ressources pour le conteneur et ce dernier utilise uniquement ces ressources. Du point de vue du conteneur, aucune autre ressource n’existe en dehors de celles qui lui ont été fournies et, par conséquent il n’a pas accès aux ressources susceptibles d’avoir été configurées pour un conteneur voisin.

Les concepts clés suivants peuvent s’avérer utiles quand vous commencez à créer des conteneurs Windows et à les utiliser.

**Container Host:** Physical or Virtual computer system configured with the Windows Container feature. L’hôte de conteneur exécute un ou plusieurs conteneurs Windows.

**Image de conteneur:** quand des modifications sont apportées au système de fichiers ou au Registre d’un conteneur, par exemple lors de l’installation d’un logiciel, elles sont capturées dans un bac à sable (sandbox). Dans de nombreux cas, vous pouvez capturer cet état pour que des conteneurs qui héritent de ces modifications puissent être créés. That’s what an image is – once the container has stopped you can either discard that sandbox or you can convert it into a new container image. For example, let’s imagine that you have deployed a container from the Windows Server Core OS image. You then install MySQL into this container. Creating a new image from this container would act as a deployable version of the container. This image would only contain the changes made (MySQL), however would work as a layer on top of the Container OS Image.

**Sandbox:** Once a container has been started, all write actions such as file system modifications, registry modifications or software installations are captured in this ‘sandbox’ layer.

**Container OS Image:** Containers are deployed from images. The container OS image is the first layer in potentially many image layers that make up a container. Cette image fournit l’environnement du système d’exploitation. Une image de système d’exploitation de conteneur est immuable. Autrement dit, elle ne peut pas être modifiée.

**Référentiel de conteneurs:** Chaque fois qu’une image de conteneur est créée, cette image et ses dépendances sont stockées dans un référentiel local. Ces images peuvent être réutilisées plusieurs fois sur l’hôte de conteneur. Les images de conteneur peuvent également être stockées dans un registre public ou privé, tel que DockerHub, afin de pouvoir être utilisées sur plusieurs hôtes de conteneurs différents.

<center>![](media/containerfund.png)</center>

Pour un utilisateur déjà familiarisé avec les machines virtuelles, les conteneurs peuvent sembler très similaires. A container runs an operating system, has a file system and can be accessed over a network just as if it was a physical or virtual computer system. Ceci dit, la technologie et les concepts derrière les conteneurs sont très différents de ceux des machines virtuelles.

Mark Russinovich, expert MicrosoftAzure, a rédigé [un excellent billet de blog](https://azure.microsoft.com/en-us/blog/containers-docker-windows-and-trends/) détaillant les différences.

## Types de conteneurs Windows

Windows Containers include two different container types, or runtimes.

**Windows Server Containers** – provide application isolation through process and namespace isolation technology. A Windows Server Container shares a kernel with the container host and all containers running on the host. Ces conteneurs ne créent pas de frontière de sécurité contre le code hostile et ne doivent pas être utilisés pour isoler du code non fiable. En raison de l’espace de noyau partagé, ces conteneurs requièrent la même version et configuration de noyau.

**Isolation Hyper-V**: Développe l’isolation fournie par les conteneurs WindowsServer en exécutant chaque conteneur dans une machine virtuelle hautement optimisée. In this configuration, the kernel of the container host is not shared with other containers on the same host. Ces conteneurs sont conçus pour un hébergement mutualisé hostile avec les mêmes garanties de sécurité qu’une machine virtuelle. Dans la mesure où ces conteneurs ne partagent pas le noyau avec l’hôte et les autres conteneurs exécutés sur l’hôte, ils peuvent exécuter des noyaux ayant des versions et des configurations différentes (dans les versions prises en charge): par exemple, tous les conteneurs Windows sur Windows10 utilisent l’isolation Hyper-V afin d’utiliser la version et la configuration du noyau WindowsServer.

L’exécution d’un conteneur sur Windows avec ou sans isolation Hyper-V est une décision d’exécution. Vous pouvez choisir de créer le conteneur avec l’isolation Hyper-V initialement, puis lors de l’exécution, de l’exécuter en tant conteneur WindowsServer à la place.

## Qu’est-ce que Docker?

À mesure que vous découvrirez les conteneurs, vous entendrez inévitablement parler de Docker. Docker est le moyen par lequel les images du conteneur sont mises en package et fournies. Ce processus automatisé génère des images (en réalité des modèles) qui peuvent ensuite être exécutées n’importe où: localement, dans le cloud ou sur un ordinateur personnel, en tant que conteneur.

<center>![](media/docker.png)</center>

Un conteneur Windows Server peut être géré avec [Docker](https://www.docker.com) comme tout autre conteneur.

## Conteneurs pour les développeurs ##

From a developer’s desktop to a testing machine to a set of production machines, a Docker image can be created that will deploy identically across any environment in seconds. This story has created a massive and growing ecosystem of applications packaged in Docker containers, with DockerHub, the public containerized-application registry that Docker maintains, currently publishing more than 180,000 applications in the public community repository.

When you containerize an app, only the app and the components needed to run the app are combined into an "image". Containers are then created from this image as you need them. You can also use an image as a baseline to create another image, making image creation even faster. Multiple containers can share the same image, which means containers start very quickly and use fewer resources. For example, you can use containers to spin up light-weight and portable app components – or ‘micro-services’ – for distributed apps and quickly scale each service separately.

Because the container has everything it needs to run your application, they are very portable and can run on any machine that is running Windows Server 2016. You can create and test containers locally, then deploy that same container image to your company's private cloud, public cloud or service provider. The natural agility of Containers supports modern app development patterns in large scale, virtualized and cloud environments.

With containers, developers can build an app in any language. These apps are completely portable and can run anywhere - laptop, desktop, server, private cloud, public cloud or service provider - without any code changes.  

Containers helps developers build and ship higher-quality applications, faster.

## Containers for IT Professionals ##

IT Professionals can use containers to provide standardized environments for their development, QA, and production teams. They no longer have to worry about complex installation and configuration steps. By using containers, systems administrators abstract away differences in OS installations and underlying infrastructure.

Containers help admins create an infrastructure that is simpler to update and maintain.

## Video Overview

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## Essayer les conteneurs WindowsServer

Vous êtes prêt à exploiter les impressionnantes fonctionnalités des conteneurs? Utilisez les liens ci-dessous pour commencer à déployer votre premier conteneur: <br/>
Pour les utilisateurs de WindowsServer, accédez à l’[introduction de démarrage rapide WindowsServer](../quick-start/quick-start-windows-server.md) <br/>
Pour les utilisateurs de Windows10, accédez à l’[introduction de démarrage rapide Windows10](../quick-start/quick-start-windows-10.md)


---
title: "À propos des conteneurs Windows"
description: En savoir plus sur les conteneurs Windows.
keywords: docker, conteneurs
author: taylorb-microsoft
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 37782c4d2491b9b1963a326204e30a6f484b5ec9
ms.sourcegitcommit: 6eefb890f090a6464119630bfbdc2794e6c3a3df
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2017
---
# <a name="windows-containers"></a>Conteneurs Windows

## <a name="what-are-containers"></a>Présentation des conteneurs

Les conteneurs constituent un moyen d’incorporer une application dans son propre contenant. L’application qui se trouve dans le conteneur n’a aucune connaissance des autres applications ou processus pouvant exister à l’extérieur de ce contenant. Tout ce dont dépend l’application pour s’exécuter correctement se trouve également dans ce conteneur.  Quel que soit l’emplacement de ce dernier, l’application fonctionnera toujours correctement, car le package contient tous les éléments nécessaires à son exécution.

Prenons l’exemple d’une cuisine. Nous y rassemblons tous les appareils ménagers et les meubles, les ustensiles divers et la vaisselle, jusqu’au liquide vaisselle et aux torchons. Il s’agit là de notre conteneur.

<center style="margin: 25px">![](media/box1.png)</center>

Nous pouvons maintenant prendre ce conteneur et le déposer dans n’importe quel appartement et la cuisine sera identique. Il suffit d’effectuer les branchements électriques et les raccordements en eau et nous pouvons commencer à cuisiner (étant donné que nous disposons de tous les appareils dont nous avons besoin!).

<center style="margin: 25px">![](media/apartment.png)</center>

Les conteneurs ressemblent à cette cuisine. Il peut y avoir différents types de pièces et de nombreuses pièces peuvent être identiques. L’essentiel est que les conteneurs incluent tous les éléments nécessaires.

Regardez une présentation rapide ici: [Conteneurs Windows: développement d’applications modernes avec un contrôle de niveau professionnel](https://youtu.be/Ryx3o0rD5lY).

## <a name="container-fundamentals"></a>Notions de base sur les conteneurs

Les conteneurs constituent un environnement d’exploitation isolé, mobile et contrôlé par les ressources, qui s’exécute sur un ordinateur hôte ou un ordinateur virtuel. Une application ou un processus qui s’exécute dans un conteneur inclut la totalité des dépendances et des fichiers de configuration requis. Il lui semble qu’aucun autre processus n’est en cours d’exécution à l’extérieur du conteneur.

L’hôte du conteneur configure un ensemble de ressources pour le conteneur et ce dernier utilise uniquement ces ressources. Du point de vue du conteneur, aucune autre ressource n’existe en dehors de celles qui lui ont été fournies et, par conséquent il n’a pas accès aux ressources susceptibles d’avoir été configurées pour un conteneur voisin.

Les concepts clés suivants peuvent s’avérer utiles quand vous commencez à créer des conteneurs Windows et à les utiliser.

**Hôte de conteneur:** système d’ordinateur physique ou virtuel configuré avec la fonctionnalité de conteneur Windows. L’hôte de conteneur exécute un ou plusieurs conteneurs Windows.

**Image de conteneur:** quand des modifications sont apportées au système de fichiers ou au Registre d’un conteneur, par exemple lors de l’installation d’un logiciel, elles sont capturées dans un bac à sable (sandbox). Dans de nombreux cas, vous pouvez capturer cet état pour que des conteneurs qui héritent de ces modifications puissent être créés. C’est ce qui constitue une image: une fois le conteneur arrêté, vous pouvez ignorer ce bac à sable (sandbox) ou vous pouvez le convertir en une nouvelle image de conteneur. Par exemple, imaginons que vous avez déployé un conteneur à partir de l’image de système d’exploitation Windows Server Core. Vous installez ensuite MySQL dans ce conteneur. La création d’une image à partir de ce conteneur fait office de version pouvant être déployée du conteneur. Cette image contient uniquement les modifications apportées (MySQL), mais fonctionne toutefois comme une couche sur l’image de système d’exploitation de conteneur.

**Bac à sable (sandbox):** Une fois un conteneur démarré, toutes les actions d’écriture, telles que les modifications du système de fichiers, les modifications du Registre ou les installations de logiciels, sont capturées dans cette couche.

**Image de système d’exploitation de conteneur:** Les conteneurs sont déployés à partir d’images. L’image de système d’exploitation de conteneur est la première couche d’un nombre éventuellement important de couches d’images qui constituent un conteneur. Cette image fournit l’environnement du système d’exploitation. Une image de système d’exploitation de conteneur est immuable. Autrement dit, elle ne peut pas être modifiée.

**Référentiel de conteneurs:** Chaque fois qu’une image de conteneur est créée, cette image et ses dépendances sont stockées dans un référentiel local. Ces images peuvent être réutilisées plusieurs fois sur l’hôte de conteneur. Les images de conteneur peuvent également être stockées dans un registre public ou privé, tel que DockerHub, afin de pouvoir être utilisées sur plusieurs hôtes de conteneurs différents.

<center>![](media/containerfund.png)</center>

Pour un utilisateur déjà familiarisé avec les machines virtuelles, les conteneurs peuvent sembler très similaires. Un conteneur exécute un système d’exploitation, a un système de fichiers et est accessible via un réseau comme s’il s’agissait d’un système d’ordinateur physique ou virtuel. Ceci dit, la technologie et les concepts derrière les conteneurs sont très différents de ceux des machines virtuelles.

Mark Russinovich, expert MicrosoftAzure, a rédigé [un excellent billet de blog](https://azure.microsoft.com/en-us/blog/containers-docker-windows-and-trends/) détaillant les différences.

## <a name="windows-container-types"></a>Types de conteneurs Windows

Les conteneurs Windows incluent deuxtypes de conteneurs différents, ou runtimes.

**Conteneurs Windows Server**: Ils assurent l’isolation des applications via une technologie d’isolation des processus et des espaces de noms. Un conteneur Windows Server partage un noyau avec l’hôte de conteneur et tous les conteneurs exécutés sur l’hôte. Ces conteneurs ne créent pas de frontière de sécurité contre le code hostile et ne doivent pas être utilisés pour isoler du code non fiable. En raison de l’espace de noyau partagé, ces conteneurs requièrent la même version et configuration de noyau.

**Isolation Hyper-V**: Développe l’isolation fournie par les conteneurs WindowsServer en exécutant chaque conteneur dans une machine virtuelle hautement optimisée. Dans cette configuration, le noyau de l’hôte de conteneur n’est pas partagé avec d’autres conteneurs exécutés sur l’hôte. Ces conteneurs sont conçus pour un hébergement mutualisé hostile avec les mêmes garanties de sécurité qu’une machine virtuelle. Dans la mesure où ces conteneurs ne partagent pas le noyau avec l’hôte et les autres conteneurs exécutés sur l’hôte, ils peuvent exécuter des noyaux ayant des versions et des configurations différentes (dans les versions prises en charge): par exemple, tous les conteneurs Windows sur Windows10 utilisent l’isolation Hyper-V afin d’utiliser la version et la configuration du noyau WindowsServer.

L’exécution d’un conteneur sur Windows avec ou sans isolation Hyper-V est une décision d’exécution. Vous pouvez choisir de créer le conteneur avec l’isolation Hyper-V initialement, puis lors de l’exécution, de l’exécuter en tant conteneur WindowsServer à la place.

## <a name="what-is-docker"></a>Qu’est-ce que Docker?

À mesure que vous découvrirez les conteneurs, vous entendrez inévitablement parler de Docker. Docker est le moyen par lequel les images du conteneur sont mises en package et fournies. Ce processus automatisé génère des images (en réalité des modèles) qui peuvent ensuite être exécutées n’importe où: localement, dans le cloud ou sur un ordinateur personnel, en tant que conteneur.

<center>![](media/docker.png)</center>

Un conteneur Windows Server peut être géré avec [Docker](https://www.docker.com) comme tout autre conteneur.

## <a name="containers-for-developers"></a>Conteneurs pour les développeurs ##

Qu’il s’agisse de l’ordinateur de bureau d’un développeur, d’un ordinateur de test ou d’un groupe de machines de production, il est possible de créer une image Docker qui sera déployée de façon identique dans tous les environnements en quelques secondes. De là est né un solide écosystème de plus en plus important d’applications empaquetées dans des conteneurs Docker avec Docker Hub, registre public d’applications en conteneur géré par Docker qui publie actuellement plus de 180000applications dans le référentiel de la communauté publique.

Quand vous mettez une application en conteneur, seule l’application et les composants nécessaires pour exécuter l’application sont combinés en une «image». Les conteneurs sont ensuite créés à partir de cette image si vous en avez besoin. Vous pouvez également utiliser une image comme référence pour créer une autre image, ce qui accélère encore davantage la création d’images. Plusieurs conteneurs peuvent partager la même image, ce qui signifie que les conteneurs démarrent très rapidement et utilisent moins de ressources. Par exemple, vous pouvez utiliser des conteneurs pour faire tourner des composants d’application légers et mobiles (ou «micro-services») pour les applications distribuées et adapter rapidement chaque service séparément.

Étant donné que le conteneur dispose de tout ce dont il a besoin pour exécuter votre application, il est très mobile et peut être exécuté sur n’importe quel ordinateur qui exécute Windows Server2016. Vous pouvez créer et tester localement des conteneurs, puis déployer cette même image de conteneur sur le cloud privé, le cloud public ou le fournisseur de services de votre société. De nature souple, les conteneurs prennent en charge les modèles de développement d’applications modernes dans des environnements cloud, virtualisés et à grande échelle.

Les conteneurs permettent aux développeurs de créer une application dans n’importe quel langage. Ces applications sont entièrement mobiles et peuvent être exécutées n’importe où (ordinateur portable, ordinateur de bureau, serveur, cloud privé, cloud public ou fournisseur de services) sans aucune modification du code.  

Les conteneurs aident les développeurs à générer et livrer des applications de meilleure qualité, plus rapidement.

## <a name="containers-for-it-professionals"></a>Conteneurs pour les professionnels de l’informatique ##

Les professionnels de l’informatique peuvent utiliser des conteneurs pour fournir des environnements standardisés à leurs équipes de développement, d’assurance qualité et de production. Ils n’ont plus à se soucier des procédures d’installation et de configuration complexes. En utilisant des conteneurs, les administrateurs système font abstraction des différences dans les installations du système d’exploitation et l’infrastructure sous-jacente.

Les conteneurs aident les administrateurs à créer une infrastructure qui est plus simple à mettre à jour et à gérer.

## <a name="video-overview"></a>Vidéo de présentation

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## <a name="try-windows-server-containers"></a>Essayer les conteneurs WindowsServer

Vous êtes prêt à exploiter les impressionnantes fonctionnalités des conteneurs? Utilisez les liens ci-dessous pour commencer à déployer votre premier conteneur: <br/>
Pour les utilisateurs de WindowsServer, accédez à l’[introduction de démarrage rapide WindowsServer](../quick-start/quick-start-windows-server.md) <br/>
Pour les utilisateurs de Windows10, accédez à l’[introduction de démarrage rapide Windows10](../quick-start/quick-start-windows-10.md)


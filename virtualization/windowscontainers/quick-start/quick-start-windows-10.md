---
title: Conteneurs Windows et Linux sur Windows 10
description: Démarrage rapide du déploiement de conteneurs
keywords: dockers, conteneurs, LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: ae311ecccdfbfc30b1079330a8eb02c1ce3ac94b
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/28/2019
ms.locfileid: "9680969"
---
# <a name="windows-containers-on-windows-10"></a>Conteneurs Windows sur Windows10

> [!div class="op_single_selector"]
> - [Conteneurs Linux sur Windows](quick-start-windows-10-linux.md)
> - [Conteneurs Windows sous Windows](quick-start-windows-10.md)

L’exercice va vous guider dans la création et l’exécution de conteneurs Windows sur Windows 10.

Dans ce démarrage rapide, vous allez effectuer les opérations suivantes:

1. Installation de la version de bureau de l’amarrage
2. Exécution d’un conteneur Windows simple

Ce démarrage rapide est spécifique à Windows10. Pour plus d’informations, reportez-vous à la documentation de démarrage rapide supplémentaire disponible dans la table des matières sur le côté gauche de cette page.

## <a name="prerequisites"></a>Prérequis
Veuillez vérifier que vous remplissez les conditions suivantes:
- Un système informatique physique exécutant Windows 10 professionnel ou entreprise avec la mise à jour anniversaire (version 1607) ou une version ultérieure. 
- Vérifiez que [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) est activé.

***Isolement Hyper-V:*** Les conteneurs Windows Server requièrent l’isolement Hyper-V sur Windows 10 afin de fournir aux développeurs la même version du noyau et la même configuration qui seront utilisées en production, en plus de l’isolation Hyper-V, qui se trouve dans la page [à propos du conteneur Windows](../about/index.md) .

> [!NOTE]
> Dans le cadre de la mise à jour de Windows octobre 2018, il n’est plus possible de ne pas autoriser les utilisateurs à exécuter un conteneur Windows en mode d’isolation de processus sur Windows 10 entreprise ou professionnel à des fins de développement et de test. Pour en savoir plus, voir le [Forum aux questions](../about/faq.md) .

## <a name="install-docker-desktop"></a>Installer le Bureau de l’ancrage

Téléchargez la version de bureau de l' [amarrage](https://store.docker.com/editions/community/docker-ce-desktop-windows) et exécutez le programme d’installation (vous devez vous connecter. Créez un compte si vous n’en avez pas encore. [Des instructions d’installation détaillées](https://docs.docker.com/docker-for-windows/install) sont disponibles dans la documentation de Docker.

## <a name="switch-to-windows-containers"></a>Basculer vers les conteneurs Windows

Après l’installation, l’option Bureau de l’ancrage utilise par défaut des conteneurs Linux. Basculez vers les conteneurs Windows à l’aide du menu de la barre d’état système ou en exécutant la commande suivante dans une invite PowerShell:

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

## <a name="install-base-container-images"></a>Installer les images de conteneur de base

Les conteneurs Windows sont créés à partir d’images de base. La commande suivante extraira l’image de base Nano Server.

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

Une fois l’image extraite, l’exécution de la commande `docker images` retourne la liste des images installées; dans le cas présent, l’image Nano Server.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Consultez le [CLUF](../images-eula.md)de l’image du système d’exploitation Windows.

## <a name="run-your-first-windows-container"></a>Exécuter votre premier conteneur Windows

Pour cet exemple simple, une image de conteneur «Hello World» sera créée et déployée. Pour une expérience optimale, exécutez ces commandes dans une interface de commande Windows ou PowerShell élevée.

> Windows PowerShell ISE ne fonctionne pas pour les sessions interactives avec des conteneurs. Même si le conteneur est en cours d’exécution, il apparaît comme étant bloqué.

Commencez par démarrer un conteneur avec une session interactive à partir de l’image `nanoserver`. Une fois que le conteneur a démarré, une interface de commande s’affiche à partir du conteneur.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

Dans le conteneur, nous allons créer un simple fichier texte «Hello World».

```cmd
echo "Hello World!" > Hello.txt
```   

Quand vous avez terminé, quittez le conteneur.

```cmd
exit
```

Vous allez maintenant créer une image de conteneur à partir du conteneur modifié. Pour afficher une liste de conteneurs, exécutez la commande suivante et notez l’ID de conteneur.

```console
docker ps -a
```

Exécutez la commande suivante pour créer l’image «Hello World». Remplacez <containerid> par l’ID de votre conteneur.

```console
docker commit <containerid> helloworld
```

Quand vous avez terminé, vous disposez d’une image personnalisée qui contient le script «Hello World». Pour l’afficher, utilisez la commande suivante.

```console
docker images
```

Enfin, pour exécuter le conteneur, utilisez la commande `docker run`.

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

Le résultat de la `docker run` commande est qu’un conteneur exécuté sous l’isolation Hyper-V a été créé à partir de l’image «HelloWorld», qu’une instance de cmd a été démarrée dans le conteneur et exécuté une lecture de notre fichier (sortie renvoyée vers l’interpréteur de commandes), puis que le conteneur a arrêté et supprimé.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez comment créer un exemple d’application](./building-sample-app.md)

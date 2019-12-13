---
title: Conteneurs Windows et Linux sur Windows 10
description: Démarrage rapide du déploiement de conteneurs
keywords: ancrage, conteneurs, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a664b5b8eb87adffdf7eba3ffca9f4194128df80
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909569"
---
# <a name="get-started-run-your-first-windows-container"></a>Prise en main : exécuter votre premier conteneur Windows

Cette rubrique décrit comment exécuter votre premier conteneur Windows, après la configuration de votre environnement, comme décrit dans [prise en main : préparer Windows pour les conteneurs](./set-up-environment.md). Pour exécuter un conteneur, vous devez d’abord installer une image de base qui fournit à votre conteneur une couche fondamentale de services de système d’exploitation. Ensuite, vous créez et exécutez une image de conteneur, qui est basée sur l’image de base. Pour plus d’informations, consultez.

## <a name="install-a-container-base-image"></a>Installer une image de base de conteneur

Tous les conteneurs sont créés à partir d’images de conteneur. Microsoft propose plusieurs images de démarrage, appelées images de base, à choisir. pour en savoir plus, consultez [images de base du conteneur](../manage-containers/container-base-images.md). Cette procédure extrait (télécharge et installe) l’image de base nano Server légère.

1. Ouvrez une fenêtre d’invite de commandes (telle que l’invite de commandes intégrée, PowerShell ou [Windows Terminal](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)), puis exécutez la commande suivante pour télécharger et installer l’image de base :

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > Si vous voyez s’afficher un message d’erreur `no matching manifest for unknown in the manifest list entries`, assurez-vous que la station d’accueil n’est pas configurée pour exécuter des conteneurs Linux.

2. Une fois le téléchargement de l’image terminé, lisez le [CLUF](../images-eula.md) en cours d’attente. Vérifiez qu’il existe sur votre système en interrogeant votre référentiel d’images de l’arrimeur local. L’exécution de la commande `docker images` retourne une liste d’images installées.

   Voici un exemple de la sortie qui affiche l’image nano Server.

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Exécuter un conteneur Windows

Pour cet exemple simple, une image de conteneur « Hello World » est créée et déployée. Pour une expérience optimale, exécutez ces commandes dans une fenêtre d’invite de commandes avec élévation de privilèges (mais n’utilisez pas la Windows PowerShell ISE, elle ne fonctionne pas pour les sessions interactives avec des conteneurs, car les conteneurs semblent se bloquer).

1. Démarrez un conteneur avec une session interactive à partir de l’image `nanoserver` en entrant la commande suivante dans la fenêtre d’invite de commandes :

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. Une fois le conteneur démarré, la fenêtre d’invite de commandes change de contexte dans le conteneur. À l’intérieur du conteneur, nous allons créer un fichier texte « Hello World » simple, puis quitter le conteneur en entrant les commandes suivantes :

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. Récupérez l’ID de conteneur pour le conteneur que vous venez de quitter en exécutant la commande [dockr PS](https://docs.docker.com/engine/reference/commandline/ps/) :

   ```console
   docker ps -a
   ```

4. Créez une nouvelle image « HelloWorld » qui comprend les modifications apportées au premier conteneur que vous avez exécuté. Pour ce faire, exécutez la commande [dockr commit](https://docs.docker.com/engine/reference/commandline/commit/) , en remplaçant `<containerid>` par l’ID de votre conteneur :

   ```console
   docker commit <containerid> helloworld
   ```

   Quand vous avez terminé, vous disposez d’une image personnalisée qui contient le script « Hello World ». Vous pouvez le voir avec la commande [dockers images](https://docs.docker.com/engine/reference/commandline/images/) .

   ```console
   docker images
   ```

   Voici un exemple de sortie :

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. Enfin, exécutez le nouveau conteneur à l’aide de la commande [dockr Run](https://docs.docker.com/engine/reference/commandline/run/) avec le paramètre `--rm` qui supprime automatiquement le conteneur une fois que la ligne de commande (cmd. exe) s’arrête.

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   Le résultat est qu’un conteneur a été créé à partir de l’image « HelloWorld », une instance de cmd. exe a été démarrée dans le conteneur qui lit notre fichier et génère le contenu du fichier dans le shell, puis le conteneur s’est arrêté et a été supprimé.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur la mise en conteneur d’un exemple d’application](./building-sample-app.md)

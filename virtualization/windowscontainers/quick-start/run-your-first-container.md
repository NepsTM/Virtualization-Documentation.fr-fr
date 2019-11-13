---
title: Conteneurs Windows et Linux sur Windows 10
description: Démarrage rapide du déploiement de conteneurs
keywords: dockers, conteneurs, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a664b5b8eb87adffdf7eba3ffca9f4194128df80
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288127"
---
# <a name="get-started-run-your-first-windows-container"></a>Commencer: exécuter votre premier conteneur Windows

Cette rubrique décrit comment exécuter votre premier conteneur Windows, après avoir configuré votre environnement comme décrit dans la rubrique mise en [route pour préparer Windows pour les conteneurs](./set-up-environment.md). Pour exécuter un conteneur, vous devez commencer par installer une image de base, qui fournit une couche de services de système d’exploitation à votre conteneur. Ensuite, vous créez et exécutez une image de conteneur, qui est basée sur l’image de base. Pour plus d’informations, poursuivez votre lecture.

## <a name="install-a-container-base-image"></a>Installer une image de base du conteneur

Tous les conteneurs sont créés à partir d’images de conteneurs. Microsoft propose plusieurs images de démarrage, appelées images de base, à partir desquelles vous pouvez effectuer votre choix (pour plus d’informations, consultez la section [images de base du conteneur](../manage-containers/container-base-images.md)). Cette procédure récupère (télécharge et installe) l’image de base nano Server légère.

1. Ouvrez une fenêtre d’invite de commandes (par exemple, une invite de commandes intégrée, PowerShell ou [terminal Windows](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)), puis exécutez la commande suivante pour télécharger et installer l’image de base:

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > Si un message d’erreur s’affiche `no matching manifest for unknown in the manifest list entries`, assurez-vous que la station d’accueil n’est pas configurée pour exécuter les conteneurs Linux.

2. Une fois le téléchargement de l’image terminée, lisez le [CLUF](../images-eula.md) lorsque vous patientez: Vérifiez qu’il existe sur votre système en interrogeant votre référentiel d’image de l’arrimeur local. L’exécution de `docker images` la commande renvoie une liste des images installées.

   Voici un exemple de sortie illustrant l’image nano Server.

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Exécuter un conteneur Windows

Pour cet exemple simple, une image de conteneur «Hello World» sera créée et déployée. Pour une utilisation optimale, exécutez les commandes suivantes dans une fenêtre d’invite de commandes avec élévation de privilèges (sans utiliser Windows PowerShell ISE), elle ne fonctionne pas pour les sessions interactives à l’aide de conteneurs, car ils semblent se bloquer.

1. Démarrez un conteneur avec une session interactive à partir `nanoserver` de l’image en entrant la commande suivante dans la fenêtre d’invite de commandes:

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. Une fois le conteneur démarré, la fenêtre d’invite de commandes change de contexte pour le conteneur. Dans le conteneur, nous allons créer un fichier texte «Hello World» simple, puis quitter le conteneur en entrant les commandes suivantes:

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. Obtenez l’ID de conteneur du conteneur que vous venez de quitter en exécutant la commande [dockr PS](https://docs.docker.com/engine/reference/commandline/ps/) :

   ```console
   docker ps -a
   ```

4. Créez une image «HelloWorld» incluant les modifications dans le premier conteneur que vous avez exécuté. Pour ce faire, exécutez la commande [Commit du docker](https://docs.docker.com/engine/reference/commandline/commit/) , `<containerid>` en remplaçant par l’ID de votre conteneur:

   ```console
   docker commit <containerid> helloworld
   ```

   Quand vous avez terminé, vous disposez d’une image personnalisée qui contient le script «Hello World». Pour cela, vous pouvez utiliser la commande images de l' [ancrage](https://docs.docker.com/engine/reference/commandline/images/) .

   ```console
   docker images
   ```

   Voici un exemple de sortie:

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. Enfin, exécutez le nouveau conteneur à l’aide de la commande exécuter de l' `--rm` [ancrer](https://docs.docker.com/engine/reference/commandline/run/) avec le paramètre qui supprime automatiquement le conteneur une fois la ligne de commande (cmd. exe) arrêtée.

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   Le résultat est qu’un conteneur a été créé à partir de l’image’HelloWorld', qu’une instance de cmd. exe a été démarrée dans le conteneur qui a lu notre fichier et qu’il a copié le contenu du fichier sur le shell, puis que le conteneur est arrêté et supprimé.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrir comment conteneurr un exemple d’application](./building-sample-app.md)

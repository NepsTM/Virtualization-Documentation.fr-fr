---
title: Conteneurs Windows et Linux sur Windows 10
description: Démarrage rapide du déploiement de conteneurs
keywords: dockers, conteneurs, LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 3d651a4a68acefa25f1b647b1b33618bbfb91ae9
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129363"
---
# <a name="get-started-run-your-first-container"></a>Commencer: exécuter votre premier conteneur

Dans le [segment précédent](./set-up-environment.md), nous avons configuré notre environnement pour les conteneurs en cours d’exécution. Cet exercice montre comment extraire une image de conteneur et l’exécuter.

## <a name="install-container-base-image"></a>Image de la base du conteneur d’installation

Les conteneurs sont instanciés à `container images`partir de. Microsoft propose plusieurs images d’accueil (appelées `base images`) à votre choix. La commande suivante extraira l’image de base Nano Server.

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

> [!TIP]
> Si un message d’erreur s’affiche `no matching manifest for unknown in the manifest list entries`, assurez-vous que la station d’accueil n’est pas configurée pour exécuter les conteneurs Linux.

Après l’extraction de l’image, vous pouvez vérifier son existence sur votre ordinateur en interrogeant votre référentiel d’image d’ancrage local. L’exécution de `docker images` la commande renvoie une liste des images installées, dans le cas présent, l’image nano Server.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Consultez le [CLUF](../images-eula.md)de l’image du système d’exploitation Windows.

## <a name="run-your-first-windows-container"></a>Exécuter votre premier conteneur Windows

Pour cet exemple simple, une image de conteneur «Hello World» sera créée et déployée. Pour une utilisation optimale, exécutez les commandes suivantes dans un interpréteur de commande Windows avec élévation de privilèges ou PowerShell.

> Windows PowerShell ISE ne fonctionne pas pour les sessions interactives avec des conteneurs. Même si le conteneur est en cours d’exécution, il apparaît comme étant bloqué.

Commencez par démarrer un conteneur avec une session interactive à partir de l’image `nanoserver`. Lorsque le conteneur est démarré, un interpréteur de commandes s’affiche à l’intérieur du conteneur.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

À l’intérieur du conteneur, nous allons créer un simple fichier texte «Hello World».

```cmd
echo "Hello World!" > Hello.txt
```   

Quand vous avez terminé, quittez le conteneur.

```cmd
exit
```

Créer une nouvelle image de conteneur à partir du conteneur modifié. Pour afficher une liste des conteneurs qui s’exécutent ou sont en sortie, exécutez les éléments suivants et prenez note de l’ID de conteneur.

```console
docker ps -a
```

Exécutez la commande suivante pour créer l’image «Hello World». Remplacez `<containerid>` par l’ID de votre conteneur.

```console
docker commit <containerid> helloworld
```

Quand vous avez terminé, vous disposez d’une image personnalisée qui contient le script «Hello World». Pour l’afficher, utilisez la commande suivante.

```console
docker images
```

Enfin, exécutez le conteneur à l’aide `docker run` de la commande.

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

Le résultat de la `docker run` commande est qu’un conteneur a été créé à partir de l’image «HelloWorld», qu’une instance de cmd a été démarrée dans le conteneur et a exécuté une lecture de notre fichier (sortie renvoyée vers l’interpréteur de commandes), puis que le conteneur est arrêté et supprimé.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrir comment conteneurr un exemple d’application](./building-sample-app.md)

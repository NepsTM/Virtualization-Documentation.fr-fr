---
title: Windows et des conteneurs Linux sur Windows 10
description: Démarrage rapide du déploiement de conteneurs
keywords: docker, conteneurs, LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 07f5929505226a50a161b4ae7df5669c2ad89d83
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9575520"
---
# <a name="windows-containers-on-windows-10"></a>Conteneurs Windows sur Windows10

> [!div class="op_single_selector"]
> - [Conteneurs Linux sur Windows](quick-start-windows-10-linux.md)
> - [Conteneurs Windows sur Windows](quick-start-windows-10.md)

L’exercice guideront à travers la création et l’exécution de conteneurs Windows sur Windows 10.

Dans ce démarrage rapide vous allez accomplir:

1. L’installation de Docker pour Windows
2. Exécution d’un conteneur Windows simple

Ce démarrage rapide est spécifique à Windows10. Vous trouverez la documentation de démarrage rapide supplémentaire dans la table des matières sur le côté gauche de cette page.

## <a name="prerequisites"></a>Prérequis
Vérifiez que vous respectez les exigences suivantes:
- Un système d’ordinateur physique exécutant Windows 10 Professionnel ou entreprise avec la mise à jour anniversaire (version 1607) ou version ultérieure. 
- Assurez-vous que [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) est activé.

***Isolation Hyper-V:*** Conteneurs Windows Server requièrent l’isolation Hyper-V sur Windows 10 afin de fournir aux développeurs la même version du noyau et configuration qui sera utilisée en production, plus à propos d’Hyper-V isolation sont accessibles sur la page à [propos des conteneurs Windows](../about/index.md) .

> [!NOTE]
> Dans la version de Windows octobre mise à jour 2018, nous n’est plus ne pas autoriser les utilisateurs à partir de l’exécution d’un conteneur Windows en mode d’isolation des processus sur Windows 10 entreprise ou Professionnel à des fins de développement/test. Consultez le [Forum aux questions](../about/faq.md) pour en savoir plus.

## <a name="install-docker-for-windows"></a>Installer Docker pour Windows

Téléchargez [Docker pour Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) et exécutez le programme d’installation (vous devrez ouvrir une session. Créez un compte si vous n’avez pas déjà). [Des instructions d’installation détaillées](https://docs.docker.com/docker-for-windows/install) sont disponibles dans la documentation de Docker.

## <a name="switch-to-windows-containers"></a>Basculer vers les fenêtres de conteneurs

Après l’installation, Docker pour Windows exécute par défaut des conteneurs Linux. Basculez vers les conteneurs Windows à l’aide du menu de barre d’état de Docker ou en exécutant la commande suivante dans un script PowerShell invite:

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
> Veuillez lire l' image de système d’exploitation [CLUF](../images-eula.md)les conteneurs Windows.

## <a name="run-your-first-windows-container"></a>Exécuter votre premier conteneur Windows

Dans cet exemple simple, une image de conteneur «Hello World» sera créée et déployée. Pour une expérience optimale, exécutez ces commandes dans une interface de commande Windows ou PowerShell élevée.

> Windows PowerShell ISE ne fonctionne pas pour les sessions interactives avec des conteneurs. Même si le conteneur est en cours d’exécution, il apparaît comme étant bloqué.

Commencez par démarrer un conteneur avec une session interactive à partir de l’image `nanoserver`. Une fois que le conteneur a démarré, une interface de commande s’affiche à partir du conteneur.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

À l’intérieur du conteneur, nous allons créer un fichier de texte «Hello World» simple.

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

Le résultat de la `docker run` commande est qu’un conteneur en cours d’exécution sous isolation Hyper-V a été créé à partir de l’image «Hello World», une instance de cmd a été démarrée dans le conteneur et exécuté une lecture de notre fichier (sortie répercuté dans l’interpréteur de commandes), puis le conteneur arrêtée et supprimée.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez comment créer un exemple d’application](./building-sample-app.md)
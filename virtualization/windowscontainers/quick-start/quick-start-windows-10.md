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
ms.openlocfilehash: 4202908f8797a2b98ab657c45cd9a6b33191bd6f
ms.sourcegitcommit: 9dfef8d261f4650f47e8137a029e893ed4433a86
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2018
ms.locfileid: "6224898"
---
# <a name="windows-and-linux-containers-on-windows-10"></a>Windows et des conteneurs Linux sur Windows 10

L’exercice guideront à travers la création et l’exécution de conteneurs Windows et Linux sur Windows 10. Une fois terminé, vous aurez:

1. Installé Docker pour Windows
2. Exécuter un conteneur Windows simple
3. Exécuter un conteneur simple de Linux à l’aide de conteneurs Linux sur Windows (LCOW)

Ce démarrage rapide est spécifique à Windows10. Vous trouverez la documentation de démarrage rapide supplémentaire dans la table des matières sur le côté gauche de cette page.

***Isolation Hyper-V:*** Conteneurs Windows Server requièrent l’isolation Hyper-V sur Windows 10 afin de fournir aux développeurs la même version du noyau et configuration qui sera utilisée en production, plus sur Hyper-V isolation sont accessibles sur la page à [propos des conteneurs Windows](../about/index.md) .

**Éléments requis:**

- Un système d’ordinateur physique exécutant Windows 10 Fall Creators Update (version 1709) ou version ultérieure (Professionnel ou entreprise) qui [peuvent s’exécuter Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)

> Si vous n’avez pas besoin pour exécuter LCOW dans ce didacticiel, les conteneurs Windows sera en mesure de s’exécuter sur la mise à jour anniversaire Windows 10 (version 1607) ou version ultérieure.

## <a name="1-install-docker-for-windows"></a>1. Installer Docker pour Windows

[Téléchargez le Docker pour Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) et exécutez le programme d’installation (vous serez requis pour vous connecter. Créer un compte si vous n’avez pas déjà). [Des instructions d’installation détaillées](https://docs.docker.com/docker-for-windows/install) sont disponibles dans la documentation de Docker.

> Si vous avez déjà installé de Docker, vérifiez que vous disposez 18.02 ou version ultérieure pour prendre en charge LCOW. Vérifier en exécutant `docker -v` ou la vérification *Sur Docker*.

> Option des «expérimental les fonctionnalités» dans *Docker Paramètres > démon* doit être activé pour exécuter des conteneurs LCOW.

## <a name="2-switch-to-windows-containers"></a>2. Basculer vers les conteneurs Windows

Après l’installation, Docker pour Windows exécute par défaut des conteneurs Linux. Basculez vers des conteneurs Windows à l’aide du menu de la barre d’état de Docker ou en exécutant la commande suivante dans une invite PowerShell `& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon`.

![](./media/docker-for-win-switch.png)
> Notez que le mode de conteneurs Windows permet de conteneurs LCOW en plus des conteneurs Windows.

## <a name="3-install-base-container-images"></a>3. Installer les images de conteneur de base

Les conteneurs Windows sont créés à partir d’images de base. La commande suivante extraira l’image de base Nano Server.

```
docker pull microsoft/nanoserver
```

Une fois l’image extraite, l’exécution de la commande `docker images` retourne la liste des images installées; dans le cas présent, l’image Nano Server.

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> Veuillez lire le Contrat de Licence Utilisateur Final (CLUF) applicable à l’image de système d’exploitation des conteneurs Windows, disponibles ici [CLUF](../images-eula.md).

## <a name="4-run-your-first-windows-container"></a>4. exécuter votre premier conteneur Windows

Pour ce simple exemple, une image de conteneur «Hello World» est créée et déployée. Pour une expérience optimale, exécutez ces commandes dans une interface de commande Windows ou PowerShell élevée.
> Windows PowerShell ISE ne fonctionne pas pour les sessions interactives avec des conteneurs. Même si le conteneur est en cours d’exécution, il apparaît comme étant bloqué.

Commencez par démarrer un conteneur avec une session interactive à partir de l’image `nanoserver`. Une fois que le conteneur a démarré, une interface de commande s’affiche à partir du conteneur.  

```
docker run -it microsoft/nanoserver cmd
```

À l’intérieur du conteneur, nous allons créer un script «Hello World» simple.

```
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

Quand vous avez terminé, quittez le conteneur.

```
exit
```

Vous allez maintenant créer une image de conteneur à partir du conteneur modifié. Pour afficher une liste de conteneurs, exécutez la commande suivante et notez l’ID de conteneur.

```
docker ps -a
```

Exécutez la commande suivante pour créer l’image «Hello World». Remplacez <containerid> par l’ID de votre conteneur.

```
docker commit <containerid> helloworld
```

Quand vous avez terminé, vous disposez d’une image personnalisée qui contient le script «Hello World». Pour l’afficher, utilisez la commande suivante.

```
docker images
```

Enfin, pour exécuter le conteneur, utilisez la commande `docker run`.

```
docker run --rm helloworld powershell c:\helloworld.ps1
```

Le résultat de la commande `docker run` est qu’un conteneur Hyper-V a été créé à partir de l’image «Hello World», un exemple de script «Hello World» a été exécuté (sortie répercutée sur l’interface de commande), puis le conteneur s’est arrêté et a été supprimé.
Les démarrages rapides suivants de Windows10 et des conteneurs exploreront la création et le déploiement d’applications dans des conteneurs sur Windows10.

## <a name="run-your-first-lcow-container"></a>Exécuter votre premier conteneur LCOW

Pour cet exemple, un conteneur BusyBox sera déployé. Tout d’abord, essayez d’exécuter une image «Hello World» BusyBox.

```
docker run --rm busybox echo hello_world
```

Notez que cela retourne une erreur lorsque Docker tente d’extraire l’image. Cela se produit car Dockers nécessite une directive via le `--platform` indicateur pour confirmer que le système d’exploitation image et hôte sont correctement mis en correspondance. Dans la mesure où la plateforme par défaut en mode de conteneur Windows est Windows, ajoutez un `--platform linux` indicateur pour extraire et exécuter le conteneur.

```
docker run --rm --platform linux busybox echo hello_world
```

Une fois que l’image a été retirée avec la plateforme indiquée, le `--platform` indicateur n’est plus nécessaire. Exécutez la commande sans qu’il pour effectuer ce test.

```
docker run --rm busybox echo hello_world
```

Exécutez `docker images` pour retourner une liste d’images installées. Dans ce cas, les images Windows et Linux.

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

## <a name="next-steps"></a>Étapes suivantes

Bonus: Voir de correspondante de Docker [billet de blog](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) sur l’exécution LCOW

Passez au didacticiel suivant pour voir un exemple de [création d’un exemple d’application](./building-sample-app.md)

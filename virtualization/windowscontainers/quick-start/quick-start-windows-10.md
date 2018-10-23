---
title: Conteneur Windows sur Windows10
description: Démarrage rapide du déploiement de conteneurs
keywords: docker, conteneurs
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: aa69739697e9e2c668aaf5026c6f5ddc69d43c4d
ms.sourcegitcommit: f6f53fd0da65ac44d16fb793c5aa1af56c14d90e
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2018
ms.locfileid: "5428991"
---
# <a name="windows-containers-on-windows-10"></a>Conteneurs Windows sur Windows10

L’exercice vous guide tout au long du déploiement et de l’utilisation de base de la fonctionnalité de conteneur Windows10 Professionnel ou Entreprise (Édition anniversaire). Lorsque vous l'aurez terminé, vous aurez installé Docker pour Windows et exécuté un conteneur simple. Si vous voulez vous familiariser avec les conteneurs, vous trouverez des informations dans la rubrique [À propos des conteneurs](../about/index.md).

Ce démarrage rapide est spécifique à Windows10. Une documentation supplémentaire du démarrage rapide est disponible dans la table des matières affichée sur la gauche de cette page.

***Isolation Hyper-V:*** les conteneurs WindowsServer requièrent l’isolation Hyper-V sur Windows10 afin de fournir aux développeurs la même version et configuration de noyau que celle utilisée en production. Pour plus d’informations à ce sujet, voir la page [À propos des conteneurs Windows](../about/index.md).

**Conditions préalables:**

- Un système d’ordinateur physique exécutant Windows10 Édition anniversaire ou Creators Update (Professionnel ou Entreprise).   
- Ce démarrage rapide peut être exécuté sur une machine virtuelle Windows10. Toutefois, la virtualisation imbriquée doit être activée. Pour plus d’informations, voir le [Guide de la virtualisation imbriquée](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

> Vous devez installer les mises à jour critiques pour que les conteneurs Windows fonctionnent.
> Pour vérifier la version de votre système d’exploitation, exécutez `winver.exe` et comparez la version indiquée dans [l’historique des mises à jour Windows10](https://support.microsoft.com/en-us/help/12387/windows-10-update-history).

> Assurez-vous d’avoir la version14393.222 ou ultérieure avant de continuer.  Cette version correspond à Windows 10 version 1607, afin que n’importe quelle version ci-dessus 1607 doit être entièrement pris en charge.

## <a name="1-install-docker-for-windows"></a>1. Installer Docker pour Windows

[Téléchargez Docker pour Windows](https://download.docker.com/win/stable/InstallDocker.msi) et exécutez le programme d’installation. [Des instructions d’installation détaillées](https://docs.docker.com/docker-for-windows/install) sont disponibles dans la documentation de Docker.

## <a name="2-switch-to-windows-containers"></a>2. Basculer vers les conteneurs Windows

Après l’installation, Docker pour Windows exécute par défaut des conteneurs Linux. Basculez vers des conteneurs Windows à l’aide du menu de la barre d’état de Docker ou en exécutant la commande suivante dans une invite PowerShell `& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon`.

![](./media/docker-for-win-switch.png)

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

## <a name="4-run-your-first-container"></a>4. Exécuter votre premier conteneur

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

## <a name="next-steps"></a>Étapes suivantes

Passez au didacticiel suivant pour voir un exemple de [création d’un exemple d’application](./building-sample-app.md)

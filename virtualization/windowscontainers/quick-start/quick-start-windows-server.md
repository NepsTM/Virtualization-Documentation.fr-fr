---
title: Conteneurs Windows sur Windows Server
description: "Démarrage rapide du déploiement de conteneurs"
keywords: docker, conteneurs
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 76e041aac426604280208f616f7994181112215a
ms.openlocfilehash: 766a99a74738fa41ef77410c70aefa7e664f014e
ms.lasthandoff: 03/01/2017

---

# Conteneurs Windows sur Windows Server

Cet exercice vous guide dans le déploiement et l’utilisation de base de la fonctionnalité de conteneur Windows sur Windows Server 2016. Au cours de cet exercice, vous installez le rôle de conteneur et vous déployez un conteneur Windows Server simple. Avant de commencer ce démarrage rapide, familiarisez-vous avec la terminologie et les concepts de base des conteneurs. Vous trouverez ces informations dans la [Présentation du démarrage rapide](./index.md).

Ce démarrage rapide est spécifique aux conteneurs Windows Server sur Windows Server 2016. Une documentation de démarrage rapide supplémentaire, incluant les conteneurs dans Windows 10, est disponible dans la table des matières affichée à gauche dans cette page.

**Conditions préalables :**

Un système informatique (physique ou virtuel) exécutant Windows Server 2016. Si vous utilisez Windows Server 2016 TP5, effectuez une mise à jour vers la [version d’évaluation de Windows Server 2016](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ). 

> Les mises à jour critiques sont nécessaires au fonctionnement de la fonctionnalité Conteneur Windows. Installez toutes les mises à jour avant de suivre ce didacticiel.

Si vous souhaitez effectuer un déploiement sur Azure, ce [modèle](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template) peut vous aider.<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>


## 1. Installer Docker

Pour installer Docker, nous allons utiliser le [module PowerShell de fournisseur OneGet](https://github.com/oneget/oneget). Le fournisseur active la fonctionnalité des conteneurs sur votre ordinateur. Vous installez également Docker, qui nécessite un redémarrage. Docker est nécessaire pour utiliser les conteneurs Windows. Il est constitué du moteur Docker et du client Docker.

Ouvrez une session PowerShell avec élévation de privilèges, puis exécutez les commandes suivantes.

Commencez par installer le fournisseur Docker-Microsoft PackageManagement à partir de la galerie PowerShell.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Utilisez ensuite le module PowerShell PackageManagement pour installer la dernière version de Docker.
```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Quand PowerShell vous demande si la source du package « DockerDefault » doit être approuvée, tapez `A` pour poursuivre l’installation. Une fois l’installation terminée, redémarrez l’ordinateur.

```none
Restart-Computer -Force
```

## 2. Installer les mises à jour Windows

Vérifiez que votre système Windows Server est à jour en exécutant :

```none
sconfig
```

Un menu de configuration de type texte s’affiche, dans lequel vous pouvez choisir l’option 6 pour télécharger et installer les mises à jour :

```none
===============================================================================
                         Server Configuration
===============================================================================

1) Domain/Workgroup:                    Workgroup:  WORKGROUP
2) Computer Name:                       WIN-HEFDK4V68M5
3) Add Local Administrator
4) Configure Remote Management          Enabled

5) Windows Update Settings:             DownloadOnly
6) Download and Install Updates
7) Remote Desktop:                      Disabled
...
```

Lorsque vous y êtes invité, choisissez l’option A pour télécharger toutes les mises à jour.

## 3. Déployer votre premier conteneur

Dans cet exercice, vous téléchargez un exemple d’image .NET préalablement créée à partir du Registre Docker Hub, puis vous déployez un conteneur simple qui exécute une application .NET « Hello World ».  

Utilisez la commande `docker run` pour déployer le conteneur .Net. Cette opération télécharge également l’image de conteneur, ce qui peut prendre plusieurs minutes.

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver
```

Le conteneur démarre, affiche le message « Hello World », puis se ferme.

```console
         Dotnet-bot: Welcome to using .NET Core!
    __________________
                      \
                       \
                          ....
                          ....'
                           ....
                        ..........
                    .............'..'..
                 ................'..'.....
               .......'..........'..'..'....
              ........'..........'..'..'.....
             .'....'..'..........'..'.......'.
             .'..................'...   ......
             .  ......'.........         .....
             .                           ......
            ..    .            ..        ......
           ....       .                 .......
           ......  .......          ............
            ................  ......................
            ........................'................
           ......................'..'......    .......
        .........................'..'.....       .......
     ........    ..'.............'..'....      ..........
   ..'..'...      ...............'.......      ..........
  ...'......     ...... ..........  ......         .......
 ...........   .......              ........        ......
.......        '...'.'.              '.'.'.'         ....
.......       .....'..               ..'.....
   ..       ..........               ..'........
          ............               ..............
         .............               '..............
        ...........'..              .'.'............
       ...............              .'.'.............
      .............'..               ..'..'...........
      ...............                 .'..............
       .........                        ..............
        .....


**Environment**
Platform: .NET Core 1.0
OS: Microsoft Windows 10.0.14393
```

Pour obtenir des informations détaillées sur la commande Docker Run, voir [Docker Run Reference]( https://docs.docker.com/engine/reference/run/) sur Docker.com.

## Étapes suivantes

[Images de conteneur sur Windows Server](./quick-start-images.md)

[Conteneurs Windows sur Windows 10](./quick-start-windows-10.md)


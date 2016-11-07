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
ms.sourcegitcommit: 13a33a6003cfa77d81a6e8e2afdf1da8d017ee30
ms.openlocfilehash: a5537a4cb56127e51e0a550220995cd5f6b93722

---

# Conteneurs Windows sur Windows Server

Cet exercice vous guide lors du déploiement et l’utilisation de base de la fonctionnalité de conteneur Windows sur Windows Server. Une fois terminé, vous aurez installé le rôle de conteneur et déployé un conteneur Windows Server simple. Avant de commencer ce démarrage rapide, familiarisez-vous avec la terminologie et les concepts de base des conteneurs. Ces informations figurent dans la [Présentation du démarrage rapide](./quick_start.md).

Ce démarrage rapide est spécifique aux conteneurs Windows Server sur Windows Server 2016. Une documentation de démarrage rapide supplémentaire est disponible dans la table des matières affichée à gauche dans cette page.

**Conditions préalables :**

Un système informatique (physique ou virtuel) exécutant Windows Server 2016. Si vous utilisez Windows Server 2016 TP5, effectuez une mise à jour vers la [version d’évaluation de Windows Server 2016](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ). 

> Les mises à jour critiques sont nécessaires au fonctionnement de la fonctionnalité Conteneur Windows. Installez toutes les mises à jour avant de suivre ce didacticiel.

## 1. Installer Docker

Pour installer Docker, nous allons utiliser le [module PowerShell de fournisseur OneGet](https://github.com/oneget/oneget). Le fournisseur active la fonctionnalité de conteneurs sur votre ordinateur et installe Docker. Cette opération nécessite un redémarrage. Docker est nécessaire pour utiliser les conteneurs Windows. Il est constitué du moteur Docker et du client Docker.

Ouvrez une session PowerShell avec élévation de privilèges, puis exécutez les commandes suivantes.

Nous allons d’abord installer le module PowerShell OneGet.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Ensuite, nous utilisons OneGet pour installer la dernière version de Docker.
```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Quand PowerShell vous demande si la source du package « DockerDefault » doit être approuvée, tapez A pour poursuivre l’installation. Une fois l’installation terminée, redémarrez l’ordinateur.

```none
Restart-Computer -Force
```

## 2. Installer les mises à jour Windows

Pour être sûr que votre système Windows Server est à jour, vous devez installer les mises à jour Windows en exécutant :

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

Dans cet exercice, vous allez télécharger un exemple d’image .NET préalablement créée à partir du Registre Docker Hub, puis déployer un conteneur simple qui exécute une application .Net « Hello World ».  

Utilisez la commande `docker run` pour déployer le conteneur .Net. Cette opération télécharge également l’image de conteneur, ce qui peut prendre plusieurs minutes.

```none
docker run microsoft/sample-dotnet
```

Ensuite, le conteneur démarre, affiche le message « Hello World » et se ferme.

```none
       Welcome to .NET Core!
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
```

Pour obtenir des informations détaillées sur la commande Docker Run, voir [Docker Run Reference]( https://docs.docker.com/engine/reference/run/) sur Docker.com.

## Étapes suivantes

[Images de conteneur sur Windows Server](./quick_start_images.md)

[Conteneurs Windows sur Windows 10](./quick_start_windows_10.md)



<!--HONumber=Nov16_HO1-->



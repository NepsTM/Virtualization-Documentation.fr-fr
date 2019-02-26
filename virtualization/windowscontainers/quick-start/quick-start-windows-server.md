---
title: Conteneurs Windows sur Windows Server
description: Démarrage rapide du déploiement de conteneurs
keywords: docker, conteneurs
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
ms.openlocfilehash: 44e415af08881d18ebb2d82f5f79098f7fb034f8
ms.sourcegitcommit: f172d14ef1ebfb5a9df1b3529e0722d9103e0eba
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/26/2019
ms.locfileid: "9112926"
---
# <a name="windows-containers-on-windows-server"></a>Conteneurs Windows sur Windows Server

Cet exercice vous guide par le biais de déploiement de base et l’utilisation de la fonctionnalité de conteneur Windows sur Windows Server 2019 et Windows Server 2016.

Dans ce démarrage rapide vous allez accomplir:

1. L’activation de la fonctionnalité de conteneurs dans Windows Server
2. L’installation de Docker
3. Exécution d’un conteneur Windows simple

Si vous voulez vous familiariser avec les conteneurs, vous trouverez des informations dans la rubrique [À propos des conteneurs](../about/index.md).

Ce démarrage rapide est spécifique aux conteneurs Windows Server sur Windows Server 2019 et Windows Server 2016. Une documentation de démarrage rapide supplémentaire, incluant les conteneurs dans Windows10, est disponible dans la table des matières affichée à gauche dans cette page.

## <a name="prerequisites"></a>Conditions préalables

Vérifiez que vous respectez les exigences suivantes:
- Un système informatique (physique ou virtuel) exécutant Windows Server 2019. Si vous utilisez Windows Server 2019 Insider Preview, mettez à jour vers la [Version d’évaluation de fenêtre Server 2019](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019 ).

> Les mises à jour critiques sont nécessaires au fonctionnement de la fonctionnalité Conteneur Windows. Installez toutes les mises à jour avant de suivre ce didacticiel.

Si vous souhaitez effectuer un déploiement sur Azure, ce [modèle](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template) peut vous aider.

<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>


## <a name="install-docker"></a>Installer Docker

Pour installer Docker, nous allons utiliser le [module PowerShell de fournisseur OneGet](https://github.com/oneget/oneget) qui fonctionne avec les fournisseurs pour effectuer l’installation--dans ce cas, le [MicrosoftDockerProvider](https://github.com/OneGet/MicrosoftDockerProvider). Le fournisseur active la fonctionnalité de conteneurs sur votre ordinateur. Vous installez également Docker, qui nécessite un redémarrage. Docker est nécessaire pour utiliser les conteneurs Windows. Il est constitué du moteur Docker et du client Docker.

Ouvrez une session PowerShell avec élévation de privilèges, puis exécutez les commandes suivantes.

Commencez par installer le fournisseur Docker-Microsoft PackageManagement à partir de la [galerie PowerShell](https://www.powershellgallery.com/packages/DockerMsftProvider).

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Utilisez ensuite le module PowerShell PackageManagement pour installer la dernière version de Docker.

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Quand PowerShell vous demande si la source du package «DockerDefault» doit être approuvée, tapez`A` pour poursuivre l’installation. Une fois l’installation terminée, redémarrez l’ordinateur.

```powershell
Restart-Computer -Force
```

> ! [CONSEIL] Si vous souhaitez mettre à jour Docker ultérieurement:
>  - Vérifiez la version installée avec `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Trouvez la version actuelle avec `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Lorsque vous êtes prêt, procédez à la mise à niveau `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, suivie de `Start-Service Docker`

## <a name="install-windows-updates"></a>Installer les mises à jour Windows

Vérifiez que votre système Windows Server est à jour en exécutant:

```console
sconfig
```

Un menu de configuration de type texte s’affiche, dans lequel vous pouvez choisir l’option6 pour télécharger et installer les mises à jour:

```console
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

## <a name="deploy-your-first-container"></a>Déployer votre premier conteneur

Dans cet exercice, vous téléchargez un exemple d’image.NET préalablement créée à partir du Registre Docker Hub, puis vous déployez un conteneur simple qui exécute une application .NET «Hello World».  

Utilisez la commande `docker run` pour déployer le conteneur.Net. Cette opération télécharge également l’image de conteneur, ce qui peut prendre plusieurs minutes. En fonction de la version de votre hôte de Windows Server, exécutez la commande suivante ci-dessous.

#### <a name="windows-server-2019"></a>Windows Server2019

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-1809
```

#### <a name="windows-server-2016"></a>Windows Server2016

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-sac2016
```

Le conteneur démarre, affiche le message «Hello World», puis se ferme.

```console
         Hello from .NET Core!
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
Platform: .NET Core
OS: Microsoft Windows 10.0.17763
```

Pour obtenir des informations détaillées sur la commande DockerRun, voir [Docker Run Reference]( https://docs.docker.com/engine/reference/run/) sur Docker.com.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez comment automatiser les builds de conteneur et enregistrer des images](./quick-start-images.md)

---
title: Conteneurs Windows sur Windows Server
description: "Démarrage rapide du déploiement de conteneurs"
keywords: docker, conteneurs
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 891c9e9805bf2089fd11f86420de5ed251916c3f
ms.openlocfilehash: 77dca1499abf406b1d599c28afdb19dd823b8401

---

# Conteneurs Windows sur Windows Server

Cet exercice vous guide lors du déploiement et l’utilisation de base de la fonctionnalité de conteneur Windows sur Windows Server. Une fois terminé, vous aurez installé le rôle de conteneur et déployé un conteneur Windows Server simple. Avant de commencer ce démarrage rapide, familiarisez-vous avec la terminologie et les concepts de base des conteneurs. Ces informations figurent dans la [Présentation du démarrage rapide](./quick_start.md).

Ce démarrage rapide est spécifique aux conteneurs Windows Server sur Windows Server 2016. Une documentation de démarrage rapide supplémentaire est disponible dans la table des matières affichée à gauche dans cette page.

**Conditions préalables :**

Un système informatique (physique ou virtuel) exécutant Windows Server 2016. Si vous utilisez Windows Server 2016 TP5, effectuez une mise à jour vers la [version d’évaluation de Windows Server 2016](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ). 

## 1. Installer la fonctionnalité de conteneur

La fonctionnalité de conteneur doit être activée avant d’utiliser des conteneurs Windows. Pour ce faire, exécutez la commande suivante dans une session PowerShell avec élévation de privilèges.

```none
Install-WindowsFeature containers
```

Une fois l’installation de la fonctionnalité terminée, redémarrez l’ordinateur.

```none
Restart-Computer -Force
```

## 2. Installer Docker

Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. Pour cet exercice, les deux seront installés.

Téléchargez la version Release Candidate du moteur et du client Docker pris en charge, fournis sous forme d’archive ZIP.

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Développez l’archive zip dans Program Files.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Ajoutez le répertoire Docker au chemin du système.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Pour installer Docker en tant que service Windows, exécutez la commande suivante.

```none
dockerd.exe --register-service
```

Une fois installé, le service peut être démarré.

```none
Start-Service docker
```

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


<!--HONumber=Sep16_HO4-->



---
title: Optimiser les fichiers Dockerfile Windows
description: Optimisez des fichiers Dockerfile pour les conteneurs Windows.
keywords: docker, conteneurs
author: cwilhit
ms.date: 05/03/2019
ms.topic: tutorial
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
ms.openlocfilehash: 1d6858a4aee8c968ad6b1f27ffad97a7b454bcc3
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87984913"
---
# <a name="optimize-windows-dockerfiles"></a>Optimiser les fichiers Dockerfile Windows

De nombreuses méthodes permettent d’optimiser le processus de génération Docker et les images Docker qui en résultent. Cet article explique comment fonctionne le processus de génération Docker et comment créer des images optimales pour les conteneurs Windows.

## <a name="image-layers-in-docker-build"></a>Couches d’image dans la génération Docker

Pour optimiser la génération Docker, vous devez en comprendre le fonctionnement. Pendant le processus de génération Docker, un fichier Dockerfile est utilisé, et chaque instruction nécessitant une action est exécutée, l’une après l’autre, dans son propre conteneur temporaire. Le résultat est une nouvelle couche d’image pour chaque instruction nécessitant une action.

L’exemple de fichier Docker suivant utilise l’image du système d’exploitation de base `mcr.microsoft.com/windows/servercore:ltsc2019`, installe IIS, puis crée un site web simple.

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Vous vous attendez peut-être à ce que ce fichier Dockerfile produise une image à deux couches, l'une pour l’image de système d’exploitation de conteneur et l'autre comprenant IIS et le site web. L’image réelle présente cependant de nombreuses couches, chaque couche dépendant de celle qui la précède.

Pour plus de clarté, nous allons exécuter la commande `docker history` sur l’image de notre exemple de fichier Dockerfile.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

Il en résulte une image à quatre couches : la couche de base et trois couches supplémentaires mappées à chaque instruction du fichier Dockerfile. La couche inférieure (`6801d964fda5` dans cet exemple) représente l’image du système d’exploitation de base. La couche au-dessus correspond à l’installation IIS. La couche suivante inclut le nouveau site web, etc.

Des fichiers Dockerfile peuvent être écrits pour réduire les couches d’image, optimiser les performances de génération, ainsi qu’optimiser l'accessibilité par le biais de la lisibilité. Enfin, il existe de nombreuses façons d’effectuer la même tâche de génération d’image. Comprendre comment le format d’un fichier Dockerfile affecte le moment de la génération et l’image créée améliore l’automatisation.

## <a name="optimize-image-size"></a>Optimiser la taille des images

Selon l'espace disque requis, la taille de l’image peut être un facteur important lors de la génération d’images de conteneur Docker. Les images de conteneur sont déplacées entre les registres et l’hôte, sont exportées et importées, et finalement consomment de l’espace. Cette section vous explique comment réduire la taille de l’image lors du processus de génération Docker pour les conteneurs Windows.

Pour plus d’informations sur les bonnes pratiques Dockerfile, voir [Best practices for writing Dockerfiles sur Docker.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Regrouper des actions connexes

Étant donné que chaque instruction `RUN` crée une couche dans l’image de conteneur, le fait de regrouper des actions en une seule instruction `RUN` peut réduire le nombre de couches dans un fichier Dockerfile. La réduction des couches peut ne pas avoir beaucoup d’impact sur la taille des images, contrairement au regroupement d’actions connexes. Cela sera présenté dans des exemples ultérieurs.

Dans cette section, nous allons comparer deux exemples de fichiers Dockerfile effectuant les mêmes opérations. À cette différence près, un fichier Dockerfile dispose d'une instruction par action et l'autre de ses propres actions connexes regroupées.

L’exemple non groupé de fichier Dockerfile suivant télécharge Python pour Windows, l’installe et supprime le fichier d’installation téléchargé au terme de l'installation. Dans ce fichier Dockerfile, chaque action reçoit sa propre  instruction `RUN`.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

L’image obtenue se compose de trois couches supplémentaires, une pour chaque instruction `RUN`.

```dockerfile
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

Le deuxième exemple illustre un fichier Dockerfile effectuant exactement la même opération. Toutefois, toutes les actions connexes ont été regroupées sous une seule instruction `RUN`. Chaque étape dans l’instruction `RUN` se trouve sur une nouvelle ligne du fichier Dockerfile. Le caractère « \\ » est utilisé pour créer un retour automatique à la ligne.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

L’image obtenue ici se compose d’une seule couche supplémentaire pour l’instruction `RUN`.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Supprimer l’excédent de fichiers

En présence d'un fichier dans votre Dockerfile, tel qu’un programme d’installation dont vous n'avez plus besoin après utilisation, vous pouvez le supprimer pour réduire la taille de l’image. Cette opération doit se produire dans la même étape que celle de la copie du fichier dans la couche de l’image. Cela évite que le fichier persiste dans une couche d’image de niveau inférieur.

Dans l’exemple de fichier Dockerfile suivant, le package Python est téléchargé, exécuté, puis supprimé. Tout cela est effectué dans une seule opération `RUN` et génère une seule couche d’image.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>Optimiser la vitesse de génération

### <a name="multiple-lines"></a>Plusieurs lignes

Vous pouvez fractionner les opérations en plusieurs instructions individuelles afin d'optimiser la vitesse de génération Docker. Plusieurs opérations `RUN` augmentent l’efficacité de la mise en cache car des couches individuelles sont créées pour chaque instruction `RUN`. Si une instruction identique a déjà été exécutée dans une autre opération de génération Docker, cette opération mise en cache (couche d’image) est réutilisée, diminuant ainsi le runtime de génération Docker.

Dans l’exemple suivant, Apache et les packages redistribuables Visual Studio sont téléchargés et installés, puis nettoyés moyennant la suppression des fichiers devenus inutiles. Tout cela se fait à l'aide d'une seule instruction `RUN`. Si l’une de ces actions est mise à jour, toutes les actions sont réexécutées.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \

  # Download software ; \

  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \

  # Install Software ; \

  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \

  # Remove unneeded files ; \

  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force; \
  Remove-Item c:\php.zip
```

L’image obtenue comprend deux couches : l'une pour l’image du système d’exploitation de base et l’autre contenant toutes les opérations de l’instruction `RUN` unique.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

À titre de comparaison, voici les mêmes actions réparties dans trois instructions `RUN`. Dans ce cas, chaque instruction `RUN` est mise en cache dans une couche d’image de conteneur, et seules celles qui ont été modifiées doivent être réexécutées sur les builds Dockerfile suivantes.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
    Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
    Remove-Item c:\apache.zip -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
    start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist.exe -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
    Remove-Item c:\php.zip -Force
```

L’image obtenue se compose de quatre couches : une pour l’image du système d’exploitation de base et une pour chacune des trois instructions `RUN`. Étant donné que chaque instruction `RUN` a été exécutée dans sa propre couche, les exécutions suivantes de ce fichier Dockerfile ou d’un ensemble identique d’instructions d’un autre fichier Dockerfile utilisent la couche d’image mise en cache, réduisant ainsi le temps de génération.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

Comme vous pourrez le constater dans la section suivante, la façon dont vous classez les instructions est importante lorsque vous utilisez des caches d’image.

### <a name="ordering-instructions"></a>Classement des instructions

Un fichier Dockerfile est traité de haut en bas, chaque Instruction étant comparée aux couches mises en cache. Quand aucune couche mise en cache n’est trouvée pour une instruction, cette dernière et toutes les instructions suivantes sont traitées dans de nouvelles couches d’image de conteneur. C’est pourquoi l’ordre dans lequel les instructions sont placées est important. Placez les instructions qui resteront constantes en haut du fichier Dockerfile. Placez les instructions qui peuvent changer en bas du fichier Dockerfile. Cela réduit la probabilité d’annuler un cache existant.

Les exemples suivants illustrent la manière dont le classement des instructions Dockerfile peut affecter l’efficacité de la mise en cache. Cet exemple Dockerfile simple présente quatre dossiers numérotés.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

L’image obtenue comprend cinq couches : une pour l’image du système d’exploitation de base, et une pour chacune des instructions `RUN`.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

Le fichier Dockerfile suivant a été légèrement modifié, la troisième instruction `RUN` ayant été remplacée par un nouveau fichier. Quand la génération Docker est exécutée sur ce fichier Dockerfile, les trois premières instructions, qui sont identiques à celles de l’exemple précédent, utilisent les couches d’image mises en cache. Toutefois, l’instruction `RUN` modifiée n'étant pas mise en cache, une nouvelle couche est créée pour celle-ci et pour toutes les instructions suivantes.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Lorsque vous comparez les ID d’image de la nouvelle image avec le premier exemple de cette section, vous remarquez que les trois premières couches de bas en haut sont partagées, mais que les quatrième et cinquième sont uniques.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>Optimisation dynamique

### <a name="instruction-case"></a>Casse des instructions

Les instructions Dockerfile ne sont pas sensibles à la casse, mais il convient d'utiliser des majuscules. Cela améliore la lisibilité en distinguant l’appel d’instructions et l’opération d’instructions. Les deux exemples suivants comparent un fichier Dockerfile sans majuscules et avec majuscules.

Voici un fichier Dockerfile sans majuscules :

```dockerfile
# Sample Dockerfile

from mcr.microsoft.com/windows/servercore:ltsc2019
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

Voici le même fichier Dockerfile avec majuscules :

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Retour automatique à la ligne

Les opérations longues et complexes peuvent être réparties sur plusieurs lignes à l’aide du caractère barre oblique inverse (`\`). Le fichier Dockerfile suivant installe le package redistribuable de Visual Studio, supprime les fichiers du programme d’installation, puis crée un fichier de configuration. Ces trois opérations sont toutes spécifiées sur une seule ligne.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

La commande peut être divisée à l'aide de barres obliques inverses afin que chaque opération de l’instruction unique `RUN` soit spécifiée sur sa propre ligne.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Informations et références supplémentaires

[Fichier Dockerfile sur Windows](manage-windows-dockerfile.md)

[Best practices for writing Dockerfiles sur Docker.com](https://docs.docker.com/engine/reference/builder/)

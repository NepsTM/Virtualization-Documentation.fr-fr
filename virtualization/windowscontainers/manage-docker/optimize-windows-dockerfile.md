---
title: Optimiser les fichiers Dockerfile Windows
description: Optimisez des fichiers Dockerfile pour les conteneurs Windows.
keywords: docker, conteneurs
author: cwilhit
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
ms.openlocfilehash: ae633c7ba5d9672335addcc582988fc47c13ed79
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910149"
---
# <a name="optimize-windows-dockerfiles"></a>Optimiser les fichiers Dockerfile Windows

Il existe de nombreuses façons d’optimiser le processus de génération de l’ancrage et les images d’ancrage qui en résultent. Cet article explique comment fonctionne le processus de génération de l’amarrage et comment créer des images optimales pour les conteneurs Windows.

## <a name="image-layers-in-docker-build"></a>Couches d’image dans la génération de l’ancrage

Avant de pouvoir optimiser la génération de votre dockers, vous devez savoir comment fonctionne la build de l’Assistant. Pendant le processus de génération Docker, un fichier Dockerfile est utilisé, et chaque instruction nécessitant une action est exécutée, l’une après l’autre, dans son propre conteneur temporaire. Le résultat est une nouvelle couche d’image pour chaque instruction nécessitant une action.

Par exemple, l’exemple suivant fichier dockerfile utilise l’image du système d’exploitation de base `mcr.microsoft.com/windows/servercore:ltsc2019`, installe IIS, puis crée un site Web simple.

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Vous vous attendez peut-être à ce que ce fichier dockerfile produise une image avec deux couches, une pour l’image de système d’exploitation de conteneur et une seconde qui comprend IIS et le site Web. Toutefois, l’image réelle a de nombreuses couches, et chaque couche dépend de celle qui le précède.

Pour le rendre plus clair, nous allons exécuter la commande `docker history` sur l’image de notre exemple de fichier dockerfile.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

La sortie montre que cette image a quatre couches : la couche de base et trois couches supplémentaires qui sont mappées à chaque instruction du fichier dockerfile. La couche inférieure (`6801d964fda5` dans cet exemple) représente l’image du système d’exploitation de base. Une couche est l’installation d’IIS. La couche suivante inclut le nouveau site web, etc.

Les fichiers dockerfile peuvent être écrits pour réduire les couches d’images, optimiser les performances de génération et optimiser l’accessibilité grâce à la lisibilité. Enfin, il existe de nombreuses façons d’effectuer la même tâche de génération d’image. Comprendre comment le format de fichier dockerfile affecte le temps de génération et l’image qu’il crée améliore l’expérience d’automatisation.

## <a name="optimize-image-size"></a>Optimiser la taille de l’image

Selon vos besoins en espace, la taille de l’image peut être un facteur important lors de la création d’images de conteneur d’ancrage. Les images de conteneur sont déplacées entre les registres et l’hôte, sont exportées et importées, et finalement consomment de l’espace. Cette section vous indique comment réduire la taille de l’image pendant le processus de génération de l’amarrage pour les conteneurs Windows.

Pour plus d’informations sur les meilleures pratiques relatives à fichier dockerfile, consultez [meilleures pratiques pour l’écriture de fichiers dockerfile sur docker.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Regrouper des actions connexes

Étant donné que chaque instruction `RUN` crée une couche dans l’image de conteneur, le regroupement des actions en une seule instruction `RUN` peut réduire le nombre de couches dans un fichier dockerfile. La réduction des couches peut ne pas avoir beaucoup d’impact sur la taille des images, contrairement au regroupement d’actions connexes. Cela sera présenté dans des exemples ultérieurs.

Dans cette section, nous allons comparer deux exemples de fichiers dockerfile qui effectuent les mêmes opérations. Toutefois, un fichier dockerfile a une instruction par action, tandis que l’autre avait ses actions associées regroupées.

L’exemple non groupé suivant fichier dockerfile télécharge Python pour Windows, l’installe et supprime le fichier d’installation téléchargé une fois l’installation terminée. Dans ce fichier dockerfile, chaque action reçoit sa propre `RUN` instruction.

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

Le deuxième exemple est un fichier dockerfile qui effectue exactement la même opération. Toutefois, toutes les actions associées ont été regroupées sous une seule instruction `RUN`. Chaque étape de l’instruction `RUN` se trouve sur une nouvelle ligne du fichier dockerfile, tandis que le caractère «\\» est utilisé pour le retour à la ligne.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

L’image résultante n’a qu’une seule couche supplémentaire pour l’instruction `RUN`.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Supprimer l’excédent de fichiers

Si un fichier de votre fichier dockerfile, tel qu’un programme d’installation, n’est pas nécessaire après son utilisation, vous pouvez le supprimer pour réduire la taille de l’image. Cette opération doit se produire dans la même étape que celle de la copie du fichier dans la couche de l’image. Cela permet d’éviter que le fichier ne soit conservé dans une couche d’image de niveau inférieur.

Dans l’exemple suivant fichier dockerfile, le package Python est téléchargé, exécuté, puis supprimé. Tout cela est effectué dans une seule opération `RUN` et génère une seule couche d’image.

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

Vous pouvez fractionner les opérations en plusieurs instructions individuelles pour optimiser la vitesse de génération de l’ancrage. Plusieurs opérations de `RUN` augmentent l’efficacité de la mise en cache, car des couches individuelles sont créées pour chaque instruction `RUN`. Si une instruction identique a déjà été exécutée dans une autre opération de génération de l’ancrage, cette opération mise en cache (couche d’image) est réutilisée, ce qui entraîne une diminution du runtime de build de l’ancrage.

Dans l’exemple suivant, Apache et les packages redistribuables de Visual Studio sont téléchargés, installés, puis nettoyés en supprimant les fichiers qui ne sont plus nécessaires. Tout cela est effectué avec une seule instruction `RUN`. Si l’une de ces actions est mise à jour, toutes les actions sont réexécutées.

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

L’image résultante a deux couches : une pour l’image du système d’exploitation de base et une qui contient toutes les opérations de l’instruction de `RUN` unique.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

Par comparaison, Voici les mêmes actions qui sont divisées en trois instructions `RUN`. Dans ce cas, chaque instruction `RUN` est mise en cache dans une couche d’image de conteneur, et seules celles qui ont été modifiées doivent être réexécutées sur les builds fichier dockerfile suivantes.

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

L’image obtenue se compose de quatre couches ; une couche pour l’image du système d’exploitation de base et chacune des trois instructions `RUN`. Étant donné que chaque `RUN` instruction s’est exécutée dans sa propre couche, toutes les exécutions ultérieures de ce fichier dockerfile ou d’un ensemble d’instructions identique dans un autre fichier dockerfile utiliseront des couches d’images mises en cache, ce qui réduit le temps de génération.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

La façon dont vous commandez les instructions est importante lorsque vous utilisez des caches d’images, comme vous le verrez dans la section suivante.

### <a name="ordering-instructions"></a>Instructions de classement

Un fichier Dockerfile est traité de haut en bas, chaque Instruction étant comparée aux couches mises en cache. Quand aucune couche mise en cache n’est trouvée pour une instruction, cette dernière et toutes les instructions suivantes sont traitées dans de nouvelles couches d’image de conteneur. C’est pourquoi l’ordre dans lequel les instructions sont placées est important. Placez les instructions qui resteront constantes en haut du fichier Dockerfile. Placez les instructions qui peuvent changer en bas du fichier Dockerfile. Cela réduit la probabilité d’annuler un cache existant.

Les exemples suivants montrent comment le classement des instructions fichier dockerfile peut affecter l’efficacité de la mise en cache. Cet exemple simple fichier dockerfile contient quatre dossiers numérotés.  

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

L’image obtenue comporte cinq couches, une pour l’image du système d’exploitation de base et chacune des instructions de `RUN`.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

Cette fichier dockerfile suivante a été légèrement modifiée, avec la troisième `RUN` instruction remplacée par un nouveau fichier. Quand la génération Docker est exécutée sur ce fichier Dockerfile, les trois premières instructions, qui sont identiques à celles de l’exemple précédent, utilisent les couches d’image mises en cache. Toutefois, étant donné que l’instruction `RUN` modifiée n’est pas mise en cache, une nouvelle couche est créée pour l’instruction modifiée et toutes les instructions suivantes.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Lorsque vous comparez les ID d’image de la nouvelle image à celle du premier exemple de cette section, vous remarquerez que les trois premières couches de bas en haut sont partagées, mais que les quatrième et cinquième sont uniques.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>Optimisation cosmétique

### <a name="instruction-case"></a>Cas d’instruction

Les instructions fichier dockerfile ne respectent pas la casse, mais la Convention consiste à utiliser des majuscules. Cela améliore la lisibilité en différenciant l’appel d’instruction et l’opération d’instruction. Les deux exemples suivants comparent un fichier dockerfile non écrit et en majuscules.

Voici un fichier dockerfile non en majuscules :

```dockerfile
# Sample Dockerfile

from mcr.microsoft.com/windows/servercore:ltsc2019
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

Voici le même fichier dockerfile en majuscules :

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Retour à la ligne

Les opérations longues et complexes peuvent être séparées sur plusieurs lignes par la barre oblique inverse `\` caractère. Le fichier Dockerfile suivant installe le package redistribuable de Visual Studio, supprime les fichiers du programme d’installation, puis crée un fichier de configuration. Ces trois opérations sont toutes spécifiées sur une seule ligne.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

La commande peut être divisée par des barres obliques inverses afin que chaque opération de l’une `RUN` instruction soit spécifiée sur sa propre ligne.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Informations supplémentaires sur la lecture et les références

[Fichier dockerfile sur Windows](manage-windows-dockerfile.md)

[Meilleures pratiques pour l’écriture de fichiers dockerfile sur Docker.com](https://docs.docker.com/engine/reference/builder/)

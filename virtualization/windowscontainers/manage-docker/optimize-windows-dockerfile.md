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
ms.openlocfilehash: d897560061fae23fda6f88ebdad6dd804da9a8f1
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610339"
---
# <a name="optimize-windows-dockerfiles"></a>Optimiser les fichiers Dockerfile Windows

Il existe plusieurs façons d’optimiser le processus de génération Docker et les images Docker qui en résulte. Cet article explique comment fonctionne le processus de génération Docker et créer des images pour les conteneurs Windows de façon optimale.

## <a name="image-layers-in-docker-build"></a>Couches d’image dans la build Docker

Avant de pouvoir optimiser votre version de Docker, vous devez savoir comment la génération Docker fonctionne. Pendant le processus de génération Docker, un fichier Dockerfile est utilisé, et chaque instruction nécessitant une action est exécutée, l’une après l’autre, dans son propre conteneur temporaire. Le résultat est une nouvelle couche d’image pour chaque instruction nécessitant une action.

Par exemple, l’exemple suivant Dockerfile utilise le `windowsservercore` image de base du système d’exploitation, installe IIS et crée ensuite un site Web simple.

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Vous pouvez vous attendre que ce fichier Dockerfile produira une image avec deux couches, une pour l’image de système d’exploitation de conteneur et une autre qui inclut IIS et le site Web. Toutefois, l’image proprement dite a de nombreuses couches, et chaque couche varie en fonction de celle avant lui.

Pour le rendre plus clair, nous allons exécuter le `docker history` commande par rapport à l’image de notre exemple de fichier Dockerfile apportée.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

La sortie montre cependant que cette image dispose de quatre couches: la couche de base et trois couches supplémentaires qui sont mappés à chaque instruction dans le fichier Dockerfile. La couche inférieure (`6801d964fda5` dans cet exemple) représente l’image du système d’exploitation de base. Une couche est l’installation d’IIS. La couche suivante inclut le nouveau site web, etc.

Fichiers Dockerfile peut être écrits pour réduire les couches d’image, optimiser les performances de génération et d’optimiser l’accessibilité par le biais d’une meilleure lisibilité. Enfin, il existe de nombreuses façons d’effectuer la même tâche de génération d’image. Comprendre comment format de fichier Dockerfile affecte le moment de la génération et l’image créée améliore l’automatisation.

## <a name="optimize-image-size"></a>Optimiser la taille de l’image

En fonction de vos besoins en espace, taille de l’image peut être un facteur important lors de la création des images de conteneur Docker. Les images de conteneur sont déplacées entre les registres et l’hôte, sont exportées et importées, et finalement consomment de l’espace. Cette section vous indique comment réduire la taille de l’image pendant le processus de génération Docker pour les conteneurs Windows.

Pour plus d’informations sur les bonnes pratiques Dockerfile, voir les [meilleures pratiques pour écrire des fichiers Dockerfile sur Docker.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Regrouper des actions connexes

Dans la mesure où chaque `RUN` instruction crée une nouvelle couche dans l’image de conteneur, au regroupement d’actions en un seul `RUN` instruction peut réduire le nombre de couches dans un fichier Dockerfile. La réduction des couches peut ne pas avoir beaucoup d’impact sur la taille des images, contrairement au regroupement d’actions connexes. Cela sera présenté dans des exemples ultérieurs.

Dans cette section, nous allons comparer deux exemples de fichiers Dockerfile que les mêmes opérations de faire. Toutefois, un fichier Dockerfile a une seule instruction par action, tandis que l’autre avait ses actions connexes regroupées.

L’exemple suivant dissociée Dockerfile télécharge Python pour Windows, il installe et supprime le fichier d’installation téléchargé une fois que l’installation est terminée. Dans ce fichier Dockerfile, chaque action est fournie dans son propre `RUN` instruction.

```dockerfile
FROM windowsservercore

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

Le deuxième exemple est un fichier Dockerfile qui effectue la même opération exacte. Toutefois, tous les associés actions ont été regroupées sous une seule `RUN` instruction. Chaque étape de la `RUN` instruction se trouve dans une nouvelle ligne du fichier Dockerfile, tandis que le ' \\' caractère est utilisé pour créer un retour automatique à la.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

L’image obtenue a uniquement une couche supplémentaire pour le `RUN` instruction.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Supprimer l’excédent de fichiers

S’il existe un fichier dans votre fichier Dockerfile, par exemple, un programme d’installation, vous n’avez pas besoin après qu’il est été utilisé, vous pouvez supprimer afin de réduire la taille de l’image. Cette opération doit se produire dans la même étape que celle de la copie du fichier dans la couche de l’image. Cela empêche le fichier persiste dans une couche d’image de bas niveau.

Dans l’exemple suivant Dockerfile, le package Python est téléchargé, exécuté, puis supprimé. Tout cela est effectué dans une seule opération `RUN` et génère une seule couche d’image.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>Optimiser la vitesse de génération

### <a name="multiple-lines"></a>Plusieurs lignes

Vous pouvez diviser les opérations en plusieurs instructions individuelles pour optimiser la vitesse de génération Docker. Plusieurs `RUN` opérations augmentent l’efficacité de la mise en cache, car des couches individuelles sont créés pour chaque `RUN` instruction. Si une instruction identique a été déjà exécutée dans une autre opération de génération Docker, cette opération de mise en cache (couche d’image) est réutilisée, ce qui entraîne une diminution runtime de génération Docker.

Dans l’exemple suivant, Apache et les packages redistribuables Visual Studio sont téléchargées, installées et nettoyés en supprimant les fichiers qui ne sont plus nécessaires. Tout cela est effectué avec un seul `RUN` instruction. Si une de ces actions sont mis à jour, toutes les actions seront exécuté à nouveau.

```dockerfile
FROM windowsservercore

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

L’image obtenue a deux couches, une pour l’image du système d’exploitation de base et celui qui contient toutes les opérations de la valeur simple `RUN` instruction.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

Par comparaison, voici le même fractionnement d’actions en trois `RUN` instructions. Dans ce cas, chaque `RUN` instruction est mis en cache dans une couche d’image de conteneur, et seules celles qui ont besoin d’ont été modifié pour être exécuté à nouveau sur le fichier Dockerfile ultérieur builds.

```dockerfile
FROM windowsservercore

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

L’image obtenue se compose de quatre couches; une couche pour l’image du système d’exploitation de base et chacune des trois `RUN` instructions. Dans la mesure où chaque `RUN` instruction a été exécuté dans sa propre couche, les exécutions suivantes de ce fichier Dockerfile ou un ensemble identique d’instructions dans un autre fichier Dockerfile utilisera les couches d’image mises en cache, réduisant ainsi le moment de la génération.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

L’ordre des instructions est important quand vous travaillez avec les caches d’image, comme vous le verrez dans la section suivante.

### <a name="ordering-instructions"></a>Classement des instructions

Un fichier Dockerfile est traité de haut en bas, chaque Instruction étant comparée aux couches mises en cache. Quand aucune couche mise en cache n’est trouvée pour une instruction, cette dernière et toutes les instructions suivantes sont traitées dans de nouvelles couches d’image de conteneur. C’est pourquoi l’ordre dans lequel les instructions sont placées est important. Placez les instructions qui resteront constantes en haut du fichier Dockerfile. Placez les instructions qui peuvent changer en bas du fichier Dockerfile. Cela réduit la probabilité d’annuler un cache existant.

Les exemples suivants montrent comment le classement des instructions Dockerfile peut affecter l’efficacité de la mise en cache. Cet exemple simple de fichier Dockerfile a quatre dossiers numérotés.  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

L’image obtenue a cinq couches, une pour l’image du système d’exploitation de base et chacun de la `RUN` instructions.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

Ce fichier Dockerfile suivant a été légèrement modifié, avec la troisième `RUN` instruction a été remplacée par un nouveau fichier. Quand la génération Docker est exécutée sur ce fichier Dockerfile, les trois premières instructions, qui sont identiques à celles de l’exemple précédent, utilisent les couches d’image mises en cache. Toutefois, étant donné que les modifications `RUN` instruction n’est pas mis en cache, une nouvelle couche est créée pour l’instruction ont été modifiée et toutes les instructions suivantes.

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Lorsque vous comparez les ID de l’image de la nouvelle image à celui dans le premier exemple de cette section, vous remarquerez que les trois premières couches de bas en haut sont partagées, mais la quatrième et la cinquième sont uniques.

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

Les instructions Dockerfile ne respectent pas la casse, mais la convention consiste à utiliser des majuscules. Cela améliore la lisibilité en distinguant l’appel et le fonctionnement de l’instruction. Les deux exemples suivants comparent un fichier Dockerfile en minuscule et en majuscules.

Voici un fichier Dockerfile en minuscule:

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

Voici la même fichier Dockerfile à l’aide de majuscule:

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Retour à la ligne

Opérations longues et complexes peuvent être réparties sur plusieurs lignes par la barre oblique inverse `\` caractère. Le fichier Dockerfile suivant installe le package redistribuable de Visual Studio, supprime les fichiers du programme d’installation, puis crée un fichier de configuration. Ces trois opérations sont toutes spécifiées sur une seule ligne.

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

La commande peut être divisée avec des barres obliques inverses donc que chaque opération de le `RUN` instruction soit spécifiée sur sa propre ligne.

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Les références et obtenir des informations supplémentaires

[Fichier Dockerfile sur Windows](manage-windows-dockerfile.md)

[Best practices for writing Dockerfiles sur Docker.com](https://docs.docker.com/engine/reference/builder/)

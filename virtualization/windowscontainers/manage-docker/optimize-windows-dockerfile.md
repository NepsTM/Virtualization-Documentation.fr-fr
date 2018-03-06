---
title: Optimiser les fichiers Dockerfile Windows
description: Optimisez des fichiers Dockerfile pour les conteneurs Windows.
keywords: docker, conteneurs
author: PatrickLang
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
ms.openlocfilehash: ba45ee94a5d5e8a15bbbeab7b9c25f15be8be731
ms.sourcegitcommit: f542e8c95b5bb31b05b7c88f598f00f76779b519
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/01/2018
---
# <a name="optimize-windows-dockerfiles"></a>Optimiser les fichiers Dockerfile Windows

Plusieurs méthodes permettent d’optimiser le processus de génération Docker et les images Docker qui en résultent. Ce document décrit en détail le fonctionnement du processus de génération Docker et présente plusieurs tactiques pouvant être utilisées pour une création d’image optimale avec des conteneurs Windows.

## <a name="docker-build"></a>Génération Docker

### <a name="image-layers"></a>Couches d’image

Avant d’examiner l’optimisation de la génération Docker, il est important de comprendre comment la génération Docker fonctionne. Pendant le processus de génération Docker, un fichier Dockerfile est utilisé, et chaque instruction nécessitant une action est exécutée, l’une après l’autre, dans son propre conteneur temporaire. Le résultat est une nouvelle couche d’image pour chaque instruction nécessitant une action. 

Examinez le fichier Dockerfile suivant. Dans cet exemple, l’image de système d’exploitation de base `windowsservercore` est utilisée, IIS est installé, puis un site web simple est créé.

```
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

À partir de ce fichier Dockerfile, on peut s’attendre à ce que l’image obtenue soit constituée de deux couches, une pour l’image de système d’exploitation de conteneur et une autre qui inclut IIS et le site web. Ce n’est cependant pas le cas. La nouvelle image est composée de nombreuses couches, chacune dépendant de la précédente. Pour visualiser cela, la commande `docker history` peut être exécutée sur la nouvelle image. Cette opération montre que l’image se compose de quatre couches: la base, puis trois couches supplémentaires, une pour chaque instruction du fichier Dockerfile.

```
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

Chacune de ces couches peut être mappée à une instruction du fichier Dockerfile. La couche inférieure (`6801d964fda5` dans cet exemple) représente l’image du système d’exploitation de base. La couche au-dessus représente l’installation d’IIS. La couche suivante inclut le nouveau site web, etc.

Des fichiers Dockerfile peuvent être écrits pour réduire les couches d’image, optimiser les performances de génération, ainsi qu’optimiser des aspects esthétiques comme une meilleure lisibilité. Enfin, il existe de nombreuses façons d’effectuer la même tâche de génération d’image. Comprendre comment le format d’un fichier Dockerfile affecte le moment de la génération et l’image obtenue améliore l’automatisation. 

## <a name="optimize-image-size"></a>Optimiser la taille des images

Quand vous créez des images de conteneur Docker, la taille des images peut être un facteur important. Les images de conteneur sont déplacées entre les registres et l’hôte, sont exportées et importées, et finalement consomment de l’espace. Plusieurs tactiques permettent de réduire la taille des images pendant le processus de génération Docker. Cette section présente en détail certaines de ces tactiques spécifiques aux conteneurs Windows. 

Pour plus d’informations sur les bonnes pratiques Dockerfile, voir [Best practices for writing Dockerfiles sur Docker.com]( https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Regrouper des actions connexes

Étant donné que chaque instruction `RUN` crée une couche dans l’image de conteneur, le fait de regrouper des actions en une seule instruction `RUN` peut réduire le nombre de couches. La réduction des couches peut ne pas avoir beaucoup d’impact sur la taille des images, contrairement au regroupement d’actions connexes. Cela sera présenté dans des exemples ultérieurs.

Les deux exemples suivants montrent la même opération, qui génère des images de conteneur de fonctionnalité identique. Toutefois, les deux fichiers Dockerfile sont construits différemment. Les images obtenues sont également comparées.  

Ce premier exemple télécharge et installe Python pour Windows, puis nettoie en supprimant le fichier d’installation téléchargé. Chacune de ces actions est exécutée dans sa propre instruction `RUN`.

```
FROM windowsservercore

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

L’image obtenue se compose de trois couches supplémentaires, une pour chaque instruction `RUN`.

```
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

Pour comparer, voici la même opération. Toutefois, toutes les étapes s’exécutent avec la même instruction `RUN`. Notez que chaque étape de l’instruction `RUN` se trouve sur une nouvelle ligne du fichier Dockerfile. Le caractère ’\\’ est utilisé pour créer un retour automatique à la ligne. 

```
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

L’image obtenue ici se compose d’une couche supplémentaire pour l’instruction `RUN`.

```
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB                
```

### <a name="remove-excess-files"></a>Supprimer l’excédent de fichiers

Si un fichier, tel qu’un programme d’installation, n’est plus nécessaire après avoir été utilisé, supprimez-le afin de réduire la taille de l’image. Cette opération doit se produire dans la même étape que celle de la copie du fichier dans la couche de l’image. Cela évite que le fichier persiste dans une couche d’image de niveau inférieur.

Dans cet exemple, le package Python est téléchargé et exécuté, puis l’exécutable est supprimé. Tout cela est effectué dans une seule opération `RUN` et génère une seule couche d’image.

```
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>Optimiser la vitesse de génération

### <a name="multiple-lines"></a>Plusieurs lignes

Lors de l’optimisation de la vitesse de génération Docker, il peut s’avérer judicieux de séparer des opérations en plusieurs instructions individuelles. Avoir plusieurs opérations `RUN` augmente l’efficacité de la mise en cache. Étant donné que des couches individuelles sont créées pour chaque instruction `RUN`, si une étape identique a déjà été exécutée dans une autre opération de génération Docker, cette opération mise en cache (couche d’image) est réutilisée. Le résultat est que l’exécution de la génération Docker est réduite.

Dans l’exemple suivant, Apache et les packages redistribuables Visual Studio sont téléchargés et installés, puis les fichiers inutiles sont nettoyés. Tout cela est effectué avec une seule instruction `RUN`. Si l’une de ces actions est mise à jour, toutes les actions sont réexécutées.

```
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

L’image obtenue se compose de deux couches: une pour l’image du système d’exploitation de base et l’autre qui contient toutes les opérations de l’instruction `RUN` unique.

```
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

Pour créer un contraste, voici les mêmes actions décomposées en trois instructions `RUN`. Dans ce cas, chaque instruction `RUN` est mise en cache dans une couche d’image de conteneur, et seules celles qui ont été modifiées doivent être réexécutées sur les builds Dockerfile suivantes.

```
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

L’image obtenue se compose de quatre couches: une pour l’image du système d’exploitation de base, puis une pour chaque instruction `RUN`. Étant donné que chaque instruction `RUN` a été exécutée dans sa propre couche, les exécutions suivantes de ce fichier Dockerfile ou d’un ensemble identique d’instructions d’un autre fichier Dockerfile utilisent la couche d’image mise en cache, ce qui réduit le temps de génération. Le classement des instructions est important quand vous utilisez un cache d’image. Pour plus d’informations, voir la section suivante de ce document.

```
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

### <a name="ordering-instructions"></a>Classement des instructions

Un fichier Dockerfile est traité de haut en bas, chaque Instruction étant comparée aux couches mises en cache. Quand aucune couche mise en cache n’est trouvée pour une instruction, cette dernière et toutes les instructions suivantes sont traitées dans de nouvelles couches d’image de conteneur. C’est pourquoi l’ordre dans lequel les instructions sont placées est important. Placez les instructions qui resteront constantes en haut du fichier Dockerfile. Placez les instructions qui peuvent changer en bas du fichier Dockerfile. Cela réduit la probabilité d’annuler un cache existant.

L’objectif de cet exemple est de démontrer comment le classement des instructions Dockerfile peut affecter l’efficacité de la mise en cache. Dans ce fichier Dockerfile simple, quatre dossiers numérotés sont créés.  

```
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```
L’image obtenue comprend cinq couches: une pour l’image du système d’exploitation de base, et une pour chacune des instructions `RUN`.

```
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B    
```

Le fichier Dockerfile a été légèrement modifié. Notez que la troisième instruction `RUN` a changé. Quand la génération Docker est exécutée sur ce fichier Dockerfile, les trois premières instructions, qui sont identiques à celles de l’exemple précédent, utilisent les couches d’image mises en cache. Toutefois, étant donné que l’instruction `RUN` modifiée n’a pas été mise en cache, une nouvelle couche est créée pour elle-même et pour toutes les instructions suivantes.

```
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

En comparant les ID d’image de la nouvelle image à celui du dernier exemple, vous pouvez voir que les trois premières couches (de bas en haut) sont partagées, mais que la quatrième et la cinquième sont uniques.

```
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

Les instructions Dockerfile ne respectent pas la casse. Toutefois, la convention est d’utiliser des majuscules. Cela améliore la lisibilité en distinguant l’appel d’instructions et l’opération d’instructions. Les deux exemples ci-dessous illustrent ce concept. 

Minuscules:
```
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```
Majuscules: 
```
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Retour automatique à la ligne

Les opérations longues et complexes peuvent être réparties sur plusieurs lignes à l’aide du caractère barre oblique inverse(`\`). Le fichier Dockerfile suivant installe le package redistribuable de Visual Studio, supprime les fichiers du programme d’installation, puis crée un fichier de configuration. Ces trois opérations sont toutes spécifiées sur une seule ligne.

```
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```
La commande peut être réécrite afin que chaque opération de l’unique instruction `RUN` soit spécifiée sur sa propre ligne. 

```
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading--references"></a>Informations et références supplémentaires

[Fichier Dockerfile sur Windows] (manage-windows-dockerfile.md)

[Best practices for writing Dockerfiles sur Docker.com](https://docs.docker.com/engine/reference/builder/)

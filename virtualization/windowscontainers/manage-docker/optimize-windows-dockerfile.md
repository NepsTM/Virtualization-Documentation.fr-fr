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
ms.openlocfilehash: 871884c04b4165da4a5ab8af65bcda252672efbc
ms.sourcegitcommit: bea2c90f31a38fc7fda356619f0dd812f79d008f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/31/2019
ms.locfileid: "9685276"
---
# <a name="optimize-windows-dockerfiles"></a>Optimiser les fichiers Dockerfile Windows

Il existe de nombreuses façons d’optimiser le processus de génération de l’ancrage, ainsi que les images d’ancrage obtenues. Cet article décrit le fonctionnement du processus de génération de l’amarrage et la création optimale d’images pour les conteneurs Windows.

## <a name="image-layers-in-docker-build"></a>Couches d’image dans la build de l’amarrage

Pour pouvoir optimiser votre build d’arrimeur, vous devez savoir comment fonctionne la build de l’amarrage. Pendant le processus de génération Docker, un fichier Dockerfile est utilisé, et chaque instruction nécessitant une action est exécutée, l’une après l’autre, dans son propre conteneur temporaire. Le résultat est une nouvelle couche d’image pour chaque instruction nécessitant une action.

Par exemple, l’exemple suivant Dockerfile utilise l' `windowsservercore` image du système d’exploitation de base, installe les services Internet (IIS), puis crée un site Web simple.

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Il est possible que ce Dockerfile produise une image avec deux couches, une pour l’image du système d’exploitation de conteneur et une seconde incluant les services Internet et le site Web. Toutefois, l’image réelle comporte de nombreuses couches et chaque couche dépend de celle qui la précède.

Pour rendre ce plus clair, nous allons exécuter la `docker history` commande par rapport à l’image de notre exemple Dockerfile.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

La sortie montre que cette image comporte quatre couches: la couche de base et trois couches supplémentaires mappées à chaque instruction dans le Dockerfile. La couche inférieure (`6801d964fda5` dans cet exemple) représente l’image du système d’exploitation de base. L’une des couches consiste à installer IIS. La couche suivante inclut le nouveau site web, etc.

Dockerfiles peut être écrit pour réduire les couches d’image, optimiser les performances de génération et optimiser l’accessibilité par le biais de la lisibilité. Enfin, il existe de nombreuses façons d’effectuer la même tâche de génération d’image. Le fait de comprendre la manière dont le format de Dockerfile affecte le temps de génération et l’image qu’il crée améliore l’utilisation de l’automatisation.

## <a name="optimize-image-size"></a>Optimiser la taille de l’image

En fonction de votre espace requis, la taille de l’image peut être un facteur important lors de la création d’images de conteneur de dock. Les images de conteneur sont déplacées entre les registres et l’hôte, sont exportées et importées, et finalement consomment de l’espace. Cette section vous explique comment réduire la taille d’image lors du processus de génération de l’ancrage des conteneurs Windows.

Pour plus d’informations sur les pratiques recommandées pour Dockerfile, voir [meilleures pratiques pour écrire Dockerfiles dans docker.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Regrouper des actions connexes

Dans la `RUN` mesure où chaque instruction crée un calque dans l’image du conteneur, le regroupement `RUN` des actions en une instruction peut réduire le nombre de couches dans un Dockerfile. La réduction des couches peut ne pas avoir beaucoup d’impact sur la taille des images, contrairement au regroupement d’actions connexes. Cela sera présenté dans des exemples ultérieurs.

Dans cette section, nous allons comparer deux exemples de Dockerfiles qui effectuent les mêmes opérations. Toutefois, un Dockerfile possède une instruction par action, tandis que l’autre a regroupé ses actions associées.

L’exemple non groupé suivant Dockerfile télécharge Python pour Windows, l’installe et supprime le fichier d’installation téléchargé une fois l’installation terminée. Dans ce Dockerfile, chaque action dispose de ses propres `RUN` instructions.

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

Le second exemple est un Dockerfile qui effectue exactement la même opération. Toutefois, toutes les actions associées ont été regroupées dans `RUN` le cadre d’une instruction unique. Chaque étape de l' `RUN` instruction se trouve sur une nouvelle ligne du Dockerfile, tandis que le caractère «\» est utilisé pour le renvoi à la ligne.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

L’image résultante ne comporte qu’une couche supplémentaire `RUN` pour l’instruction.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Supprimer l’excédent de fichiers

S’il existe un fichier dans votre Dockerfile, tel qu’un programme d’installation, dont vous n’avez pas besoin, vous pouvez le supprimer pour réduire la taille de l’image. Cette opération doit se produire dans la même étape que celle de la copie du fichier dans la couche de l’image. Ainsi, le fichier n’est pas conservé dans une couche d’image de niveau inférieur.

Dans l’exemple suivant, Dockerfile, le package Python est téléchargé, exécuté, puis supprimé. Tout cela est effectué dans une seule opération `RUN` et génère une seule couche d’image.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>Optimiser la vitesse de création

### <a name="multiple-lines"></a>Lignes multiples

Vous pouvez fractionner les opérations en plusieurs instructions individuelles pour optimiser la vitesse de génération de l’amarrage. L' `RUN` efficacité de la mise en cache d’opérations multiples augmente en `RUN` raison du fait que des couches individuelles soient créées pour chaque instruction. Si une instruction identique a déjà été exécutée dans une autre opération de génération de l’ancrage, cette opération mise en cache (couche d’image) est réutilisée, ce qui engendre une diminution du runtime de build de l’ancrage.

Dans l’exemple suivant, Apache et Visual Studio redistribuent les packages sont téléchargés, installés, puis nettoyés en supprimant les fichiers qui ne sont plus nécessaires. Pour ce faire, vous disposez d' `RUN` une instruction unique. Si l’une de ces actions est mise à jour, toutes les actions se réexécutent.

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

L’image obtenue possède deux couches, une pour l’image du système d’exploitation de base et une pour toutes les opérations `RUN` de l’instruction unique.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

Dans le cas d’une comparaison, les mêmes actions sont `RUN` divisées en trois instructions. Dans ce cas, chaque `RUN` instruction est mise en cache dans une couche d’image de conteneur et seules celles qui ont été modifiées doivent être réexécutées sur les builds Dockerfile suivantes.

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

L’image obtenue se compose de quatre couches; un calque pour l’image du système d’exploitation de base et `RUN` chacune des trois instructions. Dans la `RUN` mesure où chaque instruction s’est exécutée dans sa propre couche, toute exécution subséquente de cette Dockerfile ou d’un ensemble d’instructions similaire dans un autre Dockerfile utilisera les couches d’image mises en cache, ce qui réduit le temps de création.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

Le mode de commande des instructions est important lorsque vous travaillez avec des caches d’image, comme vous le voyez dans la section suivante.

### <a name="ordering-instructions"></a>Instructions de classement

Un fichier Dockerfile est traité de haut en bas, chaque Instruction étant comparée aux couches mises en cache. Quand aucune couche mise en cache n’est trouvée pour une instruction, cette dernière et toutes les instructions suivantes sont traitées dans de nouvelles couches d’image de conteneur. C’est pourquoi l’ordre dans lequel les instructions sont placées est important. Placez les instructions qui resteront constantes en haut du fichier Dockerfile. Placez les instructions qui peuvent changer en bas du fichier Dockerfile. Cela réduit la probabilité d’annuler un cache existant.

Les exemples suivants montrent comment le classement d’instructions Dockerfile peut affecter l’efficacité de la mise en cache. Cet exemple simple Dockerfile comporte quatre dossiers numérotés.  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

L’image obtenue comporte cinq couches, une pour l’image du système d’exploitation de base `RUN` et chacune des instructions.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

L’Dockerfile suivant est désormais légèrement modifié, avec la troisième `RUN` instruction changée en nouveau fichier. Quand la génération Docker est exécutée sur ce fichier Dockerfile, les trois premières instructions, qui sont identiques à celles de l’exemple précédent, utilisent les couches d’image mises en cache. Toutefois, étant donné que `RUN` l’instruction modifiée n’est pas mise en cache, une nouvelle couche est créée pour l’instruction modifiée et toutes les instructions ultérieures.

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Lorsque vous comparez les ID d’image de la nouvelle image à celle du premier exemple de cette section, vous remarquerez que les trois premières couches du bas vers le haut sont partagées, mais les quatre et cinquième sont uniques.

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

### <a name="instruction-case"></a>Cas d’instructions

Dockerfile instructions ne respectent pas la casse, mais la Convention consiste à utiliser majuscule. Cela améliore la lisibilité en différentiant entre l’appel d’instruction et l’opération d’instructions. Les deux exemples suivants comparent une Dockerfile en majuscule et en majuscule.

Voici un Dockerfile non majuscule:

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

Voici les mêmes Dockerfile en majuscule:

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Habillage du trait

Les opérations longues et complexes peuvent être divisées en plusieurs lignes `\` par le caractère barre oblique inverse. Le fichier Dockerfile suivant installe le package redistribuable de Visual Studio, supprime les fichiers du programme d’installation, puis crée un fichier de configuration. Ces trois opérations sont toutes spécifiées sur une seule ligne.

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

La commande peut être scindée par des barres obliques inverses, de telle `RUN` sorte que chaque opération de la même instruction soit spécifiée sur la même ligne.

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Autres lectures et références

[Fichier Dockerfile sur Windows](manage-windows-dockerfile.md)

[Best practices for writing Dockerfiles sur Docker.com](https://docs.docker.com/engine/reference/builder/)

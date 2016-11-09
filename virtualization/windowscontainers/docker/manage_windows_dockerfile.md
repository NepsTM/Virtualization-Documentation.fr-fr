---
title: Fichier Dockerfile et conteneurs Windows
description: "Créez des fichiers Dockerfile pour les conteneurs Windows."
keywords: docker, conteneurs
author: PatrickLang
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
translationtype: Human Translation
ms.sourcegitcommit: 31515396358c124212b53540af8a0dcdad3580e4
ms.openlocfilehash: 20dcc6d263488673bf0a025058c3dee8d30168a2

---

# Fichier Dockerfile sur Windows

Le moteur Docker inclut des outils pour l’automatisation de la création d’images de conteneur. Alors que les images de conteneur peuvent être créées manuellement à l’aide de la commande `docker commit`, l’adoption d’un processus de création d’image automatique offre de nombreux avantages, parmi lesquels :

- stockage d’images de conteneur sous forme de code ;
- nouvelle création rapide et précise d’images de conteneur à des fins de maintenance et de mise à niveau ;
- intégration continue entre les images de conteneur et le cycle de développement.

Les composants Docker qui dirigent cette automatisation sont le fichier Dockerfile et la commande `docker build`.

- **Fichier Dockerfile** : fichier texte contenant les instructions nécessaires pour créer une image de conteneur. Ces instructions incluent l’identification d’une image existante à utiliser comme base, les commandes à exécuter pendant le processus de création d’image et une commande qui s’exécute quand de nouvelles instances de l’image de conteneur sont déployées.
- **docker build** : commande du moteur Docker qui utilise un fichier Dockerfile, puis déclenche le processus de création d’image.

Ce document présente l’utilisation d’un fichier Dockerfile avec les conteneurs Windows, aborde la syntaxe et décrit en détail les instructions pour les fichiers Dockerfile couramment utilisées. 

Tout au long de ce document, nous aborderons le concept d’images de conteneur et de couches d’images de conteneur. Pour plus d’informations sur les images et la superposition d’images, voir [Gérer les images de conteneur Windows](../management/manage_images.md). 

Pour une étude complète des fichiers Dockerfile, voir les [informations de référence sur les fichiers Dockerfile sur docker.com]( https://docs.docker.com/engine/reference/builder/).

## Présentation d’un fichier Dockerfile

### Syntaxe de base

Sous sa forme la plus basique, un fichier Dockerfile peut être très simple. L’exemple suivant crée une image, qui comprend IIS et un site « hello world ». Cet exemple inclut des commentaires (signalés par un `#`), qui décrivent chaque étape. Les sections suivantes de cet article approfondissent les règles de syntaxe et les instructions pour les fichiers Dockerfile.

```none
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM microsoft/windowsservercore

# Metadata indicating an image maintainer.
MAINTAINER jshelton@contoso.com

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Pour obtenir des exemples supplémentaires de fichiers Dockerfile pour Windows, voir le [référentiel des fichiers Dockerfile pour Windows] (https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## Instructions

Les instructions pour les fichiers Dockerfile fournissent au moteur Docker les étapes nécessaires pour créer une image de conteneur. Ces instructions sont exécutées dans l’ordre et une par une. Voici les détails pour certaines instructions pour les fichiers Dockerfile de base. Pour obtenir une liste complète des instructions pour les fichiers Dockerfile, voir les [informations de référence sur les fichiers Dockerfile sur docker.com] (https://docs.docker.com/engine/reference/builder/).

### FROM

L’instruction `FROM` définit l’image de conteneur qui sera utilisée pendant le processus de création de l’image. Par exemple, quand vous utilisez l’instruction `FROM windowsservercore`, l’image obtenue est dérivée de l’image du système d’exploitation de base Windows Server Core et a une dépendance sur celle-ci. Si l’image spécifiée n’est pas présente sur le système où le processus de génération Docker est en cours d’exécution, le moteur Docker tente de télécharger l’image à partir d’un Registre d’images public ou privé.

**Format**

L’instruction FROM se présente sous le format suivant : 

```
FROM <image>
```

**Exemple**

```
FROM windowsservercore
```

Pour plus d’informations sur l’instruction FROM, voir les [informations de référence sur FROM sur Docker.com]( https://docs.docker.com/engine/reference/builder/#from). 

### RUN

L’instruction `RUN` spécifie les commandes à exécuter et capturer dans la nouvelle image de conteneur. Ces commandes peuvent concerner notamment l’installation de logiciels ou encore la création de fichiers et de répertoires ainsi que de la configuration de l’environnement.

**Format**

L’instruction RUN se présente sous le format suivant : 

```none
# exec form

RUN ["<executable", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

La différence entre la forme exec et la forme shell réside dans la façon dont l’instruction `RUN` est exécutée. Quand vous utilisez la forme exec, le programme spécifié est exécuté explicitement. 

L’exemple suivant a utilisé la forme exec.

```none
FROM windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

En examinant l’image obtenue, la commande exécutée est `powershell New-Item c:/test`.

```none
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

En revanche, l’exemple suivant exécute la même opération, mais en utilisant la forme shell.

```none
FROM windowsservercore

RUN powershell New-Item c:\test
```

Le résultat est l’instruction d’exécution `cmd /S /C powershell New-Item c:\test`. 

```none
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

**Considérations relatives à Windows**
 
Sur Windows, quand vous utilisez l’instruction `RUN` avec le format exec, les barres obliques inverses doivent être placées dans une séquence d’échappement.

```none
RUN ["powershell", "New-Item", "c:\\test"]
```

Quand le programme cible est un programme Windows Installer, une étape supplémentaire est nécessaire avant de lancer la procédure d’installation réelle (sans assistance) : l’extraction du programme d’installation, via l’indicateur `/x:<directory>`. Vous devez attendre la fin de la commande pour quitter, sinon le processus se termine prématurément sans avoir installé quoi que ce soit. Pour plus d’informations, consultez l’exemple ci-dessous.

**Exemples**

Cet exemple utilise DISM pour installer IIS dans l’image de conteneur.
```none
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

Cet exemple installe le package redistribuable Visual Studio. Notez ici que `Start-Process` et le paramètre `-Wait` sont utilisés pour exécuter le programme d’installation. Cela permet de garantir que l’installation est terminée avant de passer à l’étape suivante dans le fichier Dockerfile.

```none
RUN Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
``` 

Pour plus d’informations sur l’instruction RUN, voir les [informations de référence sur RUN sur Docker.com]( https://docs.docker.com/engine/reference/builder/#run). 

### COPY

L’instruction `COPY` copie les fichiers et les répertoires sur le système de fichiers du conteneur. Les fichiers et répertoires doivent se trouver dans un chemin relatif au fichier Dockerfile.

**Format**

L’instruction `COPY` se présente sous le format suivant : 

```none
COPY <source> <destination>
``` 

Si la source ou la destination contient des espaces, placez le chemin entre crochets et guillemets doubles.
 
```none
COPY ["<source>", "<destination>"]
```

**Considérations relatives à Windows**
 
Sur Windows, le format de destination doit utiliser des barres obliques. Voici, par exemple, des instructions `COPY` valides.

```none
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

Toutefois, l’instruction suivante ne fonctionne pas.

```none
COPY test1.txt c:\temp\
```

**Exemples**

Cet exemple ajoute le contenu du répertoire source à un répertoire nommé `sqllite` dans l’image de conteneur.
```none
COPY source /sqlite/
```

Cet exemple ajoute tous les fichiers qui commencent par config au répertoire `c:\temp` de l’image de conteneur.
```none
COPY config* c:/temp/
```

Pour plus d’informations sur l’instruction `COPY`, voir les [informations de référence sur COPY sur Docker.com]( https://docs.docker.com/engine/reference/builder/#copy).

### ADD

L’instruction ADD est très similaire à l’instruction COPY, mais elle inclut des fonctionnalités supplémentaires. En plus de copier des fichiers à partir de l’hôte dans l’image de conteneur, l’instruction `ADD` peut également copier des fichiers depuis un emplacement distant avec une spécification d’URL.

**Format**

L’instruction `ADD` se présente sous le format suivant : 

```none
ADD <source> <destination>
``` 

Si la source ou la destination contient des espaces, placez le chemin entre crochets et guillemets doubles.
 
```none
ADD ["<source>", "<destination>"]
```

**Considérations relatives à Windows**
 
Sur Windows, le format de destination doit utiliser des barres obliques. Voici, par exemple, des instructions `ADD` valides.

```none
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

Toutefois, l’instruction suivante ne fonctionne pas.

```none
ADD test1.txt c:\temp\
```

En outre, sur Linux, l’instruction `ADD` développera des packages compressés dans le cadre d’une copie. Cette fonctionnalité n’est pas disponible dans Windows.

**Exemples**

Cet exemple ajoute le contenu du répertoire source à un répertoire nommé `sqllite` dans l’image de conteneur.
```none
ADD source /sqlite/
```

Cet exemple ajoute tous les fichiers qui commencent par config au répertoire `c:\temp` de l’image de conteneur.
```none
ADD config* c:/temp/
```

Cet exemple télécharge Python pour Windows dans le répertoire `c:\temp` de l’image de conteneur.
```none
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Pour plus d’informations sur l’instruction `ADD`, voir les [informations de référence sur ADD sur Docker.com]( https://docs.docker.com/engine/reference/builder/#add). 

### WORKDIR

L’instruction `WORKDIR` définit un répertoire de travail pour d’autres instructions pour les fichiers Dockerfile, telles que `RUN` et `CMD`, ainsi que le répertoire de travail pour les instances en cours d’exécution de l’image de conteneur.

**Format**

L’instruction `WORKDIR` se présente sous le format suivant : 

```none
WORKDIR <path to working directory>
``` 

**Considérations relatives à Windows**

Sur Windows, si le répertoire de travail comprend une barre oblique inverse, celle-ci doit être placée dans une séquence d’échappement.

```none
WORKDIR c:\\windows
```

**Exemples**

```none
WORKDIR c:\\Apache24\\bin
```

Pour plus d’informations sur l’instruction `WORKDIR`, voir les [informations de référence sur WORKDIR sur Docker.com]( https://docs.docker.com/engine/reference/builder/#workdir). 

### CMD

L’instruction `CMD` définit la commande par défaut à exécuter lors du déploiement d’une instance de l’image de conteneur. Par exemple, si le conteneur doit héberger un serveur web NGINX, l’instruction `CMD` peut inclure des instructions pour démarrer le serveur web, par exemple `nginx.exe`. Si plusieurs instructions `CMD` sont spécifiées dans un fichier Dockerfile, seule la dernière est évaluée.

**Format**

L’instruction `CMD` se présente sous le format suivant : 

```none
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

**Considérations relatives à Windows**

Sur Windows, les chemins de fichiers spécifiés dans l’instruction `CMD` doivent utiliser des barres obliques. En outre, les barres obliques inverses `\\` doivent être placées dans une séquence d’échappement. Voici, par exemple, des instructions `CMD` valides.

```none
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```
Toutefois, l’instruction suivante ne fonctionne pas.

```none
CMD c:\Apache24\bin\httpd.exe -w
```

Pour plus d’informations sur l’instruction `CMD`, voir les [informations de référence sur CMD sur Docker.com]( https://docs.docker.com/engine/reference/builder/#cmd). 

## Caractère d’échappement

Dans de nombreux cas, une instruction de fichier Dockerfile doit s’étendre sur plusieurs lignes ; pour cela, utilisez un caractère d’échappement. Le caractère d’échappement de fichier Dockerfile par défaut est une barre oblique inverse `\`. Étant donné que la barre oblique inverse est également un séparateur de chemin de fichier dans Windows, cela peut être problématique. Pour modifier le caractère d’échappement par défaut, une directive d’analyseur peut être utilisée. Pour plus d’informations sur les directives d’analyseur, voir [Directives d’analyseur sur Docker.com]( https://docs.docker.com/engine/reference/builder/#parser-directives).

L’exemple suivant montre une instruction RUN unique qui s’étend sur plusieurs lignes à l’aide du caractère d’échappement par défaut.

```none
FROM windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Pour modifier le caractère d’échappement, placez une directive d’analyseur d’échappement sur la toute première ligne du fichier Dockerfile. Une illustration figure dans l’exemple ci-dessous.

> Notez que deux valeurs seulement peuvent être utilisées en tant que caractères d’échappement, `\` et `` ` ``.

```none
# escape=`

FROM windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Pour plus d’informations sur la directive d’analyseur d’échappement, voir [Directive d’analyseur d’échappement sur Docker.com]( https://docs.docker.com/engine/reference/builder/#escape).

## PowerShell dans un fichier Dockerfile

### Commandes PowerShell

Les commandes PowerShell peuvent être exécutées dans un fichier Dockerfile avec l’opération `RUN`. 

```none
FROM windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### Appels REST

PowerShell et la commande `Invoke-WebRequest` peuvent s’avérer utiles lors de la collecte des informations ou des fichiers à partir d’un service web. Par exemple, si vous créez une image qui inclut Python, l’exemple suivant peut servir.

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> La commande Invoke-WebRequest n’est pas prise en charge dans Nano Server

Pour télécharger les fichiers avec PowerShell pendant le processus de création d’image, une autre option consiste à employer la bibliothèque WebClient .NET. Cela peut améliorer les performances en matière de téléchargement. L’exemple suivant télécharge le logiciel Python à l’aide de la bibliothèque WebClient.

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> La bibliothèque WebClient n’est pas prise en charge dans Nano Server

### Scripts PowerShell

Dans certains cas, il peut être utile de copier un script dans le conteneur utilisé au cours du processus de création d’image, puis de l’exécuter à partir du conteneur. Remarque : Vous limitez ainsi la mise en cache des couches d’images et réduisez la lisibilité du fichier Dockerfile.

Cet exemple copie un script à partir de l’ordinateur de build dans le conteneur à l’aide de l’instruction `ADD`. Ce script est alors exécuté à l’aide de l’instruction RUN.

```
FROM windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## Docker Build 

Une fois qu’un fichier Dockerfile a été créé et enregistré sur disque, la commande `docker build` peut être exécutée pour créer l’image. La commande `docker build` accepte plusieurs paramètres facultatifs et un chemin d’accès au fichier Dockerfile. Pour obtenir une documentation complète sur Docker Build, notamment une liste de toutes les options de build, voir les [informations de référence sur build sur Docker.com](https://docs.docker.com/engine/reference/commandline/build/#build).

```none
Docker build [OPTIONS] PATH
```
Par exemple, la commande suivante crée une image nommée « iis ».

```none
docker build -t iis .
```

Quand le processus de génération a été lancé, la sortie indique l’état et retourne les erreurs levées.

```none
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM windowsservercore
 ---> 6801d964fda5

Step 2 : RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
 ---> Running in ae8759fb47db

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0

Enabling feature(s)
The operation completed successfully.

 ---> 4cd675d35444
Removing intermediate container ae8759fb47db

Step 3 : RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
 ---> Running in 9a26b8bcaa3a
 ---> e2aafdfbe392
Removing intermediate container 9a26b8bcaa3a

Successfully built e2aafdfbe392
```

Le résultat est une nouvelle image de conteneur, nommée « iis » dans cet exemple.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## Informations et références supplémentaires

[Optimiser les fichiers Dockerfile et docker build pour Windows] (./optimize_windows_dockerfile.md)

[Informations de référence sur les fichiers Dockerfile sur docker.com](https://docs.docker.com/engine/reference/builder/)



<!--HONumber=Nov16_HO1-->



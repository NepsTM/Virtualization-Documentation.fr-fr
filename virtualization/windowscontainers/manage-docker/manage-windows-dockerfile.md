---
title: Fichier Dockerfile et conteneurs Windows
description: Créez des fichiers Dockerfile pour les conteneurs Windows.
keywords: docker, conteneurs
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
ms.openlocfilehash: c08fa4d0a89bddeddd0f0a918345c33a6e2ab893
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/28/2019
ms.locfileid: "9680989"
---
# <a name="dockerfile-on-windows"></a>Fichier Dockerfile sur Windows

Le moteur de l’ancrage inclut des outils qui automatisent la création d’images de conteneurs. Même si vous pouvez créer des images de conteneurs manuellement `docker commit` en exécutant la commande, l’adoption d’un processus de création d’image automatisé présente de nombreux avantages, notamment:

- stockage d’images de conteneur sous forme de code;
- nouvelle création rapide et précise d’images de conteneur à des fins de maintenance et de mise à niveau;
- intégration continue entre les images de conteneur et le cycle de développement.

Les composants Docker qui dirigent cette automatisation sont le fichier Dockerfile et la commande `docker build`.

Le Dockerfile est un fichier texte qui contient les instructions nécessaires pour créer une nouvelle image de conteneur. Ces instructions incluent l’identification d’une image existante à utiliser comme base, les commandes à exécuter pendant le processus de création d’image et une commande qui s’exécute quand de nouvelles instances de l’image de conteneur sont déployées.

La version d’amarrage est la commande du moteur de l’ancrage qui utilise un Dockerfile et déclenche le processus de création d’image.

Cette rubrique vous montre comment utiliser Dockerfiles avec des conteneurs Windows, comprendre leur syntaxe de base et les instructions Dockerfile les plus courantes.

Ce document traite du concept d’images de conteneurs et de couches d’image de conteneur. Pour en savoir plus sur les images et les couches d’images, voir [le Guide de démarrage rapide pour les images](../quick-start/quick-start-images.md).

Pour obtenir une vue d’ensemble de Dockerfiles, voir la [référence Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="basic-syntax"></a>Syntaxe de base

Sous sa forme la plus basique, un fichier Dockerfile peut être très simple. L’exemple suivant crée une image, qui comprendIIS et un site «hello world». Cet exemple inclut des commentaires (signalés par un `#`), qui décrivent chaque étape. Les sections suivantes de cet article approfondissent les règles de syntaxe et les instructions pour les fichiers Dockerfile.

>[!NOTE]
>Une Dockerfile doit être créée sans extension. Pour cela, dans Windows, créez le fichier avec votre éditeur de votre choix, puis enregistrez-le avec la notation «Dockerfile» (avec les guillemets).

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM mcr.microsoft.com/windows/servercore:ltsc2019

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Pour obtenir des exemples supplémentaires de Dockerfiles pour Windows, voir le [référentiel Dockerfile pour Windows](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## <a name="instructions"></a>Instructions

Dockerfile instructions fournissent aux instructions du moteur de l’ancrage les instructions nécessaires pour créer une image de conteneur. Ces instructions sont effectuées une par une et dans l’ordre. Les exemples suivants sont les instructions les plus fréquemment utilisées dans Dockerfiles. Pour obtenir la liste complète des instructions Dockerfile, consultez la [référence Dockerfile](https://docs.docker.com/engine/reference/builder/).

### <a name="from"></a>FROM

L’instruction `FROM` définit l’image de conteneur qui sera utilisée pendant le processus de création de l’image. Par exemple, quand vous utilisez l’instruction `FROM microsoft/windowsservercore`, l’image obtenue est dérivée de l’image du système d’exploitation de base Windows Server Core et a une dépendance sur celle-ci. Si l’image spécifiée n’est pas présente sur le système où le processus de génération Docker est en cours d’exécution, le moteur Docker tente de télécharger l’image à partir d’un Registre d’images public ou privé.

Le format de l’instruction est semblable à ce qui suit:

```dockerfile
FROM <image>
```

Voici un exemple de commande FROM:

Pour télécharger la version ltsc2019 de Windows Server Core à partir du registre des conteneurs Microsoft:
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

Pour obtenir des informations plus détaillées, consultez la [référence de](https://docs.docker.com/engine/reference/builder/#from).

### <a name="run"></a>RUN

L’instruction `RUN` spécifie les commandes à exécuter et capturer dans la nouvelle image de conteneur. Ces commandes peuvent concerner notamment l’installation de logiciels ou encore la création de fichiers et de répertoires ainsi que de la configuration de l’environnement.

L’instruction RUN ressemble à ceci:

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

La différence entre les formulaires exec et Shell réside dans le mode `RUN` d’exécution de l’instruction. Quand vous utilisez la forme exec, le programme spécifié est exécuté explicitement.

Voici un exemple du formulaire exec:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

L’image résultante exécute `powershell New-Item c:/test` la commande suivante:

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

Par exemple, l’exemple suivant exécute la même opération sous forme d’interpréteur de commande:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

L’image obtenue a une instruction Run de `cmd /S /C powershell New-Item c:\test`.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Remarques relatives à l’utilisation de la fonction exécuter avec Windows

Sur Windows, quand vous utilisez l’instruction `RUN` avec le format exec, les barres obliques inverses doivent être placées dans une séquence d’échappement.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Lorsque le programme cible est un programme d’installation Windows, vous devez extraire le programme d’installation `/x:<directory>` par le biais de l’indicateur pour pouvoir lancer la procédure d’installation réelle (silencieuse). Vous devez également attendre que la commande s’arrête avant d’effectuer une autre action. Dans le cas contraire, le processus se termine prématurément sans avoir à installer du tout. Pour plus d’informations, consultez l’exemple ci-dessous.

#### <a name="examples-of-using-run-with-windows"></a>Exemples d’utilisation de la fonction exécuter avec Windows

L’exemple suivant Dockerfile utilise DISM pour installer IIS dans l’image du conteneur:

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

Cet exemple installe le package redistribuable Visual Studio. `Start-Process` le `-Wait` paramètre est utilisé pour exécuter le programme d’installation. Cela permet de s’assurer que l’installation est terminée avant de passer à l’instruction suivante dans le Dockerfile.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Pour plus d’informations sur l’instruction RUN, voir la [référence Run](https://docs.docker.com/engine/reference/builder/#run).

### <a name="copy"></a>COPY

L' `COPY` instruction copie les fichiers et répertoires dans le système de fichiers du conteneur. Les fichiers et répertoires doivent se trouver dans un chemin d’accès relatif au Dockerfile.

Le `COPY` format de l’instruction ressemble à ceci:

```dockerfile
COPY <source> <destination>
```

Si les éléments source ou de destination comportent des espaces blancs, entourez le chemin d’accès entre crochets et guillemets doubles, comme le montre l’exemple suivant:

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Remarques relatives à l’utilisation de la fonction copier avec Windows

Sur Windows, le format de destination doit utiliser des barres obliques. Par exemple, Voici des instructions `COPY` valides:

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

Entre-temps, le format suivant ne fonctionne pas avec les barres obliques inverses:

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Exemples d’utilisation de la fonction copier avec Windows

L’exemple suivant ajoute le contenu du répertoire source à un répertoire nommé `sqllite` dans l’image du conteneur:

```dockerfile
COPY source /sqlite/
```

L’exemple suivant ajoute tous les fichiers qui commencent par config dans le `c:\temp` répertoire de l’image du conteneur:

```dockerfile
COPY config* c:/temp/
```

Pour plus d’informations sur l' `COPY` instruction, voir la [référence de copie](https://docs.docker.com/engine/reference/builder/#copy).

### <a name="add"></a>ADD

L’instruction ADD est semblable à l’instruction de copie, mais elle est dotée d’encore plus de fonctionnalités. En plus de copier des fichiers à partir de l’hôte dans l’image de conteneur, l’instruction `ADD` peut également copier des fichiers depuis un emplacement distant avec une spécification d’URL.

Le `ADD` format de l’instruction ressemble à ceci:

```dockerfile
ADD <source> <destination>
```

Si la source ou la destination incluent un espace blanc, placez le chemin entre crochets et guillemets doubles:

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Remarques relatives à l’exécution de la fonction Ajouter avec Windows

Sur Windows, le format de destination doit utiliser des barres obliques. Par exemple, Voici des instructions `ADD` valides:

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

Entre-temps, le format suivant ne fonctionne pas avec les barres obliques inverses:

```dockerfile
ADD test1.txt c:\temp\
```

En outre, sur Linux, l’instruction `ADD` développera des packages compressés dans le cadre d’une copie. Cette fonctionnalité n’est pas disponible dans Windows.

#### <a name="examples-of-using-add-with-windows"></a>Exemples d’utilisation de la fonction Ajouter avec Windows

L’exemple suivant ajoute le contenu du répertoire source à un répertoire nommé `sqllite` dans l’image du conteneur:

```dockerfile
ADD source /sqlite/
```

L’exemple suivant ajoute tous les fichiers qui commencent par «config» dans le `c:\temp` répertoire de l’image du conteneur.

```dockerfile
ADD config* c:/temp/
```

L’exemple suivant permet de Télécharger python pour Windows dans `c:\temp` le répertoire de l’image du conteneur.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Pour plus d’informations sur l' `ADD` instruction, voir la [référence ajouter](https://docs.docker.com/engine/reference/builder/#add).

### <a name="workdir"></a>WORKDIR

L’instruction `WORKDIR` définit un répertoire de travail pour d’autres instructions pour les fichiers Dockerfile, telles que `RUN` et `CMD`, ainsi que le répertoire de travail pour les instances en cours d’exécution de l’image de conteneur.

Le `WORKDIR` format de l’instruction ressemble à ceci:

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Remarques relatives à l’utilisation de WORKDIR avec Windows

Sur Windows, si le répertoire de travail comprend une barre oblique inverse, celle-ci doit être placée dans une séquence d’échappement.

```dockerfile
WORKDIR c:\\windows
```

**Exemples**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

Pour plus d’informations sur `WORKDIR` l’instruction, voir la [référence de WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir).

### <a name="cmd"></a>CMD

L’instruction `CMD` définit la commande par défaut à exécuter lors du déploiement d’une instance de l’image de conteneur. Par exemple, si le conteneur doit héberger un serveur Web NGINX, les `CMD` instructions peuvent être associées pour démarrer le serveur Web avec une commande `nginx.exe`telle que. Si plusieurs instructions `CMD` sont spécifiées dans un fichier Dockerfile, seule la dernière est évaluée.

Le `CMD` format de l’instruction ressemble à ceci:

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Remarques relatives à l’utilisation de CMD avec Windows

Sur Windows, les chemins de fichiers spécifiés dans l’instruction `CMD` doivent utiliser des barres obliques. En outre, les barres obliques inverses `\\` doivent être placées dans une séquence d’échappement. Les instructions suivantes sont `CMD` valides:

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

Toutefois, le format suivant sans les barres obliques inappropriées ne fonctionne pas:

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

Pour plus d’informations sur l' `CMD` instruction, voir la [référence de cmd](https://docs.docker.com/engine/reference/builder/#cmd).

## <a name="escape-character"></a>Caractère d’échappement

Dans de nombreux cas, une instruction Dockerfile doit s’étendre sur plusieurs lignes. Pour cela, vous pouvez utiliser un caractère d’échappement. Le caractère d’échappement de fichier Dockerfile par défaut est une barre oblique inverse `\`. Toutefois, étant donné que la barre oblique inverse est également un séparateur de chemin d’accès à un fichier dans Windows, son utilisation pour s’étendre sur plusieurs lignes peut poser des problèmes. Pour y accéder, vous pouvez utiliser une directive d’analyse pour modifier le caractère d’échappement par défaut. Pour plus d’informations sur les directives de l’analyseur, voir [directives](https://docs.docker.com/engine/reference/builder/#parser-directives)de l’analyseur.

L’exemple suivant illustre une instruction RUN unique qui s’étend sur plusieurs lignes à l’aide du caractère d’échappement par défaut:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Pour modifier le caractère d’échappement, placez une directive d’analyseur d’échappement sur la toute première ligne du fichier Dockerfile. Cela peut être illustré dans l’exemple suivant.

>[!NOTE]
>Seules deux valeurs peuvent être utilisées en tant que caractères `\` d' `` ` ``échappement: et.

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Pour plus d’informations sur la directive d’analyse d’échappement, voir [directive d’analyse d’échappement](https://docs.docker.com/engine/reference/builder/#escape).

## <a name="powershell-in-dockerfile"></a>PowerShell dans un fichier Dockerfile

### <a name="powershell-cmdlets"></a>Applets de commande PowerShell

Les `RUN` applets de commande PowerShell peuvent être exécutées dans une Dockerfile.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>Appels REST

L’applet `Invoke-WebRequest` de cmdlet PowerShell peut s’avérer utile lors de la collecte d’informations ou de fichiers à partir d’un service Web. Par exemple, si vous créez une image incluant Python, vous pouvez définir `$ProgressPreference` `SilentlyContinue` pour obtenir des téléchargements plus rapides, comme le montre l’exemple suivant.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` fonctionne également sur nano Server.

Pour télécharger les fichiers avec PowerShell pendant le processus de création d’image, une autre option consiste à employer la bibliothèque WebClient .NET. Cela peut améliorer les performances en matière de téléchargement. L’exemple suivant télécharge le logiciel Python à l’aide de la bibliothèque WebClient.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Le nano Server ne prend actuellement pas en charge WebClient.

### <a name="powershell-scripts"></a>Scripts PowerShell

Dans certains cas, il est possible que vous deviez copier un script dans les conteneurs que vous utilisez lors du processus de création d’image, puis exécuter le script à partir du conteneur.

>[!NOTE]
>Cela limite la mise en cache de la couche d’image et diminue la lisibilité de Dockerfile.

Cet exemple copie un script à partir de l’ordinateur de build dans le conteneur à l’aide de l’instruction `ADD`. Ce script est alors exécuté à l’aide de l’instruction RUN.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Build de l’amarrage

Une fois qu’un Dockerfile a été créé et enregistré sur le disque, `docker build` vous pouvez l’exécuter pour créer la nouvelle image. La commande `docker build` accepte plusieurs paramètres facultatifs et un chemin d’accès au fichier Dockerfile. Pour obtenir une documentation complète sur la build de l’ancrage, y compris la liste de toutes les options de génération, voir la [référence de génération](https://docs.docker.com/engine/reference/commandline/build/#build).

Le format de la `docker build` commande est semblable à ce qui suit:

```dockerfile
docker build [OPTIONS] PATH
```

Par exemple, la commande suivante crée une image nommée «IIS».

```dockerfile
docker build -t iis .
```

Une fois le processus de génération lancé, la sortie indique l’État et renvoie les erreurs levées.

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM mcr.microsoft.com/windows/servercore:ltsc2019
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

Le résultat est une nouvelle image de conteneur, qui dans cet exemple est nommée «IIS».

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>Autres lectures et références

- [Optimize Dockerfiles et version d’amarrage pour Windows](optimize-windows-dockerfile.md)
- [Référence Dockerfile](https://docs.docker.com/engine/reference/builder/)

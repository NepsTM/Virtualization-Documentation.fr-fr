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
ms.openlocfilehash: 9fef74c029dc3efc220b1f9924d2695cdbaa61be
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909659"
---
# <a name="dockerfile-on-windows"></a>Fichier Dockerfile sur Windows

Le moteur de l’ancrage comprend des outils qui automatisent la création d’images de conteneur. Bien que vous puissiez créer des images de conteneur manuellement en exécutant la commande `docker commit`, l’adoption d’un processus de création d’images automatisées présente de nombreux avantages, notamment :

- stockage d’images de conteneur sous forme de code ;
- nouvelle création rapide et précise d’images de conteneur à des fins de maintenance et de mise à niveau ;
- intégration continue entre les images de conteneur et le cycle de développement.

Les composants Docker qui dirigent cette automatisation sont le fichier Dockerfile et la commande `docker build`.

Le fichier dockerfile est un fichier texte qui contient les instructions nécessaires à la création d’une image de conteneur. Ces instructions incluent l’identification d’une image existante à utiliser comme base, les commandes à exécuter pendant le processus de création d’image et une commande qui s’exécute quand de nouvelles instances de l’image de conteneur sont déployées.

La build de l’ancreur est la commande du moteur de l’ancrage qui consomme un fichier dockerfile et déclenche le processus de création de l’image.

Cette rubrique vous montre comment utiliser fichiers dockerfile avec des conteneurs Windows, comprendre leur syntaxe de base et les instructions fichier dockerfile les plus courantes.

Ce document aborde le concept d’images conteneur et de couches d’images de conteneur. Si vous souhaitez en savoir plus sur les images et la superposition d’images, consultez [conteneurs d’images de base](../manage-containers/container-base-images.md).

Pour une présentation complète de fichiers dockerfile, consultez la [référence fichier dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="basic-syntax"></a>Syntaxe de base

Sous sa forme la plus basique, un fichier Dockerfile peut être très simple. L’exemple suivant crée une image, qui comprend IIS et un site « hello world ». Cet exemple inclut des commentaires (signalés par un `#`), qui décrivent chaque étape. Les sections suivantes de cet article approfondissent les règles de syntaxe et les instructions pour les fichiers Dockerfile.

>[!NOTE]
>Un fichier dockerfile doit être créé sans extension. Pour ce faire, dans Windows, créez le fichier avec votre éditeur de votre choix, puis enregistrez-le avec la notation « fichier dockerfile » (y compris les guillemets).

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

Pour obtenir des exemples supplémentaires de fichiers dockerfile pour Windows, consultez le [référentiel fichier dockerfile pour Windows](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## <a name="instructions"></a>Instructions

Les instructions fichier dockerfile fournissent au moteur de l’ancrage les instructions dont il a besoin pour créer une image conteneur. Ces instructions sont exécutées une par une et dans l’ordre. Les exemples suivants sont les instructions les plus couramment utilisées dans fichiers dockerfile. Pour obtenir la liste complète des instructions fichier dockerfile, consultez la [référence fichier dockerfile](https://docs.docker.com/engine/reference/builder/).

### <a name="from"></a>FROM

L’instruction `FROM` définit l’image de conteneur qui sera utilisée pendant le processus de création de l’image. Par exemple, quand vous utilisez l’instruction `FROM mcr.microsoft.com/windows/servercore`, l’image obtenue est dérivée de l’image du système d’exploitation de base Windows Server Core et a une dépendance sur celle-ci. Si l’image spécifiée n’est pas présente sur le système où le processus de génération Docker est en cours d’exécution, le moteur Docker tente de télécharger l’image à partir d’un Registre d’images public ou privé.

Le format de l’instruction FROM se présente comme suit :

```dockerfile
FROM <image>
```

Voici un exemple de la commande FROM :

Pour télécharger la version ltsc2019 de Windows Server Core à partir de la Container Registry Microsoft :
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

Pour plus d’informations, consultez [à partir de la référence](https://docs.docker.com/engine/reference/builder/#from).

### <a name="run"></a>RUN

L’instruction `RUN` spécifie les commandes à exécuter et capturer dans la nouvelle image de conteneur. Ces commandes peuvent concerner notamment l’installation de logiciels ou encore la création de fichiers et de répertoires ainsi que de la configuration de l’environnement.

L’instruction RUN se présente comme suit :

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

La différence entre le formulaire d’exécution et celui de l’interpréteur de commandes réside dans la façon dont l’instruction `RUN` est exécutée. Quand vous utilisez la forme exec, le programme spécifié est exécuté explicitement.

Voici un exemple de formulaire exec :

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

L’image résultante exécute la commande `powershell New-Item c:/test` :

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

Pour contraste, l’exemple suivant exécute la même opération sous forme de Shell :

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

L’image résultante a une instruction d’exécution de `cmd /S /C powershell New-Item c:\test`.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Considérations relatives à l’utilisation de l’exécution avec Windows

Sur Windows, quand vous utilisez l’instruction `RUN` avec le format exec, les barres obliques inverses doivent être placées dans une séquence d’échappement.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Lorsque le programme cible est un programme d’installation Windows, vous devez extraire le programme d’installation à l’aide de l’indicateur `/x:<directory>` avant de lancer la procédure d’installation réelle (sans assistance). Vous devez également attendre la fin de la commande avant de faire quoi que ce soit d’autre. Dans le cas contraire, le processus se termine prématurément sans avoir à installer quoi que ce soit. Pour plus d’informations, consultez l’exemple ci-dessous.

#### <a name="examples-of-using-run-with-windows"></a>Exemples d’utilisation de l’exécution avec Windows

L’exemple suivant fichier dockerfile utilise DISM pour installer IIS dans l’image de conteneur :

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

Cet exemple installe le package redistribuable Visual Studio. `Start-Process` et le paramètre `-Wait` sont utilisés pour exécuter le programme d’installation. Cela permet de s’assurer que l’installation est terminée avant de passer à l’instruction suivante dans le fichier dockerfile.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Pour plus d’informations sur l’instruction RUN, consultez la [référence Run](https://docs.docker.com/engine/reference/builder/#run).

### <a name="copy"></a>COPY

L’instruction `COPY` copie les fichiers et les répertoires dans le système de fichiers du conteneur. Les fichiers et les répertoires doivent se trouver dans un chemin d’accès relatif au fichier dockerfile.

Le format de l’instruction `COPY` se présente comme suit :

```dockerfile
COPY <source> <destination>
```

Si la source ou la destination comprend un espace blanc, placez le chemin d’accès entre crochets et guillemets doubles, comme indiqué dans l’exemple suivant :

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Considérations relatives à l’utilisation de la copie avec Windows

Sur Windows, le format de destination doit utiliser des barres obliques. Par exemple, il s’agit d’instructions `COPY` valides :

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

Pendant ce temps, le format suivant avec des barres obliques inverses ne fonctionnera pas :

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Exemples d’utilisation de la copie avec Windows

L’exemple suivant ajoute le contenu du répertoire source à un répertoire nommé `sqllite` dans l’image de conteneur :

```dockerfile
COPY source /sqlite/
```

L’exemple suivant ajoute tous les fichiers qui commencent par config au répertoire `c:\temp` de l’image de conteneur :

```dockerfile
COPY config* c:/temp/
```

Pour plus d’informations sur l’instruction `COPY`, consultez la [référence de copie](https://docs.docker.com/engine/reference/builder/#copy).

### <a name="add"></a>ADD

L’instruction ADD est semblable à l’instruction COPY, mais avec encore plus de fonctionnalités. En plus de copier des fichiers à partir de l’hôte dans l’image de conteneur, l’instruction `ADD` peut également copier des fichiers depuis un emplacement distant avec une spécification d’URL.

Le format de l’instruction `ADD` se présente comme suit :

```dockerfile
ADD <source> <destination>
```

Si la source ou la destination incluent un espace blanc, placez le chemin entre crochets et guillemets doubles :

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Considérations relatives à l’exécution de la fonction ADD avec Windows

Sur Windows, le format de destination doit utiliser des barres obliques. Par exemple, il s’agit d’instructions `ADD` valides :

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

Pendant ce temps, le format suivant avec des barres obliques inverses ne fonctionnera pas :

```dockerfile
ADD test1.txt c:\temp\
```

En outre, sur Linux, l’instruction `ADD` développera des packages compressés dans le cadre d’une copie. Cette fonctionnalité n’est pas disponible dans Windows.

#### <a name="examples-of-using-add-with-windows"></a>Exemples d’utilisation de l’ajout avec Windows

L’exemple suivant ajoute le contenu du répertoire source à un répertoire nommé `sqllite` dans l’image de conteneur :

```dockerfile
ADD source /sqlite/
```

L’exemple suivant ajoute tous les fichiers qui commencent par « config » dans le répertoire `c:\temp` de l’image de conteneur.

```dockerfile
ADD config* c:/temp/
```

L’exemple suivant télécharge Python pour Windows dans le répertoire `c:\temp` de l’image conteneur.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Pour plus d’informations sur l’instruction `ADD`, consultez [Ajouter une référence](https://docs.docker.com/engine/reference/builder/#add).

### <a name="workdir"></a>WORKDIR

L’instruction `WORKDIR` définit un répertoire de travail pour d’autres instructions pour les fichiers Dockerfile, telles que `RUN` et `CMD`, ainsi que le répertoire de travail pour les instances en cours d’exécution de l’image de conteneur.

Le format de l’instruction `WORKDIR` se présente comme suit :

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Considérations relatives à l’utilisation de WORKDIR avec Windows

Sur Windows, si le répertoire de travail comprend une barre oblique inverse, celle-ci doit être placée dans une séquence d’échappement.

```dockerfile
WORKDIR c:\\windows
```

**Exemples**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

Pour plus d’informations sur l’instruction `WORKDIR`, consultez la [référence WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir).

### <a name="cmd"></a>CMD

L’instruction `CMD` définit la commande par défaut à exécuter lors du déploiement d’une instance de l’image de conteneur. Par exemple, si le conteneur doit héberger un serveur Web NGINX, le `CMD` peut inclure des instructions pour démarrer le serveur Web avec une commande comme `nginx.exe`. Si plusieurs instructions `CMD` sont spécifiées dans un fichier Dockerfile, seule la dernière est évaluée.

Le format de l’instruction `CMD` se présente comme suit :

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Considérations relatives à l’utilisation de CMD avec Windows

Sur Windows, les chemins de fichiers spécifiés dans l’instruction `CMD` doivent utiliser des barres obliques. En outre, les barres obliques inverses `\\` doivent être placées dans une séquence d’échappement. Les instructions `CMD` valides sont les suivantes :

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

Toutefois, le format suivant, sans les barres obliques appropriées, ne fonctionne pas :

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

Pour plus d’informations sur l’instruction `CMD`, consultez la [référence cmd](https://docs.docker.com/engine/reference/builder/#cmd).

## <a name="escape-character"></a>Caractère d'échappement

Dans de nombreux cas, une instruction fichier dockerfile doit s’étendre sur plusieurs lignes. Pour ce faire, vous pouvez utiliser un caractère d’échappement. Le caractère d’échappement de fichier Dockerfile par défaut est une barre oblique inverse `\`. Toutefois, étant donné que la barre oblique inverse est également un séparateur de chemin de fichier dans Windows, son utilisation pour s’étendre sur plusieurs lignes peut entraîner des problèmes. Pour contourner ce point, vous pouvez utiliser une directive d’analyseur pour modifier le caractère d’échappement par défaut. Pour plus d’informations sur les directives de l’analyseur, consultez [directives d’analyseur](https://docs.docker.com/engine/reference/builder/#parser-directives).

L’exemple suivant illustre une instruction RUN unique qui s’étend sur plusieurs lignes à l’aide du caractère d’échappement par défaut :

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Pour modifier le caractère d’échappement, placez une directive d’analyseur d’échappement sur la toute première ligne du fichier Dockerfile. Cela peut être observé dans l’exemple suivant.

>[!NOTE]
>Seules deux valeurs peuvent être utilisées comme caractères d’échappement : `\` et `` ` ``.

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Pour plus d’informations sur la directive d’analyseur d’échappement, voir [directive d’analyseur d’échappement](https://docs.docker.com/engine/reference/builder/#escape).

## <a name="powershell-in-dockerfile"></a>PowerShell dans un fichier Dockerfile

### <a name="powershell-cmdlets"></a>Applets de commande PowerShell

Les applets de commande PowerShell peuvent être exécutées dans un fichier dockerfile avec l’opération `RUN`.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>Appels REST

L’applet de commande `Invoke-WebRequest` de PowerShell peut être utile lors de la collecte d’informations ou de fichiers à partir d’un service Web. Par exemple, si vous générez une image qui comprend Python, vous pouvez définir `$ProgressPreference` sur `SilentlyContinue` pour obtenir des téléchargements plus rapides, comme illustré dans l’exemple suivant.

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
>`Invoke-WebRequest` fonctionne également dans nano Server.

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
>Nano Server ne prend pas actuellement en charge WebClient.

### <a name="powershell-scripts"></a>Scripts PowerShell

Dans certains cas, il peut être utile de copier un script dans les conteneurs que vous utilisez pendant le processus de création de l’image, puis d’exécuter le script à partir du conteneur.

>[!NOTE]
>Cela permet de limiter la mise en cache de la couche d’images et de réduire la lisibilité du fichier dockerfile.

Cet exemple copie un script à partir de l’ordinateur de build dans le conteneur à l’aide de l’instruction `ADD`. Ce script est alors exécuté à l’aide de l’instruction RUN.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>version de l’arrimeur

Une fois qu’un fichier dockerfile a été créé et enregistré sur le disque, vous pouvez exécuter `docker build` pour créer la nouvelle image. La commande `docker build` accepte plusieurs paramètres facultatifs et un chemin d’accès au fichier Dockerfile. Pour obtenir une documentation complète sur la build de l’ancrage, y compris une liste de toutes les options de génération, consultez la [référence de build](https://docs.docker.com/engine/reference/commandline/build/#build).

Le format de la commande `docker build` se présente comme suit :

```dockerfile
docker build [OPTIONS] PATH
```

Par exemple, la commande suivante crée une image nommée « IIS ».

```dockerfile
docker build -t iis .
```

Lorsque le processus de génération a été initialisé, la sortie indique l’État et retourne toutes les erreurs levées.

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

Le résultat est une nouvelle image de conteneur, qui, dans cet exemple, est nommée « IIS ».

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>Informations supplémentaires sur la lecture et les références

- [Optimiser fichiers dockerfile et la build de l’arrimeur pour Windows](optimize-windows-dockerfile.md)
- [Informations de référence sur Dockerfile](https://docs.docker.com/engine/reference/builder/)

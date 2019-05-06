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
ms.openlocfilehash: 9ff6256ab9708533f72e9b3210f8a5fd32f4048a
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610269"
---
# <a name="dockerfile-on-windows"></a>Fichier Dockerfile sur Windows

Le moteur Docker inclut des outils permettant d’automatiser la création d’images de conteneur. Pendant que vous pouvez créer des images de conteneur manuellement en exécutant le `docker commit` commande, adopter un processus de création d’image automatique offre de nombreux avantages, notamment:

- stockage d’images de conteneur sous forme de code;
- nouvelle création rapide et précise d’images de conteneur à des fins de maintenance et de mise à niveau;
- intégration continue entre les images de conteneur et le cycle de développement.

Les composants Docker qui dirigent cette automatisation sont le fichier Dockerfile et la commande `docker build`.

Le fichier Dockerfile est un fichier texte contenant les instructions nécessaires pour créer une nouvelle image de conteneur. Ces instructions incluent l’identification d’une image existante à utiliser comme base, les commandes à exécuter pendant le processus de création d’image et une commande qui s’exécute quand de nouvelles instances de l’image de conteneur sont déployées.

Génération docker est la commande du moteur Docker qui utilise un fichier Dockerfile et déclenche le processus de création d’image.

Cette rubrique vous montre comment utiliser des fichiers Dockerfile avec des conteneurs Windows, comprendre leur syntaxe de base, et quels sont les plus courants instructions Dockerfile.

Ce document aborde le concept d’images de conteneur et les couches d’image de conteneur. Si vous souhaitez en savoir plus sur les images et la superposition d’images, voir [le Guide de démarrage rapide consacré aux images](../quick-start/quick-start-images.md).

Pour une étude complète des fichiers Dockerfile, voir la [référence du fichier Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="basic-syntax"></a>Syntaxe de base

Sous sa forme la plus basique, un fichier Dockerfile peut être très simple. L’exemple suivant crée une image, qui comprendIIS et un site «hello world». Cet exemple inclut des commentaires (signalés par un `#`), qui décrivent chaque étape. Les sections suivantes de cet article approfondissent les règles de syntaxe et les instructions pour les fichiers Dockerfile.

>[!NOTE]
>Un fichier Dockerfile doit être créé sans extension. Pour ce faire, dans Windows, créez le fichier avec l’éditeur de votre choix, puis enregistrez ce dernier avec la notation «Dockerfile» (y compris les guillemets).

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM microsoft/windowsservercore

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Pour obtenir des exemples supplémentaires de fichiers Dockerfile pour Windows, voir le [référentiel de fichier Dockerfile pour Windows](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## <a name="instructions"></a>Instructions

Les instructions Dockerfile fournissent au moteur Docker les instructions que nécessaires pour créer une image de conteneur. Ces instructions sont effectuées un-à-un et dans l’ordre. Les exemples suivants correspondent aux instructions plus couramment utilisées dans les fichiers Dockerfile. Pour obtenir une liste complète des instructions Dockerfile, voir la [référence du fichier Dockerfile](https://docs.docker.com/engine/reference/builder/).

### <a name="from"></a>FROM

L’instruction `FROM` définit l’image de conteneur qui sera utilisée pendant le processus de création de l’image. Par exemple, quand vous utilisez l’instruction `FROM microsoft/windowsservercore`, l’image obtenue est dérivée de l’image du système d’exploitation de base Windows Server Core et a une dépendance sur celle-ci. Si l’image spécifiée n’est pas présente sur le système où le processus de génération Docker est en cours d’exécution, le moteur Docker tente de télécharger l’image à partir d’un Registre d’images public ou privé.

Format de l’instruction FROM passe comme suit:

```dockerfile
FROM <image>
```

Voici un exemple de la commande FROM:

```dockerfile
FROM microsoft/windowsservercore
```

Pour plus d’informations, voir la [référence FROM](https://docs.docker.com/engine/reference/builder/#from).

### <a name="run"></a>RUN

L’instruction `RUN` spécifie les commandes à exécuter et capturer dans la nouvelle image de conteneur. Ces commandes peuvent concerner notamment l’installation de logiciels ou encore la création de fichiers et de répertoires ainsi que de la configuration de l’environnement.

L’instruction RUN passe comme suit:

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

La différence entre la forme exec et l’interpréteur de commandes est dans la façon dont le `RUN` instruction est exécutée. Quand vous utilisez la forme exec, le programme spécifié est exécuté explicitement.

Voici un exemple de la forme exec:

```dockerfile
FROM microsoft/windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

L’image obtenue s’exécute le `powershell New-Item c:/test` commande:

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

En revanche, l’exemple suivant exécute la même opération sous forme de l’interpréteur de commandes:

```dockerfile
FROM microsoft/windowsservercore

RUN powershell New-Item c:\test
```

L’image obtenue a une instruction d’exécution `cmd /S /C powershell New-Item c:\test`.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Considérations pour l’utilisation s’exécute avec Windows

Sur Windows, quand vous utilisez l’instruction `RUN` avec le format exec, les barres obliques inverses doivent être placées dans une séquence d’échappement.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Lorsque le programme cible est un programme d’installation de Windows, vous devez extraire le programme d’installation par le biais de la `/x:<directory>` indicateur avant de pouvoir lancer la procédure d’installation (silencieuse) réel. Vous devez également attendre la commande quitter avant d’effectuer tout autre élément. Dans le cas contraire, le processus se termine prématurément sans rien installer. Pour plus d’informations, consultez l’exemple ci-dessous.

#### <a name="examples-of-using-run-with-windows"></a>Exemples d’utilisation de s’exécuter avec Windows

L’exemple suivant Dockerfile utilise DISM pour installer IIS dans l’image de conteneur:

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

Cet exemple installe le package redistribuable Visual Studio. `Start-Process` et le `-Wait` paramètre sont utilisés pour exécuter le programme d’installation. Cela garantit que l’installation est terminée avant de passer à l’instruction suivante dans le fichier Dockerfile.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Pour plus d’informations sur l’instruction RUN, voir l' [exécuter référence](https://docs.docker.com/engine/reference/builder/#run).

### <a name="copy"></a>COPY

Le `COPY` instruction copie les fichiers et les répertoires système de fichiers du conteneur. Les fichiers et répertoires doivent être dans un chemin d’accès relatif au fichier Dockerfile.

Le `COPY` format de l’instruction passe comme suit:

```dockerfile
COPY <source> <destination>
```

Si la source ou la destination contient un espace blanc, placez le chemin d’accès crochets et guillemets doubles, comme illustré dans l’exemple suivant:

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Considérations pour l’utilisation de la copie avec Windows

Sur Windows, le format de destination doit utiliser des barres obliques. Par exemple, ils ne sont pas valides `COPY` instructions:

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

Par ailleurs, le format suivant de barres obliques inverses ne fonctionnera pas:

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Exemples d’utilisation de copie avec Windows

L’exemple suivant ajoute le contenu du répertoire source vers un répertoire nommé `sqllite` dans l’image de conteneur:

```dockerfile
COPY source /sqlite/
```

L’exemple suivant ajoute tous les fichiers qui commencent par la configuration pour le `c:\temp` répertoire de l’image de conteneur:

```dockerfile
COPY config* c:/temp/
```

Pour plus d’informations sur la `COPY` obtenir des instructions, voir la [référence de la copie](https://docs.docker.com/engine/reference/builder/#copy).

### <a name="add"></a>ADD

L’instruction ADD est à l’instruction COPY, mais avec des fonctionnalités encore plus. En plus de copier des fichiers à partir de l’hôte dans l’image de conteneur, l’instruction `ADD` peut également copier des fichiers depuis un emplacement distant avec une spécification d’URL.

Le `ADD` format de l’instruction passe comme suit:

```dockerfile
ADD <source> <destination>
```

Si la source ou la destination espace blanc, placez le chemin d’accès des crochets et guillemets doubles:

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Considérations pour l’ajout en cours d’exécution avec Windows

Sur Windows, le format de destination doit utiliser des barres obliques. Par exemple, ils ne sont pas valides `ADD` instructions:

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

Par ailleurs, le format suivant de barres obliques inverses ne fonctionnera pas:

```dockerfile
ADD test1.txt c:\temp\
```

En outre, sur Linux, l’instruction `ADD` développera des packages compressés dans le cadre d’une copie. Cette fonctionnalité n’est pas disponible dans Windows.

#### <a name="examples-of-using-add-with-windows"></a>AJOUTENT des exemples d’utilisation avec Windows

L’exemple suivant ajoute le contenu du répertoire source vers un répertoire nommé `sqllite` dans l’image de conteneur:

```dockerfile
ADD source /sqlite/
```

L’exemple suivant ajoute tous les fichiers qui commencent par «configuration» à la `c:\temp` répertoire de l’image de conteneur.

```dockerfile
ADD config* c:/temp/
```

L’exemple suivant télécharge Python pour Windows dans le `c:\temp` répertoire de l’image de conteneur.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Pour plus d’informations sur la `ADD` obtenir des instructions, voir [Ajouter une référence](https://docs.docker.com/engine/reference/builder/#add).

### <a name="workdir"></a>WORKDIR

L’instruction `WORKDIR` définit un répertoire de travail pour d’autres instructions pour les fichiers Dockerfile, telles que `RUN` et `CMD`, ainsi que le répertoire de travail pour les instances en cours d’exécution de l’image de conteneur.

Le `WORKDIR` format de l’instruction passe comme suit:

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Considérations pour l’utilisation sur WORKDIR avec Windows

Sur Windows, si le répertoire de travail comprend une barre oblique inverse, celle-ci doit être placée dans une séquence d’échappement.

```dockerfile
WORKDIR c:\\windows
```

**Exemples**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

Pour plus d’informations sur la `WORKDIR` obtenir des instructions, voir la [référence sur WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir).

### <a name="cmd"></a>CMD

L’instruction `CMD` définit la commande par défaut à exécuter lors du déploiement d’une instance de l’image de conteneur. Par exemple, si le conteneur doit héberger un serveur web NGINX, le `CMD` peut inclure des instructions pour démarrer le serveur web avec une commande comme `nginx.exe`. Si plusieurs instructions `CMD` sont spécifiées dans un fichier Dockerfile, seule la dernière est évaluée.

Le `CMD` format de l’instruction passe comme suit:

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Considérations pour l’utilisation de CMD avec Windows

Sur Windows, les chemins de fichiers spécifiés dans l’instruction `CMD` doivent utiliser des barres obliques. En outre, les barres obliques inverses `\\` doivent être placées dans une séquence d’échappement. Les éléments suivants sont valides `CMD` instructions:

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

Toutefois, le format suivant sans les barres obliques appropriées ne fonctionnera pas:

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

Pour plus d’informations sur la `CMD` obtenir des instructions, voir la [référence sur CMD](https://docs.docker.com/engine/reference/builder/#cmd).

## <a name="escape-character"></a>Caractère d’échappement

Dans de nombreux cas, une instruction de fichier Dockerfile devront s’étendre sur plusieurs lignes. Pour ce faire, vous pouvez utiliser un caractère d’échappement. Le caractère d’échappement de fichier Dockerfile par défaut est une barre oblique inverse `\`. Toutefois, étant donné que la barre oblique inverse est également un séparateur de chemin d’accès de fichier dans Windows, l’utiliser pour s’étendre sur plusieurs lignes peut provoquer des problèmes. Pour contourner ce problème, vous pouvez utiliser une directive d’analyseur pour modifier le caractère d’échappement par défaut. Pour plus d’informations sur les directives d’analyseur, consultez les [directives d’analyseur](https://docs.docker.com/engine/reference/builder/#parser-directives).

L’exemple suivant montre une instruction RUN unique qui s’étend sur plusieurs lignes à l’aide du caractère d’échappement par défaut:

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Pour modifier le caractère d’échappement, placez une directive d’analyseur d’échappement sur la toute première ligne du fichier Dockerfile. Cela est visible dans l’exemple suivant.

>[!NOTE]
>Seulement deux valeurs peuvent être utilisées comme des caractères d’échappement: `\` et `` ` ``.

```dockerfile
# escape=`

FROM microsoft/windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Pour plus d’informations sur la directive d’analyseur d’échappement, voir [directive d’analyseur d’échappement](https://docs.docker.com/engine/reference/builder/#escape).

## <a name="powershell-in-dockerfile"></a>PowerShell dans un fichier Dockerfile

### <a name="powershell-cmdlets"></a>Applets de commande PowerShell

Applets de commande PowerShell peuvent être exécutées dans un fichier Dockerfile avec le `RUN` opération.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>Appels REST

De PowerShell `Invoke-WebRequest` applet de commande peut être utile lors de la collecte des informations ou des fichiers à partir d’un service web. Par exemple, si vous générez une image qui inclut Python, vous pouvez définir `$ProgressPreference` à `SilentlyContinue` pour que les téléchargements plus rapides, comme illustré dans l’exemple suivant.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` fonctionne également dans Nano Server.

Pour télécharger les fichiers avec PowerShell pendant le processus de création d’image, une autre option consiste à employer la bibliothèque WebClient .NET. Cela peut améliorer les performances en matière de téléchargement. L’exemple suivant télécharge le logiciel Python à l’aide de la bibliothèque WebClient.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>NANO Server ne prend pas en charge WebClient.

### <a name="powershell-scripts"></a>Scripts PowerShell

Dans certains cas, il peut être utile de copier un script dans le conteneur que vous utilisez pendant le processus de création d’image, puis exécutez le script à partir du conteneur.

>[!NOTE]
>Cela limite aucune couche d’image mise en cache et réduisez la lisibilité du fichier Dockerfile.

Cet exemple copie un script à partir de l’ordinateur de build dans le conteneur à l’aide de l’instruction `ADD`. Ce script est alors exécuté à l’aide de l’instruction RUN.

```dockerfile
FROM microsoft/windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Génération docker

Une fois qu’un fichier Dockerfile a été créé et enregistré sur le disque, vous pouvez exécuter `docker build` pour créer l’image. La commande `docker build` accepte plusieurs paramètres facultatifs et un chemin d’accès au fichier Dockerfile. Pour obtenir une documentation complète sur Docker Build, notamment une liste de tous les options de build, voir la [build de référence](https://docs.docker.com/engine/reference/commandline/build/#build).

Le format de le `docker build` commande passe comme suit:

```dockerfile
docker build [OPTIONS] PATH
```

Par exemple, la commande suivante crée une image nommée «iis».

```dockerfile
docker build -t iis .
```

Lorsque le processus de génération a été lancé, la sortie sera indiquent l’état et retourne les erreurs levées.

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM micrsoft/windowsservercore
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

Le résultat est une nouvelle image de conteneur, ce qui, dans cet exemple est nommée «iis».

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>Les références et obtenir des informations supplémentaires

- [Optimiser les fichiers Dockerfile et la build Docker pour Windows](optimize-windows-dockerfile.md)
- [Référence du fichier Dockerfile](https://docs.docker.com/engine/reference/builder/)

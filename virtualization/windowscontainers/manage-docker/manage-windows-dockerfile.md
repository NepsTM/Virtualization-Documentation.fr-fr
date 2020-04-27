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
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/24/2020
ms.locfileid: "74909659"
---
# <a name="dockerfile-on-windows"></a>Fichier Dockerfile sur Windows

Le moteur Docker comprend des outils qui automatisent la création d'images de conteneur. Rien ne vous empêche de créer des images de conteneur manuellement en exécutant la commande `docker commit`, mais l'adoption d'un processus de création d'image automatisé présente de nombreux avantages, parmi lesquels :

- stockage d’images de conteneur sous forme de code ;
- nouvelle création rapide et précise d’images de conteneur à des fins de maintenance et de mise à niveau ;
- intégration continue entre les images de conteneur et le cycle de développement.

Les composants Docker qui dirigent cette automatisation sont le fichier Dockerfile et la commande `docker build`.

Le fichier Dockerfile est un fichier texte qui contient les instructions nécessaires à la création d'une image de conteneur. Ces instructions incluent l’identification d’une image existante à utiliser comme base, les commandes à exécuter pendant le processus de création d’image et une commande qui s’exécute quand de nouvelles instances de l’image de conteneur sont déployées.

Docker build est la commande du moteur Docker qui utilise un fichier Dockerfile et déclenche le processus de création d'image.

Cette rubrique vous explique comment utiliser des fichiers Dockerfile avec des conteneurs Windows, vous aide à comprendre leur syntaxe de base et vous présente les instructions les plus courantes des fichiers Dockerfile.

Ce document aborde le concept d'images de conteneur et de couches d'images de conteneur. Si vous souhaitez en savoir plus sur les images et la superposition d'images, consultez [Images de base de conteneur](../manage-containers/container-base-images.md).

Pour une présentation complète des fichiers Dockerfile, consultez les [Informations de référence sur les fichiers Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="basic-syntax"></a>Syntaxe de base

Sous sa forme la plus basique, un fichier Dockerfile peut être très simple. L’exemple suivant crée une image, qui comprend IIS et un site « hello world ». Cet exemple inclut des commentaires (signalés par un `#`), qui décrivent chaque étape. Les sections suivantes de cet article approfondissent les règles de syntaxe et les instructions pour les fichiers Dockerfile.

>[!NOTE]
>Le fichier Dockerfile créé ne doit pas comporter d'extension. Pour ce faire dans Windows, créez le fichier à l'aide de l'éditeur de votre choix, puis enregistrez-le en utilisant la notation "Dockerfile" (guillemets compris).

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

Pour accéder à d'autres exemples de fichiers Dockerfile pour Windows, consultez le [Référentiel de fichiers Dockerfile pour Windows](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## <a name="instructions"></a>Instructions

Les instructions relatives aux fichiers Dockerfile fournissent au moteur Docker les instructions nécessaires à la création d'une image de conteneur. Ces instructions sont exécutées une par une et dans l'ordre. Les exemples suivants correspondent aux instructions les plus couramment utilisées dans les fichiers Dockerfile. Pour obtenir la liste complète des instructions relatives aux fichiers Dockerfile, consultez les [Informations de référence sur les fichiers Dockerfile](https://docs.docker.com/engine/reference/builder/).

### <a name="from"></a>FROM

L’instruction `FROM` définit l’image de conteneur qui sera utilisée pendant le processus de création de l’image. Par exemple, quand vous utilisez l’instruction `FROM mcr.microsoft.com/windows/servercore`, l’image obtenue est dérivée de l’image du système d’exploitation de base Windows Server Core et a une dépendance sur celle-ci. Si l’image spécifiée n’est pas présente sur le système où le processus de génération Docker est en cours d’exécution, le moteur Docker tente de télécharger l’image à partir d’un Registre d’images public ou privé.

Le format de l'instruction FROM se présente comme suit :

```dockerfile
FROM <image>
```

Voici un exemple de commande FROM :

Pour télécharger la version ltsc2019 de Windows Server Core à partir de Microsoft Container Registry (MCR) :
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

Pour plus d'informations, consultez les [Informations de référence sur FROM](https://docs.docker.com/engine/reference/builder/#from).

### <a name="run"></a>RUN

L’instruction `RUN` spécifie les commandes à exécuter et capturer dans la nouvelle image de conteneur. Ces commandes peuvent concerner notamment l’installation de logiciels ou encore la création de fichiers et de répertoires ainsi que de la configuration de l’environnement.

L'instruction RUN se présente comme suit :

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

La différence entre la forme exec et la forme shell réside dans la façon dont l'instruction `RUN` est exécutée. Quand vous utilisez la forme exec, le programme spécifié est exécuté explicitement.

Voici un exemple de forme exec :

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

L'image qui en résulte exécute la commande `powershell New-Item c:/test` :

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

En revanche, l'exemple suivant exécute cette même opération sous la forme shell :

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

L'instruction d'exécution de l'image qui en résulte est la suivante : `cmd /S /C powershell New-Item c:\test`.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Éléments à prendre en considération pour utiliser RUN avec Windows

Sur Windows, quand vous utilisez l’instruction `RUN` avec le format exec, les barres obliques inverses doivent être placées dans une séquence d’échappement.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Lorsque le programme cible est un programme d'installation Windows, vous devez l'extraire à l'aide de l'indicateur `/x:<directory>` avant de pouvoir lancer la procédure d'installation (silencieuse) proprement dite. Vous devez également attendre la fin de l'exécution de la commande avant de faire quoi que ce soit d'autre. Sinon le processus se termine prématurément sans avoir installé quoi que ce soit. Pour plus d’informations, consultez l’exemple ci-dessous.

#### <a name="examples-of-using-run-with-windows"></a>Exemples d'utilisation de RUN avec Windows

L'exemple de fichier Dockerfile suivant utilise DISM pour installer IIS dans l'image du conteneur :

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

Cet exemple installe le package redistribuable Visual Studio. `Start-Process` et le paramètre `-Wait` sont utilisés pour exécuter le programme d'installation. Cela permet de s'assurer que l'installation est terminée avant de passer à l'instruction suivante dans le fichier Dockerfile.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Pour plus d'informations sur l'instruction RUN, consultez les [Informations de référence sur RUN](https://docs.docker.com/engine/reference/builder/#run).

### <a name="copy"></a>COPY

L'instruction `COPY` copie les fichiers et les répertoires dans le système de fichiers du conteneur. Les fichiers et répertoires doivent se trouver dans un chemin relatif au fichier Dockerfile.

Le format de l'instruction `COPY` se présente comme suit :

```dockerfile
COPY <source> <destination>
```

Si la source ou la destination comprend des espaces blancs, placez le chemin entre crochets et guillemets doubles, comme indiqué dans l'exemple suivant :

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Éléments à prendre en considération pour utiliser COPY avec Windows

Sur Windows, le format de destination doit utiliser des barres obliques. Voici, par exemple, des instructions `COPY` valides :

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

Le format suivant, avec barres obliques inverses, ne fonctionnera pas :

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Exemples d'utilisation de COPY avec Windows

L'exemple suivant ajoute le contenu du répertoire source à un répertoire nommé `sqllite` dans l'image de conteneur.

```dockerfile
COPY source /sqlite/
```

L'exemple suivant ajoute tous les fichiers qui commencent par config au répertoire `c:\temp` de l'image de conteneur :

```dockerfile
COPY config* c:/temp/
```

Pour plus d'informations sur l'instruction `COPY`, consultez les [Informations de référence sur COPY](https://docs.docker.com/engine/reference/builder/#copy).

### <a name="add"></a>ADD

L'instruction ADD est semblable à l'instruction COPY, mais avec encore plus de possibilités. En plus de copier des fichiers à partir de l’hôte dans l’image de conteneur, l’instruction `ADD` peut également copier des fichiers depuis un emplacement distant avec une spécification d’URL.

Le format de l'instruction `ADD` se présente comme suit :

```dockerfile
ADD <source> <destination>
```

Si la source ou la destination contient des espaces blancs, placez le chemin entre crochets et guillemets doubles :

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Éléments à prendre en considération pour utiliser ADD avec Windows

Sur Windows, le format de destination doit utiliser des barres obliques. Voici, par exemple, des instructions `ADD` valides :

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

Le format suivant, avec barres obliques inverses, ne fonctionnera pas :

```dockerfile
ADD test1.txt c:\temp\
```

En outre, sur Linux, l’instruction `ADD` développera des packages compressés dans le cadre d’une copie. Cette fonctionnalité n’est pas disponible dans Windows.

#### <a name="examples-of-using-add-with-windows"></a>Exemples d'utilisation de ADD avec Windows

L'exemple suivant ajoute le contenu du répertoire source à un répertoire nommé `sqllite` dans l'image de conteneur.

```dockerfile
ADD source /sqlite/
```

L'exemple suivant ajoute tous les fichiers qui commencent par « config » au répertoire `c:\temp` de l'image de conteneur.

```dockerfile
ADD config* c:/temp/
```

L'exemple suivant télécharge Python pour Windows dans le répertoire `c:\temp` de l'image de conteneur.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Pour plus d'informations sur l'instruction `ADD`, consultez les [Informations de référence sur ADD](https://docs.docker.com/engine/reference/builder/#add).

### <a name="workdir"></a>WORKDIR

L’instruction `WORKDIR` définit un répertoire de travail pour d’autres instructions pour les fichiers Dockerfile, telles que `RUN` et `CMD`, ainsi que le répertoire de travail pour les instances en cours d’exécution de l’image de conteneur.

Le format de l'instruction `WORKDIR` se présente comme suit :

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Éléments à prendre en considération pour utiliser WORKDIR avec Windows

Sur Windows, si le répertoire de travail comprend une barre oblique inverse, celle-ci doit être placée dans une séquence d’échappement.

```dockerfile
WORKDIR c:\\windows
```

**Exemples**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

Pour plus d'informations sur l'instruction `WORKDIR`, consultez les [Informations de référence sur WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir).

### <a name="cmd"></a>CMD

L’instruction `CMD` définit la commande par défaut à exécuter lors du déploiement d’une instance de l’image de conteneur. Par exemple, si le conteneur doit héberger un serveur web NGINX, l'instruction `CMD` peut inclure des instructions de démarrage du serveur web avec une commande comme `nginx.exe`. Si plusieurs instructions `CMD` sont spécifiées dans un fichier Dockerfile, seule la dernière est évaluée.

Le format de l'instruction `CMD` se présente comme suit :

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Éléments à prendre en considération pour utiliser CMD avec Windows

Sur Windows, les chemins de fichiers spécifiés dans l’instruction `CMD` doivent utiliser des barres obliques. En outre, les barres obliques inverses `\\` doivent être placées dans une séquence d’échappement. Les instructions `CMD` valides sont les suivantes :

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

En revanche, le format suivant, sans les barres obliques appropriées, ne fonctionnera pas :

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

Pour plus d'informations sur l'instruction `CMD`, consultez les [Informations de référence sur CMD](https://docs.docker.com/engine/reference/builder/#cmd).

## <a name="escape-character"></a>Caractère d'échappement

Souvent, une instruction de fichier Dockerfile doit occuper plusieurs lignes. Pour cela, vous pouvez utiliser un caractère d'échappement. Le caractère d’échappement de fichier Dockerfile par défaut est une barre oblique inverse `\`. Cependant, comme la barre oblique inverse est également un séparateur de chemin de fichier dans Windows, son utilisation pour occuper plusieurs lignes peut entraîner des problèmes. Pour y remédier, vous pouvez utiliser une directive d'analyseur afin de modifier le caractère d'échappement par défaut. Pour plus d'informations sur les directives d'analyseur, consultez [Directives d'analyseur](https://docs.docker.com/engine/reference/builder/#parser-directives).

L'exemple suivant présente une instruction RUN unique qui occupe plusieurs lignes grâce au caractère d'échappement par défaut.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Pour modifier le caractère d’échappement, placez une directive d’analyseur d’échappement sur la toute première ligne du fichier Dockerfile. Une illustration figure dans l'exemple suivant.

>[!NOTE]
>Seules deux valeurs peuvent être utilisées en tant que caractères d'échappement : `\` et `` ` ``.

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Pour plus d'informations sur la directive d'analyseur d'échappement, consultez [Directive d'analyseur d'échappement](https://docs.docker.com/engine/reference/builder/#escape).

## <a name="powershell-in-dockerfile"></a>PowerShell dans un fichier Dockerfile

### <a name="powershell-cmdlets"></a>Applets de commande PowerShell

Les cmdlets PowerShell peuvent être exécutées dans un fichier Dockerfile avec l'opération `RUN`.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>Appels REST

La cmdlet `Invoke-WebRequest` de PowerShell peut s'avérer utile lors de la collecte d'informations ou de fichiers à partir d'un service web. Par exemple, si vous créez une image qui inclut Python, vous pouvez définir `$ProgressPreference` sur `SilentlyContinue` pour bénéficier de téléchargements plus rapides, comme illustré dans l'exemple suivant.

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
>`Invoke-WebRequest` fonctionne également dans Nano Server.

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
>Nano Server ne prend actuellement pas en charge WebClient.

### <a name="powershell-scripts"></a>Scripts PowerShell

Dans certains cas, il peut être utile de copier un script dans les conteneurs que vous utilisez au cours du processus de création d'image, puis d'exécuter le script à partir du conteneur.

>[!NOTE]
>Vous limitez ainsi la mise en cache des couches d'images et réduisez la lisibilité du fichier Dockerfile.

Cet exemple copie un script à partir de l’ordinateur de build dans le conteneur à l’aide de l’instruction `ADD`. Ce script est alors exécuté à l’aide de l’instruction RUN.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Docker build

Une fois qu'un fichier Dockerfile a été créé et enregistré sur disque, vous pouvez exécuter la commande `docker build` pour créer l'image. La commande `docker build` accepte plusieurs paramètres facultatifs et un chemin d’accès au fichier Dockerfile. Pour obtenir une documentation complète sur Docker build, avec notamment la liste de toutes les options de build, consultez les [Informations de référence sur build](https://docs.docker.com/engine/reference/commandline/build/#build).

Le format de la commande `docker build` se présente comme suit :

```dockerfile
docker build [OPTIONS] PATH
```

Par exemple, la commande suivante crée une image nommée « iis ».

```dockerfile
docker build -t iis .
```

Une fois le processus build lancé, la sortie indique l'état et renvoie les erreurs éventuelles.

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

Le résultat obtenu est une nouvelle image de conteneur, nommée « iis » dans cet exemple.

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>Informations et références supplémentaires

- [Optimiser les fichiers Dockerfile et Docker build pour Windows](optimize-windows-dockerfile.md)
- [Informations de référence sur les fichiers Dockerfile](https://docs.docker.com/engine/reference/builder/)

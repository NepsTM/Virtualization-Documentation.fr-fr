---
author: neilpeterson
---

# Images de conteneur

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.**

Les images de conteneur sont utilisées pour déployer des conteneurs. Ces images peuvent inclure un système d’exploitation, des applications et toutes les dépendances d’application. Par exemple, vous pouvez développer une image de conteneur qui a été préconfigurée avec Nano Server, IIS et une application s’exécutant dans IIS. Cette image de conteneur peut alors être stockée dans un Registre de conteneur pour être utilisée plus tard, déployée sur un hôte de conteneur Windows (localement, dans le cloud, ou même dans un service de conteneur) et utilisée également comme base pour une nouvelle image de conteneur.

Il existe deux types d’images de conteneur :

- **Images de système d’exploitation de base** : elles sont fournies par Microsoft et incluent les composants principaux du système d’exploitation.
- **Images de conteneur** : une image de conteneur personnalisée est dérivée d’une image de système d’exploitation de base.

## Images de système d’exploitation de base

### Installer une image

Les images de système d’exploitation du conteneur peuvent être installées pour la gestion PowerShell et Docker à l’aide du module PowerShell ContainerProvider. Vous devez installer ce module avant de l’utiliser. La commande suivante peut être utilisée pour installer le module.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

Une fois installé, il est possible de retourner la liste des images de système d’exploitation à l’aide de `Find-ContainerImage`.

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

Pour télécharger et installer l’image du système d’exploitation de base Nano Server, exécutez la commande suivante. Le paramètre `-version` est facultatif. Sans version d’image de système d’exploitation de base spécifiée, la dernière version est installée.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

Cette commande permet de télécharger et d’installer l’image du système d’exploitation de base Windows Server Core. Le paramètre `-version` est facultatif. Sans version d’image de système d’exploitation de base spécifiée, la dernière version est installée.

> **Problème** : Les applets de commande Save-ContainerImage et Install-ContainerImage peuvent ne pas fonctionner avec une image de conteneur WindowsServerCore dans une session de communication à distance PowerShell. **Solution de contournement :** ouvrez une session sur l’ordinateur à l’aide du Bureau à distance et utilisez directement l’applet de commande Save-ContainerImage.

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0

Downloaded in 0 hours, 2 minutes, 28 seconds.
```

Utilisez la commande `Get-ContainerImage` pour vérifier que les images ont été installées.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

> **Install-ContainerImage** installe une image de système d’exploitation de base à utiliser dans des conteneurs gérés PowerShell ou Docker. Si l’image de système d’exploitation de base est téléchargée, mais ne s’affiche pas lors de l’exécution de la commande `docker images`, redémarrez le service Docker à l’aide du panneau de contrôle Services ou de la commande 'sc docker stop' suivie de la commande 'sc docker start'.

### Installation hors connexion

Les images de système d’exploitation de base peuvent également être installées sans connexion Internet. Dans ce cas, les images sont téléchargées depuis un ordinateur disposant d’une connexion Internet, copiées sur le système cible, puis importées à l’aide de la commande `Install-ContainerOSImages`.

Avant de télécharger l’image de système d’exploitation de base, préparez le système à l’aide du fournisseur d’images de conteneur en exécutant la commande suivante.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

Retournez une liste d’images à partir du gestionnaire de package PowerShell OneGet :

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

Pour télécharger une image, utilisez la commande `Save-ContainerImage`.

```powershell
PS C:\> Save-ContainerImage -Name NanoServer -Destination c:\container-image\NanoServer.wim
```

L’image de conteneur téléchargée peut maintenant être copiée sur un autre hôte de conteneur et installée à l’aide de la commande `Install-ContainerOSImage`.

```powershell
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### Marquer des images

Quand vous référencez une image de conteneur par son nom, le moteur Docker recherche la dernière version de l’image. Si la version la plus récente ne peut pas être déterminée, l’erreur suivante est déclenchée.

```powershell
PS C:\> docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

Une fois que les images de système d’exploitation de base Windows Server Core ou Nano Server sont installées, vous devez marquer leur version à l’aide de l’indicateur 'latest'. Pour ce faire, utilisez la commande `docker tag`.

Pour plus d’informations sur `docker tag`, consultez l’article relatif au [marquage et aux transmissions de type Push et Pull des images sur docker.com](https://docs.docker.com/mac/step_six/).

```powershell
PS C:\> docker tag <image id> windowsservercore:latest
```

Quand elle est marquée, la sortie de `docker images` affiche deux versions de la même image, l’une avec l’indicateur de la version de l’image et l’autre avec l’indicateur 'latest'. L’image peut désormais être référencée par son nom.

```powershell
PS C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14289.1000     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14289.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### Désinstaller une image de système d’exploitation

Les images de système d’exploitation de base peuvent être désinstallées à l’aide de la commande `Uninstall-ContainerOSImage`. L’exemple suivant désinstalle l’image de système d’exploitation de base NanoServer.

```powershell
Get-ContainerImage -Name NanoServer | Uninstall-ContainerOSImage
```

## Images de conteneur PowerShell

### Répertorier des images

Pour obtenir la liste des images présentes sur l’hôte du conteneur, exécutez la commande `Get-ContainerImage`. Le type d’image de conteneur est déterminé avec la propriété `IsOSImage`.

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### Créer une image

Vous pouvez créer une image de conteneur à partir d’un conteneur existant. Pour cela, utilisez la commande `New-ContainerImage`.

```powershell
PS C:\> New-ContainerImage -Container $container -Publisher Demo -Name DemoImage -Version 1.0
```

### Supprimer une image

Les images de conteneur ne peuvent pas être supprimées si un conteneur, même dans un état arrêté, a une dépendance sur l’image.

Supprimez une image unique avec PowerShell.

```powershell
PS C:\> Get-ContainerImage -Name newimage | Remove-ContainerImage -Force
```

### Dépendance d’image

Quand une image est créée, elle devient dépendante de l’image à partir de laquelle elle a été créée. Vous pouvez afficher cette dépendance à l’aide de la commande `Get-ContainerImage`. Si une image parente n’est pas répertoriée, l’image est une image de système d’exploitation de base.

```powershell
PS C:\> Get-ContainerImage | select Name, ParentImage

Name              ParentImage
----              -----------
NanoServerIIS     ContainerImage (Name = 'NanoServer') [Publisher = 'CN=Microsoft', Version = '10.0.10586.0']
NanoServer
WindowsServerCore
```

### Déplacer le référentiel d’images

Quand une nouvelle image de conteneur est créée à l’aide de la commande `New-ContainerImage`, l’image est stockée à l’emplacement par défaut : C:\ProgramData\Microsoft\Windows\Hyper-V\Container Image Store. Ce référentiel peut être déplacé à l’aide de la commande `Move-ContainerImageRepository`. Par exemple, ce qui suit créerait un nouveau référentiel d’images de conteneur à l’emplacement suivant : c:\container-images.

```powershell
Move-ContainerImageRepository -Path c:\container-images
```
> Le chemin utilisé avec la commande `Move-ContainerImageRepository` ne doit pas déjà exister quand vous exécutez la commande.

## Images de conteneur Docker

### Répertorier des images

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### Créer une image

Vous pouvez créer une image de conteneur à partir d’un conteneur existant. Pour ce faire, utilisez la commande `docker commit`. L’exemple suivant crée une image de conteneur nommée « windowsservercoreiis ».

```powershell
C:\> docker commit 475059caef8f windowsservercoreiis

ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### Supprimer une image

Les images de conteneur ne peuvent pas être supprimées si un conteneur, même dans un état arrêté, a une dépendance sur l’image.

Quand vous supprimez une image avec docker, les images peuvent être référencées par ID ou nom d’image.

```powershell
C:\> docker rmi windowsservercoreiis

Untagged: windowsservercoreiis:latest
Deleted: ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### Dépendance d’image

Pour visualiser les dépendances d’image avec Docker, utilisez la commande `docker history`.

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Hub Docker

Le Registre du hub Docker contient des images prédéfinies qui peuvent être téléchargées sur un hôte de conteneur. Une fois ces images téléchargées, elles peuvent servir comme base pour les applications de conteneur Windows.

Pour afficher la liste des images disponibles à partir du hub Docker, utilisez la commande `docker search`. Remarque : L’image de système d’exploitation de base Windows Server Core ou Nano Server doit être installée avant l’extraction des images dépendantes à partir du hub Docker.

> Les images dont le nom commence par « nano- » ont une dépendance à l’image de système d’exploitation de base Nano Server.

```powershell
C:\> docker search *

NAME                    DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/aspnet        ASP.NET 5 framework installed in a Windows...   1         [OK]       [OK]
microsoft/django        Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35      .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/golang        Go Programming Language installed in a Win...   1                    [OK]
microsoft/httpd         Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis           Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/mongodb       MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/mysql         MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/nginx         Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/node          Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/php           PHP running on Internet Information Servic...   1                    [OK]
microsoft/python        Python installed in a Windows Server Core ...   1                    [OK]
microsoft/rails         Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/redis         Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/ruby          Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sqlite        SQLite installed in a Windows Server Core ...   1                    [OK]
microsoft/nano-golang   Go Programming Language installed in a Nan...   1                    [OK]
microsoft/nano-httpd    Apache httpd installed in a Nano Server ba...   1                    [OK]
microsoft/nano-iis      Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/nano-mysql    MySQL installed in a Nano Server based con...   1                    [OK]
microsoft/nano-nginx    Nginx installed in a Nano Server based con...   1                    [OK]
microsoft/nano-node     Node installed in a Nano Server based cont...   1                    [OK]
microsoft/nano-python   Python installed in a Nano Server based co...   1                    [OK]
microsoft/nano-rails    Ruby on Rails installed in a Nano Server b...   1                    [OK]
microsoft/nano-redis    Redis installed in a Nano Server based con...   1                    [OK]
microsoft/nano-ruby     Ruby installed in a Nano Server based cont...   1                    [OK]
```

Pour télécharger une image à partir du hub Docker, utilisez la commande `docker pull`.

```powershell
C:\> docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

L’image est désormais visible quand vous exécutez la commande `docker images`.

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```






<!--HONumber=Mar16_HO3-->



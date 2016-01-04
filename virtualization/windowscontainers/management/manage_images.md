# Images de conteneur

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.**

Les images de conteneur sont utilisées pour déployer des conteneurs. Ces images peuvent inclure un système d’exploitation, des applications et toutes les dépendances d’application. Par exemple, vous pouvez développer une image de conteneur qui a été préconfigurée avec Nano Server, IIS et une application s’exécutant dans IIS. Cette image de conteneur peut alors être stockée dans un Registre de conteneur pour être utilisée plus tard, déployée sur un hôte de conteneur Windows (localement, dans le cloud, ou même dans un service de conteneur) et utilisée également comme base pour une nouvelle image de conteneur.

Il existe deux types d’images de conteneur :

- Images de système d’exploitation de base : elles sont fournies par Microsoft et incluent les composants principaux du système d’exploitation.
- Images de conteneurs : image de conteneur qui a été créée à partir d’une image de système d’exploitation de base.

## PowerShell

### Répertorier des images

Pour obtenir la liste d’images sur l’hôte de conteneur, exécutez la commande `get-containerImage`. Le type d’image de conteneur est déterminé avec la propriété `IsOSImage`.

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### Installation des images de système d’exploitation de base

Les images de système d’exploitation de conteneur sont accessibles et installées à l’aide du module PowerShell ContainerProvider. Vous devez installer ce module avant de l’utiliser. Les commandes suivantes peuvent être utilisées pour installer le module.

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

Pour télécharger et installer l’image du système d’exploitation de base Nano Server, exécutez la commande suivante.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

Cette commande télécharge et installe l’image du système d’exploitation de base Windows Server Core.

> **Problème :** les applets de commande Save-ContainerImage et Install-ContainerImage ne fonctionnent pas avec une image de conteneur WindowsServerCore dans une session de communication à distance PowerShell. **Solution de contournement :** ouvrez une session sur l’ordinateur à l’aide du Bureau à distance et utilisez directement l’applet de commande Save-ContainerImage.

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
Pour plus d’informations sur la gestion des images de conteneur, voir [Images de conteneur Windows](../management/manage_images.md).

### Création d’une image

```powershell
PS C:\> New-ContainerImage -Container $container -Publisher Demo -Name DemoImage -Version 1.0
```

### Suppression d’une image

Les images de conteneur ne peuvent pas être supprimées si un conteneur, même dans un état arrêté, a une dépendance sur l’image.

Supprimez une image unique avec PowerShell.

```powershell
PS C:\> Get-ContainerImage -Name newimage | Remove-ContainerImage -Force
```

### Dépendance d’image

Quand une image est créée, elle devient dépendante de l’image à partir de laquelle elle a été créée. Vous pouvez afficher cette dépendance à l’aide de la commande `get-containerimage`. Si une image parente n’est pas répertoriée, l’image est une image de système d’exploitation de base.

```powershell
PS C:\> Get-ContainerImage | select Name, ParentImage

Name              ParentImage
----              -----------
NanoServerIIS     ContainerImage (Name = 'NanoServer') [Publisher = 'CN=Microsoft', Version = '10.0.10586.0']
NanoServer
WindowsServerCore
```

## Docker

### Répertorier des images

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### Création d’une image

```powershell
C:\> docker commit 475059caef8f windowsservercoreiis

ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### Suppression d’une image

Les images de conteneur ne peuvent pas être supprimées si un conteneur, même dans un état arrêté, a une dépendance sur l’image.

Quand vous supprimez une image avec docker, les images peuvent être référencées par ID ou nom d’image.

```powershell
C:\> docker rmi windowsservercoreiis

Untagged: windowsservercoreiis:latest
Deleted: ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### Hub Docker

Le Registre du hub Docker contient des images prédéfinies qui peuvent être téléchargées sur un hôte de conteneur. Une fois ces images téléchargées, elles peuvent servir comme base pour les applications de conteneur Windows.

Pour afficher la liste des images disponibles à partir du hub Docker, utilisez la commande `docker search`. Remarque : l’image du système d’exploitation de base Windows Server Core doit être installée avant d’extraire des images dépendantes sur Windows Server Core à partir du hub Docker.

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

### Dépendance d’image

Pour visualiser les dépendances d’image avec Docker, utilisez la commande `docker history`.

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```




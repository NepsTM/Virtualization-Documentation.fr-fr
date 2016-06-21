---
author: enderb-ms
redirect_url: https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples
---


# Création d’une image de conteneur .NET 3.5 Server Core

Ce guide décrit en détail la création d’un conteneur Windows Server Core incluant .NET Framework 3.5. Avant de commencer cet exercice, vous devez disposer du fichier .iso de Windows Server 2016, ou accéder au support Windows Server 2016.

## Préparation du support

Avant de créer une image de conteneur compatible avec .NET 3.5, le package .NET 3.5 doit être préparé pour une utilisation dans un conteneur. Pour cet exemple, le fichier `Microsoft-windows-netfx3-ondemand-package.cab` est copié du support Windows Server 2016 sur l’hôte de conteneur.

Créez sur l’hôte de conteneur un répertoire nommé `dotnet3.5\source`.

```powershell
New-Item -ItemType Directory c:\dotnet3.5\source
```

Copiez le fichier `Microsoft-windows-netfx3-ondemand-package.cab` dans ce répertoire. Ce fichier se trouve dans le dossier sources\sxs du support Windows Server 2016.

```powershell
$file = "d:\sources\sxs\Microsoft-windows-netfx3-ondemand-package.cab"
Copy-Item -Path $file -Destination c:\dotnet3.5\source
``` 
    
Si l’hôte de votre conteneur s’exécute sur une machine virtuelle Hyper-V et a été déployé avec un script de démarrage rapide, vous pouvez également exécuter l’opération suivante. Notez que cette opération est exécutée sur l’hôte Hyper-V et non sur l’hôte du conteneur. 

```powershell
$vm = "<Container Host VM Name>"
$iso = "$((Get-VMHost).VirtualHardDiskPath)".TrimEnd("\") + "\WindowsServerTP4.iso"
Mount-DiskImage -ImagePath $iso
$ISOImage = Get-DiskImage -ImagePath $iso | Get-Volume
$ISODrive = "$([string]$iSOImage.DriveLetter):"
Get-VM -Name $vm | Enable-VMIntegrationService -Name "Guest Service Interface"
Copy-VMFile -Name $vm -SourcePath "$iSODrive\sources\sxs\microsoft-windows-netfx3-ondemand-package.cab" -DestinationPath "c:\dotnet3.5\source\microsoft-windows-netfx3-ondemand-package.cab" -FileSource Host -CreateFullPath
Dismount-DiskImage -ImagePath $iso
```

Vous pouvez à présent créer une image de conteneur incluant .NET Framework 3.5. Vous pouvez exécuter cette opération avec PowerShell ou Docker. Les deux méthodes sont illustrées ci-dessous.

## Création d’image - PowerShell

Lors de la création d’une image à l’aide de PowerShell, un conteneur est créé, modifié avec tous les changements souhaités, puis capturé dans une nouvelle image.

Créez un conteneur à partir de l’image de base de Windows Server Core.

```powershell
New-Container -Name dotnet35 -ContainerImageName windowsservercore -SwitchName "Virtual Switch"
```

Créez un dossier partagé avec le nouveau conteneur. Celui-ci doit servir à rendre le fichier cab de .NET 3.5 accessible à l’intérieur du conteneur.  Notez que, lors de l’exécution de la commande suivante, le conteneur doit être arrêté.

```powershell
Add-ContainerSharedFolder -ContainerName dotnet35 -SourcePath C:\dotnet3.5\source -DestinationPath c:\sxs
```

Démarrez le conteneur, puis exécutez la commande suivante pour installer .NET 3.5.

```powershell
Start-Container dotnet35
Invoke-Command -ContainerName dotnet35 -ScriptBlock {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs} -RunAsAdministrator
```

Une fois l’installation terminée, arrêtez le conteneur.

```powershell
Stop-Container dotnet35
```

Pour créer une image de ce conteneur, exécutez la commande suivante sur l’hôte du conteneur.

```powershell
New-ContainerImage -ContainerName dotnet35 -Name dotnet35 -Publisher Demo -Version 1.0
```

Exécutez `Get-ContainerImages` pour afficher la nouvelle image. Vous pouvez à présent utiliser cette image pour exécuter un conteneur avec .NET Framework 3.5 préinstallé.

```powershell
Get-ContainerImages
```

## Création d’image - Docker
 
Lors de la création d’une image à l’aide de Docker, un fichier Dockerfile crée des instructions relatives à la création de la nouvelle image. Ce fichier Dockerfile est ensuite exécuté, ce qui produit une nouvelle image du conteneur. Notez que les commandes suivantes sont exécutées sur la machine virtuelle hôte du conteneur.

Créez un fichier Dockerfile et ouvrez-le dans le Bloc-notes.

```powershell
New-Item C:\dotnet3.5\dockerfile -Force
Notepad C:\dotnet3.5\dockerfile
```

Copiez ce texte dans le fichier Dockerfile, puis enregistrez-le.

```powershell
FROM windowsservercore
ADD source /sxs
RUN powershell -Command "& {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs}"
```

Exécutez `docker build` qui utilise le fichier Dockerfile pour créer l’image de conteneur.

```powershell
Docker build -t dotnet35 C:\dotnet3.5\
```

Exécutez `docker images` pour afficher la nouvelle image. Vous pouvez à présent utiliser cette image pour exécuter un conteneur avec .NET Framework 3.5 préinstallé.

```powershell
docker images
```


<!--HONumber=May16_HO3-->



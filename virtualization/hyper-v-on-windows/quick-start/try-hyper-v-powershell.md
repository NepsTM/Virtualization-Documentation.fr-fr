---
title: "Utilisation d’Hyper-V et de Windows PowerShell"
description: "Utilisation d’Hyper-V et de Windows PowerShell"
keywords: "Windows 10, Hyper-V"
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6d1ae036-0841-4ba5-b7e0-733aad31e9a7
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 520ef410eddad61c40d31655e6bf7ede7846d4b2

---

# Utilisation d’Hyper-V et de Windows PowerShell

Maintenant que vous connaissez les principes de base liés au déploiement d’Hyper-V, à la création de machines virtuelles et à la gestion de celles-ci, voyons comment automatiser une partie de ces opérations avec PowerShell.

### Retourner une liste de commandes de Hyper-V

1.  Cliquez sur le bouton Démarrer de Windows, puis tapez **PowerShell**.
2.  Exécutez la commande suivante pour afficher une liste pouvant faire l’objet d’une recherche des commandes PowerShell disponibles dans le module Hyper-V PowerShell.

 ```powershell
Get-Command -Module hyper-v | Out-GridView
```
  Vous obtenez une liste qui ressemble à ceci :

  ![](media\command_grid.png)

3. Pour en savoir plus sur une commande PowerShell en particulier, utilisez `Get-Help`. Par exemple, pour obtenir des informations sur la commande Hyper-V `Get-VM`, exécutez la commande suivante.

  ```powershell
Get-Help Get-VM
```
 La sortie indique la structure de la commande, les paramètres obligatoires et facultatifs, ainsi que les alias que vous pouvez utiliser.

 ![](media\get_help.png)


### Retourner une liste de machines virtuelles

Pour obtenir la liste des machines virtuelles, utilisez la commande `Get-VM`.

1. Dans PowerShell, exécutez la commande suivante :
 
 ```powershell
Get-VM
```
 Une liste semblable à celle-ci s’affiche :

 ![](media\get_vm.png)

2. Pour obtenir uniquement la liste des machines virtuelles sous tension, ajoutez un filtre à la commande `Get-VM`. Vous pouvez ajouter un filtre à l’aide de la commande `Where-Object`. Pour plus d’informations sur le filtrage, voir la documentation [Utilisation de Where-Object](https://technet.microsoft.com/en-us/library/ee177028.aspx).   

 ```powershell
 Get-VM | where {$_.State -eq 'Running'}
 ```
3.  Pour répertorier toutes les machines virtuelles dans un état hors tension, exécutez la commande suivante. Cette commande est une copie de la commande de l’étape 2 dans laquelle le filtre « Running » est remplacé par le filtre « Off ».

 ```powershell
 Get-VM | where {$_.State -eq 'Off'}
 ```

### Démarrer et arrêter des machines virtuelles

1. Pour démarrer une machine virtuelle spécifique, exécutez la commande suivante avec le nom de la machine virtuelle :

 ```powershell
 Start-VM -Name <virtual machine name>
 ```

2. Pour démarrer toutes les machines virtuelles actuellement hors tension, obtenez la liste de ces machines et canalisez-la vers la commande `Start-VM` :

  ```powershell
 Get-VM | where {$_.State -eq 'Off'} | Start-VM
 ```
3. Pour arrêter toutes les machines virtuelles en cours d’exécution, exécutez la commande suivante :
 
  ```powershell
 Get-VM | where {$_.State -eq 'Running'} | Stop-VM
 ```

### Créer un point de contrôle de machine virtuelle

Pour créer un point de contrôle à l’aide de PowerShell, sélectionnez la machine virtuelle à l’aide de la commande `Get-VM` et canalisez-la vers la commande `Checkpoint-VM`. Pour terminer, nommez le point de contrôle à l’aide de `-SnapshotName`. La commande complète ressemble à ce qui suit :

 ```powershell
 Get-VM -Name <VM Name> | Checkpoint-VM -SnapshotName <name for snapshot>
 ```
### Créer une machine virtuelle

L’exemple suivant montre comment créer une machine virtuelle dans l’environnement d’écriture de scripts intégré de Windows PowerShell (ISE). Vous pouvez étendre cet exemple simple de manière à inclure des fonctionnalités supplémentaires de PowerShell et des déploiements de machines virtuelles plus avancés.

1. Pour ouvrir PowerShell ISE, cliquez sur Démarrer, puis tapez **PowerShell ISE**.
2. Exécutez le code suivant pour créer une machine virtuelle. Pour plus d’informations sur la commande `New-VM`, consultez la documentation de [New-VM](https://technet.microsoft.com/en-us/library/hh848537.aspx).

  ```powershell
 $VMName = "VMNAME"

 $VM = @{
     Name = $VMName 
     MemoryStartupBytes = 2147483648
     Generation = 2
     NewVHDPath = "C:\Virtual Machines\$VMName\$VMName.vhdx"
     NewVHDSizeBytes = 53687091200
     BootDevice = "VHD"
     Path = "C:\Virtual Machines\$VMName"
     SwitchName = (Get-VMSwitch).Name[0]
 }

 New-VM @VM
  ```

## Conclusion et références

Dans ce document, nous avons effectué quelques étapes simples pour explorer le module PowerShell Hyper-V et quelques scénarios d’utilisation. Pour plus d’informations sur le module PowerShell Hyper-V, voir les [informations de référence sur les applets de commande Hyper-V dans Windows PowerShell](https://technet.microsoft.com/%5Clibrary/Hh848559.aspx).  
 



<!--HONumber=Oct16_HO4-->



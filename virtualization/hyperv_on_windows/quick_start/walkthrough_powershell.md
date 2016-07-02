---
title: "Utilisation d’Hyper-V et de Windows PowerShell"
description: "Utilisation d’Hyper-V et de Windows PowerShell"
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6d1ae036-0841-4ba5-b7e0-733aad31e9a7
translationtype: Human Translation
ms.sourcegitcommit: e14ede0a2b13de08cea0a955b37a21a150fb88cf
ms.openlocfilehash: a8e567b6447aa73f14825b7054d977d2b003a726

---

# Utilisation d’Hyper-V et de Windows PowerShell

Maintenant que vous connaissez les principes de base liés au déploiement d’Hyper-V, à la création de machines virtuelles et à la gestion de celles-ci, voyons comment automatiser une partie de ces opérations avec PowerShell.

### Retourner une liste de commandes de Hyper-V

1.  Cliquez sur le bouton Démarrer de Windows, puis tapez **PowerShell**.
2.  Exécutez la commande suivante pour afficher une liste pouvant faire l’objet d’une recherche des commandes PowerShell disponibles dans le module Hyper-V PowerShell.

 ```powershell
get-command -module hyper-v | out-gridview
```
  Vous obtenez une liste qui ressemble à ceci :

  ![](media\command_grid.png)

3. Pour plus d’informations sur une commande PowerShell particulière, utilisez `get-help`. Par exemple, pour obtenir des informations sur la commande Hyper-V `get-vm`, exécutez la commande suivante.

  ```powershell
get-help get-vm
```
 La sortie indique la structure de la commande, les paramètres obligatoires et facultatifs, ainsi que les alias que vous pouvez utiliser.

 ![](media\get_help.png)


### Retourner une liste de machines virtuelles

Pour obtenir la liste des machines virtuelles, utilisez la commande `get-vm`.

1. Dans PowerShell, exécutez la commande suivante :
 
 ```powershell
get-vm
```
 Une liste semblable à celle-ci s’affiche :

 ![](media\get_vm.png)

2. Pour obtenir uniquement la liste des machines virtuelles sous tension, ajoutez un filtre à la commande `get-vm`. Vous pouvez ajouter un filtre à l’aide de la commande where-object. Pour plus d’informations sur le filtrage, voir la documentation [Utilisation de Where-Object](https://technet.microsoft.com/en-us/library/ee177028.aspx).   

 ```powershell
 get-vm | where {$_.State -eq ‘Running’}
 ```
3.  Pour répertorier toutes les machines virtuelles dans un état hors tension, exécutez la commande suivante. Cette commande est une copie de la commande de l’étape 2 dans laquelle le filtre « Running » est remplacé par le filtre « Off ».

 ```powershell
 get-vm | where {$_.State -eq ‘Off’}
 ```

### Démarrer et arrêter des machines virtuelles

1. Pour démarrer une machine virtuelle spécifique, exécutez la commande suivante avec le nom de la machine virtuelle :

 ```powershell
 Start-vm -Name <virtual machine name>
 ```

2. Pour démarrer toutes les machines virtuelles actuellement hors tension, obtenez la liste de ces machines et dirigez-la vers la commande « start-vm » :

  ```powershell
 get-vm | where {$_.State -eq ‘Off’} | start-vm
 ```
3. Pour arrêter toutes les machines virtuelles en cours d’exécution, exécutez la commande suivante :
 
  ```powershell
 get-vm | where {$_.State -eq ‘Running’} | stop-vm
 ```

### Créer un point de contrôle de machine virtuelle

Pour créer un point de contrôle à l’aide de PowerShell, sélectionnez la machine virtuelle à l’aide de la commande `get-vm` et canalisez-la vers la commande `checkpoint-vm`. Pour terminer, nommez le point de contrôle à l’aide de `-snapshotname`. La commande complète ressemble à ce qui suit :

 ```powershell
 get-vm -Name <VM Name> | checkpoint-vm -snapshotname <name for snapshot>
 ```
### Créer une machine virtuelle

L’exemple suivant montre comment créer une machine virtuelle dans l’environnement d’écriture de scripts intégré de Windows PowerShell (ISE). Vous pouvez étendre cet exemple simple de manière à inclure des fonctionnalités supplémentaires de PowerShell et des déploiements de machines virtuelles plus avancés.

1. Pour ouvrir PowerShell ISE, cliquez sur Démarrer, puis tapez **PowerShell ISE**.
2. Exécutez le code suivant pour créer une machine virtuelle. Pour plus d’informations sur la commande New-VM, voir la documentation de [New-VM](https://technet.microsoft.com/en-us/library/hh848537.aspx).

  ```powershell
 $VMName = "VMNAME"

 $VM = @{
     Name = $VMName 
     MemoryStartupBytes = 2147483648
     Generation = 2
     NewVHDPath = "C:\Virtual Machines\$VMName\$VMName.vhdx"
     NewVHDSizeBytes = 53687091200
     BootDevice = "VHD"
     Path = "C:\Virtual Machines\$VMName "
     SwitchName = (get-vmswitch).Name[0]
 }

 New-VM @VM
  ```

## Conclusion et références

Dans ce document, nous avons effectué quelques étapes simples pour explorer le module PowerShell Hyper-V et quelques scénarios d’utilisation. Pour plus d’informations sur le module PowerShell Hyper-V, voir les [informations de référence sur les applets de commande Hyper-V dans Windows PowerShell](https://technet.microsoft.com/%5Clibrary/Hh848559.aspx).  
 


<!--HONumber=Jun16_HO4-->



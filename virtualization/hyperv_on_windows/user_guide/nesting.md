---
title: Virtualisation imbriquée
description: Virtualisation imbriquée
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
---

# Virtualisation imbriquée

La virtualisation imbriquée fournit la possibilité d’exécuter des hôtes Hyper-V dans un environnement virtualisé. En d’autres termes, avec la virtualisation imbriquée, un hôte Hyper-V peut être virtualisé. Certains cas d’utilisation de la virtualisation imbriquée consistent à exécuter un laboratoire Hyper-V dans un environnement virtualisé, pour fournir des services de virtualisation à d’autres personnes sans besoin de matériel, et la technologie de conteneur Windows repose sur la virtualisation imbriquée lors de l’exécution de conteneurs Hyper-V sur un hôte de conteneur virtualisé. Ce document décrit en détail la configuration matérielle et logicielle requise, les étapes de configuration et la résolution des problèmes.

> La virtualisation imbriquée est disponible en version préliminaire et ne doit pas être utilisée en production.

## Conditions préalables

- Windows Insiders (Windows Server 2016, Nano Server ou Windows 10) qui exécutent la Build 10565 ou version ultérieure.
- Les deux hyperviseurs (parent et enfant) doivent exécuter des builds Windows identiques (10565 ou version ultérieure).
- 4 Go de RAM disponible au minimum.
- Processeur Intel avec la technologie Intel VT-x.

## Configurer la virtualisation imbriquée

Créez d’abord une machine virtuelle exécutant la même build que celle de votre hôte, **n’activez pas la machine virtuelle**. Pour plus d’informations, consultez [Créer une machine virtuelle](../quick_start/walkthrough_create_vm.md).

Une fois que la machine virtuelle a été créée, exécutez la commande suivante sur l’hyperviseur parent, ce qui permet d’activer la virtualisation imbriquée sur la machine virtuelle.

```none
Set-VMProcessor -VMName <virtual machine> -ExposeVirtualizationExtensions $true -Count 2
```

Lors de l’exécution d’un hôte Hyper-V imbriqué, la mémoire dynamique doit être désactivée sur la machine virtuelle. Cette configuration peut être effectuée sur les propriétés de la machine virtuelle ou en utilisant la commande PowerShell suivante.

```none
Set-VMMemory <virtual machine> -DynamicMemoryEnabled $false
```

Pour que les machines virtuelles imbriquées reçoivent des adresses IP, l’usurpation d’adresse MAC doit être activée. Pour cela, exécutez la commande PowerShell suivante.

```none
Get-VMNetworkAdapter -VMName <virtual machine> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

Une fois ces étapes terminées, la machine virtuelle peut être démarrée et Hyper-V installé. Pour plus d’informations sur l’installation d’Hyper-V, consultez [Installer Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install).

## du périphérique VPN

Le script suivant peut éventuellement être utilisé pour activer et configurer la virtualisation imbriquée. Les commandes suivantes téléchargent et exécutent le script.
  
```none
# download script
Invoke-WebRequest https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Enable-NestedVm.ps1 -OutFile .\Enable-NestedVm.ps1 

# run script
.\Enable-NestedVm.ps1 -VmName "DemoVM"
```

## Problèmes connus

- Les hôtes sur lesquels la fonctionnalité Device Guard est activée ne peuvent pas exposer les extensions de virtualisation aux invités.
- Les hôtes sur lesquels la sécurité basée sur la virtualisation est activée ne peuvent pas exposer les extensions de virtualisation aux invités. Vous devez d’abord désactiver la sécurité basée sur la virtualisation pour pouvoir utiliser la virtualisation imbriquée.
- La connexion de la machine virtuelle peut être perdue si vous utilisez un mot de passe vide. Modifiez le mot de passe pour résoudre le problème.
- Une fois la virtualisation imbriquée activée sur une machine virtuelle, les fonctionnalités suivantes ne sont plus compatibles avec cette machine virtuelle.  
  * Redimensionnement de la mémoire d’exécution.
  * Application d’un point de contrôle à une machine virtuelle en cours d’exécution.
  * Une machine virtuelle qui héberge d’autres machines virtuelles ne peut pas faire l’objet d’une migration dynamique.
  * L’enregistrement et la restauration ne fonctionnent pas.

## FAQ et résolution des problèmes

Ma machine virtuelle ne démarre pas, que dois-je faire ?

1. Assurez-vous que la mémoire dynamique est désactivée.
2. Exécutez [ce script PowerShell](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1) sur votre machine hôte à partir d’une invite de commandes avec élévation de privilèges.
  
## Commentaires

Indiquez tout autre problème dans l’application Commentaires sur Windows, les [forums sur la virtualisation](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv) ou sur [GitHub](https://github.com/Microsoft/Virtualization-Documentation).



<!--HONumber=Jun16_HO2-->



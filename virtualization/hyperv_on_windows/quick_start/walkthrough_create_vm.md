---
title: "Déployer une machine virtuelle Windows dans Hyper-V sur Windows 10"
description: "Déployer une machine virtuelle Windows dans Hyper-V sur Windows 10"
keywords: "Windows 10, Hyper-V"
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 66723f33-b12c-49d1-82cf-71ba9d6087e9
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 2b644f56829d6f92822875e77ba53e5b68d2f1ef

---

# Déployer une machine virtuelle Windows dans Hyper-V sur Windows 10

Pour créer une machine virtuelle et déployer sur celle-ci un système d’exploitation, plusieurs options s’offrent à vous : vous pouvez recourir aux services de déploiement Windows, attacher un disque dur virtuel préparé ou encore procéder manuellement à l’aide du support d’installation. Cet article vous explique pas à pas comment créer une machine virtuelle et déployer sur celle-ci un système d’exploitation figurant sur un support d’installation.

Avant de commencer cet exercice, vous avez besoin du fichier .iso correspondant au système d’exploitation à déployer. Si nécessaire, téléchargez une copie d’évaluation de Windows 8.1 ou de Windows 10 à partir du [Centre d’évaluation TechNet](http://www.microsoft.com/en-us/evalcenter/).

## Créer une machine virtuelle avec le Gestionnaire Hyper-V
La procédure manuelle suivante vous montre comment créer une machine virtuelle et comment déployer un système d’exploitation sur celle-ci.

1. Dans le Gestionnaire Hyper-V, cliquez sur **Action** > **Nouveau** > **Ordinateur virtuel** pour ouvrir l’Assistant Nouvel ordinateur virtuel.

2. Passez en revue le contenu de la section « Avant de commencer », puis cliquez sur **Suivant**. 

3. Attribuez un nom à la machine virtuelle.
  > **Remarque :** Il s’agit du nom utilisé par Hyper-V pour la machine virtuelle, et non pas du nom d’ordinateur attribué au système d’exploitation invité qui sera déployé dans la machine virtuelle.

4. Choisissez un emplacement dans lequel stocker les fichiers de la machine virtuelle, par exemple **c:\virtualmachine**. Vous pouvez aussi accepter l’emplacement par défaut. Quand vous avez terminé, cliquez sur **Suivant**.
    
  ![](media/new_vm_upd.png)

5. Sélectionnez la génération de la machine, puis cliquez sur **Suivant**.  

  Introduites avec Windows Server 2012 R2, les machines virtuelles de génération 2 fournissent un modèle de matériel virtuel simplifié et des fonctionnalités supplémentaires. Vous pouvez uniquement installer un système d’exploitation 64 bits sur une machine virtuelle de génération 2. Pour plus d’informations sur les machines virtuelles de génération 2, voir [Vue d’ensemble des machines virtuelles de génération 2](https://technet.microsoft.com/en-us/library/dn282285.aspx).
  
  > Si la nouvelle machine virtuelle est configurée comme étant de génération 2 et exécute une distribution Linux, le démarrage sécurisé doit être désactivé. Pour plus d’informations sur le démarrage sécurisé, voir [Démarrage sécurisé](https://technet.microsoft.com/en-us/library/dn486875.aspx).

6. Sélectionnez **2048** Mo comme valeur de **Mémoire de démarrage** et laissez l’option **Utiliser la mémoire dynamique** sélectionnée. Cliquez sur le bouton **Suivant**.  

  La mémoire est partagée entre un hôte Hyper-V et la machine virtuelle en cours d’exécution sur l’hôte. Le nombre de machines virtuelles pouvant s’exécuter sur un seul hôte dépend en partie de la mémoire disponible. Vous pouvez également configurer une machine virtuelle de manière à ce qu’elle utilise la mémoire dynamique. Une fois activée, la mémoire dynamique récupère la mémoire inutilisée de la machine virtuelle en cours d’exécution. Cela permet d’exécuter davantage de machines virtuelles sur l’hôte. Pour plus d’informations sur la mémoire dynamique, voir [Présentation de la mémoire dynamique Hyper-V](https://technet.microsoft.com/en-us/library/hh831766.aspx).

7. Dans l’Assistant de configuration de la mise en réseau, sélectionnez un commutateur virtuel pour la machine virtuelle, puis cliquez sur **Suivant**. Pour plus d’informations, voir [Créer un commutateur virtuel](walkthrough_virtual_switch.md).

8. Nommez le disque dur virtuel, sélectionnez un emplacement ou conservez la valeur par défaut, puis spécifiez une taille. Quand vous êtes prêt, cliquez sur **Suivant**.

  À l’instar d’un disque dur dans un ordinateur physique, un disque dur virtuel fournit à la machine virtuelle un espace de stockage. Vous devez disposer d’un disque dur virtuel pour installer un système d’exploitation sur la machine virtuelle.
  
  ![](media/new_vhd_upd.png)  

9. Dans l’Assistant Options d’installation, sélectionnez **Installer un système d’exploitation à partir d’un fichier image de démarrage**, puis sélectionnez le fichier .iso d’un système d’exploitation. Une fois l’opération terminée, cliquez sur **Suivant**.

  Quand vous créez une machine virtuelle, vous pouvez configurer certaines options d’installation du système d’exploitation. Les trois options disponibles sont les suivantes :

  - **Installer un système d’exploitation ultérieurement** : cette option n’apporte aucune modification supplémentaire à la machine virtuelle.

  - **Installer un système d’exploitation à partir d’un fichier image de démarrage** : cette option équivaut à insérer un CD dans le lecteur de CD-ROM physique d’un ordinateur physique. Pour configurer cette option, sélectionnez une image .iso. Cette image est montée sur le lecteur de CD-ROM virtuel de la machine virtuelle. L’ordre de démarrage de la machine virtuelle est modifié pour faire passer le lecteur de CD-ROM en première position.

  - **Installer un système d’exploitation à partir d’un serveur d’installation réseau** : cette option n’est accessible que si vous avez connecté la machine virtuelle à un commutateur réseau. Dans cette configuration, la machine virtuelle tente de démarrer à partir du réseau.
  
10. Passez en revue les détails de la machine virtuelle, puis cliquez sur **Terminer** pour terminer la création de la machine virtuelle.

## Créer une machine virtuelle avec PowerShell

1. Ouvrez PowerShell ISE en tant qu’administrateur.

2. Exécutez le script suivant.

  ```powershell
  # Set VM Name, Switch Name, and Installation Media Path.
  $VMName = 'TESTVM'
  $Switch = 'External VM Switch'
  $InstallMedia = 'C:\Users\Administrator\Desktop\en_windows_10_enterprise_x64_dvd_6851151.iso'
  
  # Create New Virtual Machine
  New-VM -Name $VMName -MemoryStartupBytes 2147483648 -Generation 2 -NewVHDPath "D:\Virtual Machines\$VMName\$VMName.vhdx" -NewVHDSizeBytes 53687091200 -Path "D:\Virtual Machines\$VMName" -SwitchName $Switch
  
  # Add DVD Drive to Virtual Machine
  Add-VMScsiController -VMName $VMName
  Add-VMDvdDrive -VMName $VMName -ControllerNumber 1 -ControllerLocation 0 -Path $InstallMedia
  
  # Mount Installation Media
  $DVDDrive = Get-VMDvdDrive -VMName $VMName
  
  # Configure Virtual Machine to Boot from DVD
  Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive
  ```
  
## Terminer le déploiement du système d’exploitation

Pour terminer la création de votre machine virtuelle, vous devez la démarrer et suivre les étapes d’installation du système d’exploitation.

1. Dans le Gestionnaire Hyper-V, double-cliquez sur la machine virtuelle. Cette opération lance l’outil VMConnect.

2. Dans VMConnect, cliquez sur le bouton vert Démarrer. Cela revient à appuyer sur le bouton Marche/Arrêt d’un ordinateur physique. Vous pouvez être invité à appuyer sur une touche quelconque pour démarrer à partir d’un CD ou d’un DVD. Dans ce cas, appuyez sur une touche pour continuer.
  > **Remarque :** Vous devrez éventuellement cliquer dans la fenêtre VMConnect pour vérifier que vos séquences de touches sont envoyées à la machine virtuelle.

3. Quand la machine virtuelle démarre, suivez les étapes du programme d’installation comme sur un ordinateur physique.

  ![](media/OSDeploy_upd.png) 

> **Remarque :** Si vous n’exécutez pas une version avec licence en volume de Windows, vous devez avoir une licence distincte pour la copie de Windows s’exécutant sur une machine virtuelle. Le système d’exploitation de la machine virtuelle est indépendant du système d’exploitation de l’hôte.

## Étape suivante : Utiliser PowerShell et Hyper-V
[Hyper-V et Windows PowerShell](walkthrough_powershell.md)


<!--HONumber=Oct16_HO4-->



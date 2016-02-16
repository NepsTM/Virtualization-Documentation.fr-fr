# Déployer une machine virtuelle Windows dans Hyper-V sur Windows 10

Pour créer une machine virtuelle et déployer sur celle-ci un système d’exploitation, plusieurs options s’offrent à vous : vous pouvez recourir aux services de déploiement Windows, attacher un disque dur virtuel préparé ou encore procéder manuellement à l’aide du support d’installation. Cet article vous explique pas à pas comment créer une machine virtuelle et déployer sur celle-ci un système d’exploitation figurant sur un support d’installation.

Avant de commencer cet exercice, vous avez besoin du fichier .iso correspondant au système d’exploitation à déployer. Si nécessaire, téléchargez une copie d’évaluation de Windows 8.1 ou de Windows 10 à partir du [Centre d’évaluation TechNet](http://www.microsoft.com/en-us/evalcenter/).

## Créer une machine virtuelle avec le Gestionnaire Hyper-V

La procédure manuelle suivante vous montre comment créer une machine virtuelle et comment déployer un système d’exploitation sur celle-ci.

1. Dans le Gestionnaire Hyper-V, cliquez sur **Action** > **Nouveau** > **Machine virtuelle**.

2. Passez en revue le contenu de la section « Avant de commencer », puis cliquez sur **Suivant**.

3. Nommez la machine virtuelle. Notez qu’il s’agit du nom de la machine virtuelle et non pas du nom de l’ordinateur attribué au système une fois le système d’exploitation déployé.

4. Choisissez un emplacement dans lequel stocker les fichiers de la machine virtuelle, par exemple : **c:\virtualmachine**. Vous pouvez aussi accepter l’emplacement par défaut. Quand vous avez terminé, cliquez sur **Suivant**.

  ![](media/new_vm_upd.png)

5. Sélectionnez la génération de la machine, puis cliquez sur **Suivant**.

  Introduites avec Windows Server 2012 R2, les machines virtuelles de génération 2 fournissent un modèle de matériel virtuel simplifié et des fonctionnalités supplémentaires. Vous pouvez uniquement installer un système d’exploitation 64 bits sur une machine virtuelle de génération 2. Pour plus d’informations sur les machines virtuelles de génération 2, voir [Vue d’ensemble des machines virtuelles de génération 2](https://technet.microsoft.com/en-us/library/dn282285.aspx).

> Si la nouvelle machine virtuelle est configurée en tant que génération 2 et doit être exécutée sur Linux, le démarrage sécurisé doit être désactivé. Pour plus d’informations sur le démarrage sécurisé, voir [Démarrage sécurisé](https://technet.microsoft.com/en-us/library/dn486875.aspx).

6. Sélectionnez **2048** comme valeur de **Mémoire de démarrage** et laissez l’option **Utiliser la mémoire dynamique** sélectionnée. Cliquez sur le bouton **Suivant**.

  La mémoire est partagée entre un hôte Hyper-V et la machine virtuelle en cours d’exécution sur l’hôte. Le nombre de machines virtuelles pouvant s’exécuter sur un seul hôte dépend en partie de la mémoire disponible. Vous pouvez également configurer une machine virtuelle de manière à ce qu’elle utilise la mémoire dynamique. Une fois activée, la mémoire dynamique récupère la mémoire inutilisée de la machine virtuelle en cours d’exécution. Cela permet d’exécuter davantage de machines virtuelles sur l’hôte. Pour plus d’informations sur la mémoire dynamique, voir [Présentation de la mémoire dynamique Hyper-V](https://technet.microsoft.com/en-us/library/hh831766.aspx).

7. Dans l’Assistant de configuration réseau, sélectionnez un commutateur virtuel pour la machine virtuelle, puis cliquez sur **Suivant**. Pour plus d’informations, voir [Créer un commutateur virtuel](walkthrough_virtual_switch.md).

8. Nommez le disque dur virtuel, sélectionnez un emplacement ou conservez la valeur par défaut, puis spécifiez une taille. Quand vous êtes prêt, cliquez sur **Suivant**.

  À l’instar d’un disque dur dans un ordinateur physique, un disque dur virtuel fournit à la machine virtuelle un espace de stockage. Vous devez disposer d’un disque dur virtuel pour installer un système d’exploitation sur la machine virtuelle.

  ![](media/new_vhd_upd.png)

9. Dans l’Assistant Options d’installation, sélectionnez **Installer un système d’exploitation à partir d’un fichier image de démarrage**, puis sélectionnez le fichier .iso d’un système d’exploitation. Une fois l’opération terminée, cliquez sur **Suivant**.

  Quand vous créez une machine virtuelle, vous pouvez configurer certaines options d’installation du système d’exploitation. Les trois options disponibles sont les suivantes :

  - **Installer un système d’exploitation ultérieurement** : cette option n’apporte aucune modification supplémentaire à la machine virtuelle.

  - **Installer un système d’exploitation à partir d’un fichier image de démarrage** : cette option équivaut à insérer un CD dans le lecteur de CD-ROM d’un ordinateur physique. Pour configurer cette option, sélectionnez une image .iso. Cette image est montée sur le lecteur de CD-ROM virtuel de la machine virtuelle. L’ordre de démarrage de la machine virtuelle est modifié pour faire passer le lecteur de CD-ROM en première position.

  - **Installer un système d’exploitation à partir d’un serveur d’installation réseau** : cette option n’est accessible que si vous avez connecté la machine virtuelle à un commutateur réseau. Dans cette configuration, la machine virtuelle tente de démarrer à partir du réseau.

10. Passez en revue les détails de la machine virtuelle, puis cliquez sur **Terminer** pour terminer la création de la machine virtuelle.

## Créer une machine virtuelle avec PowerShell

1. Ouvrez PowerShell ISE en tant qu’administrateur.

2. Exécutez le script suivant. Ce script

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

3. Quand la machine virtuelle démarre, suivez les étapes du programme d’installation comme sur un ordinateur physique.

  ![](media/OSDeploy_upd.png)

> **Remarque :** si vous n’exécutez pas une version avec licence en volume de Windows, vous devez posséder une licence distincte pour la copie de Windows s’exécutant sur une machine virtuelle. Le système d’exploitation de la machine virtuelle est indépendant du système d’exploitation de l’hôte.

## Étape suivante : utiliser PowerShell et Hyper-V

[Hyper-V et Windows PowerShell](walkthrough_powershell.md)



<!--HONumber=Feb16_HO2-->

---
redirect_url: ../windows_welcome
---

## Ajout et suppression à chaud de mémoire et de cartes réseau

Vous pouvez à présent ajouter ou supprimer une carte réseau quand la machine virtuelle est en cours d’exécution, ce qui évite les temps morts. Cela s’applique aux machines virtuelles de génération 2 qui exécutent des systèmes d’exploitation Windows et Linux. 

Vous pouvez également ajuster la quantité de mémoire allouée à une machine virtuelle en cours d’exécution, et ce même si vous n’avez pas activé la mémoire dynamique. Cela s’applique aux machines virtuelles de génération 1 et de génération 2.

## Points de contrôle de production

Les points de contrôle de production permettent de créer facilement des images « dans le temps » d’une machine virtuelle. Vous pouvez par la suite restaurer ces images, celles-ci étant entièrement prises en charge par toutes les charges de travail de production. Notez que la création du point de contrôle repose sur la technologie de sauvegarde de l’invité, et non sur la technologie de l’état de mise en mémoire. Pour les points de contrôle de production, le service d’instantanés de volume (VSS) est utilisé dans les machines virtuelles Windows. Les machines virtuelles Linux vident les mémoires tampons de leur système de fichiers pour créer un point de contrôle cohérent avec le système de fichiers. Si vous souhaitez créer des points de contrôle à l’aide de la technologie de l’état de mise en mémoire, vous pouvez utiliser des points de contrôle standard pour votre machine virtuelle. 


> **Important :** Pour les nouvelles machines virtuelles, la procédure par défaut consiste à créer des points de contrôle de production avec des points de contrôle standard comme secours. 
 

## Améliorations apportées au Gestionnaire Hyper-V

- **Prise en charge d’autres informations d’identification** : vous pouvez désormais utiliser un jeu d’informations d’identification différent dans le Gestionnaire Hyper-V quand vous vous connectez à un autre hôte distant Windows 10 Technical Preview. Vous pouvez également enregistrer ces informations d’identification pour faciliter les connexions ultérieures. 

- **Gestion de bas niveau** : vous pouvez désormais utiliser le Gestionnaire Hyper-V pour gérer plus de versions d’Hyper-V. Grâce au Gestionnaire Hyper-V dans Windows 10 Technical Preview, vous pouvez gérer des ordinateurs exécutant Hyper-V sur Windows Server 2012, Windows 8, Windows Server 2012 R2 et Windows 8.1.

- **Protocole de gestion mis à jour** : le Gestionnaire Hyper-V a été mis à jour pour communiquer avec les hôtes Hyper-V distants à l’aide du protocole WS-MAN, celui-ci autorisant l’authentification CredSSP, Kerberos ou NTLM. Quand vous utilisez CredSSP pour vous connecter à un hôte Hyper-V distant, vous pouvez effectuer une migration dynamique sans devoir tout d’abord activer la délégation contrainte dans Active Directory. L’infrastructure basée sur WS-MAN simplifie également la configuration à définir pour gérer un hôte à distance. WS-MAN se connecte par le biais du port 80 qui est ouvert par défaut.


## Compatibilité de la veille connectée 

Quand Hyper-V est activé sur un ordinateur qui utilise le mode d’alimentation AOAC (Always On/Always Connected), l’état de veille connectée n’est pas disponible.

Dans Windows 8 et 8.1, Hyper-V empêche les ordinateurs qui utilisent le modèle d’alimentation AOAC, également appelé InstantON, d’entrer en veille. Pour obtenir une description complète, voir cet [article de la Base de connaissances](
https://support.microsoft.com/en-us/kb/2973536).


## Démarrage sécurisé de Linux 

Vous pouvez désormais démarrer davantage de systèmes d’exploitation Linux, exécutés sur des machines virtuelles de génération 2, avec l’option de démarrage sécurisé activée.  Ubuntu 14.04 et versions ultérieures, ainsi que SUSE Linux Enterprise Server 12, prennent en charge le démarrage sécurisé sur des hôtes qui exécutent la version Technical Preview. Avant de démarrer la machine virtuelle pour la première fois, vous devez indiquer qu’elle doit utiliser l’autorité de certification Microsoft UEFI.  À partir d’une invite Windows PowerShell avec élévation de privilèges, tapez ce qui suit :

    Set-VMFirmware [-VMName] <VMName> [-SecureBootTemplate] <MicrosoftUEFICertificateAuthority>

Pour plus d’informations sur l’exécution de machines virtuelles Linux sur Hyper-V, voir [Machines virtuelles Linux et FreeBSD sur Hyper-V](http://technet.microsoft.com/library/dn531030.aspx).
 
 
## Version de configuration de la machine virtuelle

Quand vous déplacez ou importez une machine virtuelle vers un hôte exécutant Hyper-V sur Windows 10 à partir d’un hôte exécutant Windows 8.1, le fichier de configuration de la machine virtuelle n’est pas automatiquement mis à niveau. Vous pouvez ainsi redéplacer la machine virtuelle vers un hôte exécutant Windows 8.1. Pour pouvoir utiliser les nouvelles fonctionnalités Hyper-V avec votre machine virtuelle, vous devez tout d’abord mettre manuellement à jour la version de configuration de la machine virtuelle. 

Celle-ci correspond aux versions d’Hyper-V compatibles avec la configuration, l’état de mise en mémoire et les fichiers de capture instantanée de la machine virtuelle. Les machines virtuelles avec la version de configuration 5 sont compatibles avec Windows 8.1 et peuvent s’exécuter sur Windows 8.1 et Windows 10. Les machines virtuelles avec la version de configuration 6 sont compatibles avec Windows 10, mais ne peuvent pas s’exécuter sur Windows 8.1.

### Vérifier la version de configuration

À partir d’une invite de commandes avec élévation de privilèges, exécutez la commande suivante :

``` PowerShell
Get-VM * | Format-Table Name, Version
```

### Mettre à niveau la version de configuration 

À partir d’une invite Windows PowerShell avec élévation de privilèges, exécutez l’une des commandes suivantes :

``` 
Update-VmConfigurationVersion <VMName>
```

Ou

``` 
Update-VmConfigurationVersion <VMObject>
```

> **Important :**
>
- Après avoir mis à niveau la version de configuration d’une machine virtuelle, vous ne pouvez pas déplacer la machine virtuelle vers un hôte exécutant Windows 8.1.
- Vous ne pouvez pas rétrograder la version de configuration d’une machine virtuelle de la version 6 à la version 5.
- Vous devez éteindre la machine virtuelle pour mettre à niveau sa configuration.
- Au terme de la mise à niveau, la machine virtuelle utilise le nouveau format du fichier de configuration. Pour plus d’informations, voir [Format du fichier de configuration](#configuration-file-format).


## <a name="configuration-file-format"></a>Format du fichier de configuration

Les machines virtuelles disposent d’un nouveau format de fichier de configuration. Celui-ci est conçu pour accroître les performances de lecture et d’écriture des données de configuration des machines virtuelles. Il vise également à réduire le risque de corruption des données en cas de défaillance du stockage. Les nouveaux fichiers de configuration utilisent l’extension .VMCX pour les données de configuration des machines virtuelles et l’extension .VMRS pour les données d’état de l’exécution. 

> **Important :** Le fichier .VMCX a un format binaire. Vous ne pouvez pas modifier directement un fichier .VMCX ou .VMRS.

<!--HONumber=Jun16_HO2-->



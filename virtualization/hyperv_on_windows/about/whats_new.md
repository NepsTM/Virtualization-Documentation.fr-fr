# Nouveautés d’Hyper-V sur Windows 10

Cette rubrique décrit les nouvelles fonctionnalités d’Hyper-V sur Windows 10® et les modifications apportées.

## Windows PowerShell Direct

Vous disposez à présent d’un moyen simple et fiable pour exécuter des commandes Windows PowerShell dans une machine virtuelle à partir du système d’exploitation hôte. Aucune configuration spéciale n’est requise, même au niveau du réseau ou du pare-feu. 
Windows PowerShell Direct fonctionne indépendamment de la configuration de la gestion à distance. Pour vous en servir, vous devez exécuter Windows 10 ou Windows Server Technical Preview sur l’hôte et sur la machine virtuelle (système d’exploitation invité).

Pour créer une session PowerShell Direct, utilisez l’une des commandes suivantes :

``` PowerShell
Enter-PSSession -VMName VMName
Invoke-Command -VMName VMName -ScriptBlock { commands }
```

Aujourd’hui, les administrateurs Hyper-V font appel à deux catégories d’outils pour se connecter à une machine virtuelle sur leur hôte Hyper-V :
- Outils de gestion à distance tels que PowerShell ou Bureau à distance
- Connection à une machine virtuelle Hyper-V (VMConnect)

Ces deux technologies fonctionnent bien, mais chacune implique des compromis à mesure que votre déploiement Hyper-V se développe. VMConnect est fiable, mais son automatisation peut être délicate. Remote PowerShell est un outil puissant, mais son installation et sa maintenance peuvent se révéler difficiles.

Windows PowerShell Direct associe la puissance d’un environnement d’automatisation et de génération de script à la simplicité de VMConnect. Étant donné que Windows PowerShell Direct s’exécute entre l’hôte et la machine virtuelle, il est inutile d’établir une connexion réseau ou d’activer la gestion à distance. Par contre, vous avez besoin des informations d’identification de l’invité pour vous connecter à la machine virtuelle.

### Spécifications

- Vous devez être connecté à un hôte Windows 10 ou Windows Server Technical Preview avec des machines virtuelles qui exécutent Windows 10 ou Windows Server Technical Preview en tant qu’invités.
- Vous devez être connecté à l’hôte avec les informations d’identification de l’administrateur Hyper-V.
- Vous avez besoin des informations d’identification de l’utilisateur pour la machine virtuelle.
- La machine virtuelle à laquelle vous souhaitez vous connecter doit être démarrée et en cours d’exécution.


## Ajout et suppression à chaud de mémoire et de cartes réseau

Vous pouvez à présent ajouter ou supprimer une carte réseau quand la machine virtuelle est en cours d’exécution, évitant ainsi les temps morts. Cela s’applique aux machines virtuelles de génération 2 qui exécutent des systèmes d’exploitation Windows et Linux.

Vous pouvez également ajuster la quantité de mémoire allouée à une machine virtuelle en cours d’exécution, et ce même si vous n’avez pas activé la mémoire dynamique. Cela s’applique aux machines virtuelles de générations 1 et 2.

## Points de contrôle de production

Les points de contrôle de production permettent de créer facilement des images « dans le temps » d’une machine virtuelle. Vous pouvez par la suite restaurer ces images, celles-ci étant entièrement prises en charge par toutes les charges de travail de production. Notez que la création du point de contrôle repose sur la technologie de sauvegarde de l’invité, et non sur la technologie de l’état de mise en mémoire. Pour les points de contrôle de production, le service d’instantanés de volume (VSS) est utilisé dans les machines virtuelles Windows. Les machines virtuelles Linux vident les mémoires tampons de leur système de fichiers pour créer un point de contrôle cohérent avec le système de fichiers. Si vous souhaitez créer des points de contrôle à l’aide de la technologie de l’état de mise en mémoire, vous pouvez utiliser des points de contrôle standard pour votre machine virtuelle.


> **Important :** pour les nouvelles machines virtuelles, la procédure par défaut consiste à créer des points de contrôle de production avec des points de contrôle standard comme secours.


## Améliorations apportées au Gestionnaire Hyper-V

- **Prise en charge d’autres informations d’identification** : vous pouvez désormais utiliser un jeu différent d’informations d’identification dans le Gestionnaire Hyper-V quand vous vous connectez à un autre hôte distant Windows 10 Technical Preview. Vous pouvez également enregistrer ces informations d’identification pour faciliter les connexions ultérieures.

- **Gestion de bas niveau** : vous pouvez désormais utiliser le Gestionnaire Hyper-V pour gérer plusieurs versions d’Hyper-V. Grâce au Gestionnaire Hyper-V dans Windows 10 Technical Preview, vous pouvez gérer des ordinateurs exécutant Hyper-V sur Windows Server 2012, Windows 8, Windows Server 2012 R2 et Windows 8.1.

- **Protocole de gestion mis à jour** : le Gestionnaire Hyper-V a été mis à jour pour communiquer avec les hôtes Hyper-V distants à l’aide du protocole WS-MAN, celui-ci autorisant l’authentification CredSSP, Kerberos ou NTLM. Quand vous utilisez CredSSP pour vous connecter à un hôte Hyper-V distant, vous pouvez effectuer une migration dynamique sans devoir tout d’abord activer la délégation contrainte dans Active Directory. L’infrastructure basée sur WS-MAN simplifie également la configuration à définir pour gérer un hôte à distance. WS-MAN se connecte par le biais du port 80 qui est ouvert par défaut.


## La veille connectée fonctionne

Quand Hyper-V est activé sur un ordinateur qui utilise le mode d’alimentation AOAC (Always On/Always Connected), l’état de veille connectée n’est pas disponible.

Dans Windows 8 et 8.1, Hyper-V empêche les ordinateurs qui utilisent le modèle d’alimentation AOAC, également appelé InstantON, d’entrer en veille. Pour obtenir une description complète, voir cet article dans la Base de connaissances Microsoft (
https://support.microsoft.com/en-us/kb/2973536).


## Démarrage sécurisé de Linux

Vous pouvez désormais démarrer davantage de systèmes d’exploitation Linux, exécutés sur des machines virtuelles de génération 2, avec l’option de démarrage sécurisé activée. Ubuntu 14.04 et versions ultérieures, ainsi que SUSE Linux Enterprise Server 12, prennent en charge le démarrage sécurisé sur des hôtes qui exécutent la version Technical Preview. Avant de démarrer la machine virtuelle pour la première fois, vous devez indiquer qu’elle doit utiliser l’autorité de certification Microsoft UEFI. À partir d’une invite Windows PowerShell avec élévation de privilèges, tapez ce qui suit :

    Set-VMFirmware vmname -SecureBootTemplate MicrosoftUEFICertificateAuthority

Pour plus d’informations sur l’exécution de machines virtuelles Linux sur Hyper-V, voir [Machines virtuelles Linux et FreeBSD sur Hyper-V](http://technet.microsoft.com/library/dn531030.aspx).


## Version de configuration de la machine virtuelle

Quand vous déplacez ou importez une machine virtuelle vers un hôte exécutant Hyper-V sur Windows 10 à partir d’un hôte exécutant Windows 8.1, le fichier de configuration de la machine virtuelle n’est pas automatiquement mis à niveau. Vous pouvez ainsi redéplacer la machine virtuelle vers un hôte exécutant Windows 8.1. Pour bénéficier des nouvelles fonctionnalités de la machine virtuelle, vous devez tout d’abord mettre manuellement à jour la version de configuration de la machine virtuelle.

Celle-ci correspond à la version d’Hyper-V compatible avec la configuration, l’état de mise en mémoire et les fichiers de capture instantanée de la machine virtuelle. Les machines virtuelles avec la version de configuration 5 sont compatibles avec Windows 8.1 et peuvent s’exécuter sur Windows 8.1 et Windows 10. Les machines virtuelles avec la version de configuration 6 sont compatibles avec Windows 10, mais ne peuvent pas s’exécuter sur Windows 8.1.

### Vérifier la version de configuration

À partir d’une invite de commandes avec élévation de privilèges, exécutez la commande suivante :

``` PowerShell
Get-VM * | Format-Table Name, Version
```

### Mettre à niveau la version de configuration

À partir d’une invite de commandes Windows PowerShell avec élévation de privilèges, exécutez l’une des commandes suivantes :

``` PowerShell
Update-VmConfigurationVersion <vmname>
```

Ou

``` PowerShell
Update-VmConfigurationVersion <vmobject>
```

**Important : **
- Après avoir mis à niveau la version de configuration d’une machine virtuelle, vous ne pouvez pas déplacer la machine virtuelle vers un hôte exécutant Windows 8.1.
- Vous ne pouvez pas rétrograder la version de configuration d’une machine virtuelle de la version 6 à la version 5.
- Vous devez éteindre la machine virtuelle pour mettre à niveau sa configuration.
- Au terme de la mise à niveau, la machine virtuelle utilise le nouveau format du fichier de configuration. Pour plus d’informations, voir Nouveau format du fichier de configuration de machine virtuelle.


## Format du fichier de configuration

Les machines virtuelles disposent d’un nouveau format de fichier de configuration. Celui-ci est conçu pour accroître les performances de lecture et d’écriture des données de configuration des machines virtuelles. Il vise également à réduire le risque de corruption des données en cas de défaillance du stockage. Les nouveaux fichiers de configuration utilisent l’extension .VMCX pour les données de configuration des machines virtuelles et l’extension .VMRS pour les données d’état de l’exécution.


> **Important :** le fichier .VMCX a un format binaire. Vous ne pouvez pas modifier directement un fichier .VMCX ou .VMRS.

## Services d’intégration par le biais de Windows Update

Les mises à jour des services d’intégration pour les invités Windows sont désormais distribuées par le biais de Windows Update.

Les composants d’intégration (également appelés « services d’intégration ») regroupent des pilotes synthétiques qui permettent à une machine virtuelle de communiquer avec le système d’exploitation d’un hôte. Ils contrôlent des services allant de la synchronisation à la copie des fichiers de l’invité. Au cours d’entretiens menés cette année auprès de clients, nous avons constaté que l’installation et la mise à jour des composants d’intégration{b> <b}constituent une source d’irritation majeure lors du processus de mise à niveau.


Par le passé, toutes les nouvelles versions d’Hyper-V étaient fournies avec de nouveaux composants d’intégration. Pour mettre à niveau l’hôte Hyper-V, vous deviez obligatoirement mettre à niveau les composants d’intégration dans les machines virtuelles. Les nouveaux composants d’intégration, fournis avec l’hôte Hyper-V, devaient être installés dans les machines virtuelles à l’aide de vmguest.iso. Ce processus, qui exigeait le redémarrage de la machine virtuelle, ne pouvait pas être combiné avec d’autres mises à jour Windows. Comme l’administrateur Hyper-V devait fournir le fichier vmguest.iso et que l’administrateur de la machine virtuelle devait l’installer, la mise à niveau des composants d’intégration obligeait l’administrateur Hyper-V à disposer d’informations d’identification administrateur dans les machines virtuelles, ce qui posait parfois problème.
　　


Dans Windows 10 et versions ultérieures, tous les composants d’intégration sont remis aux machines virtuelles par le biais de Windows Update, au même titre que les autres mises à jour importantes.


Des mises à jour sont disponibles dès aujourd’hui pour les machines virtuelles exécutant :
*  Windows Server 2012
*  Windows Server 2008 R2
*  Windows 8
*  Windows 7

La machine virtuelle doit être connectée à Windows Update ou à un serveur WSUS. À l’avenir, les mises à jour des composants d’intégration seront associés à un ID de catégorie. Pour le moment, ils sont répertoriés dans la Base de connaissances.

Pour plus d’informations sur la façon dont nous déterminons les conditions d’application, voir ce [billet de blog](http://blogs.technet.com/b/virtualization/archive/2014/11/24/integration-components-how-we-determine-windows-update-applicability.aspx).


Pour obtenir la procédure pas à pas de l’installation des services d’intégration, voir ce [billet de blog](http://blogs.msdn.com/b/virtual_pc_guy/archive/2014/11/12/updating-integration-components-over-windows-update.aspx).


> **Important :** le fichier image ISO vmguest.iso n’est plus obligatoire pour mettre à jour les composants d’intégration. Il n’est donc pas fourni avec Hyper-V sur Windows 10.


## Étape suivante

[Hyper-V sur Windows 10 : procédure pas à pas](..\quick_start\walkthrough.md)




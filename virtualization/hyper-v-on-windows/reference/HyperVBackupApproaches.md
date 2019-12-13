# <a name="hyper-v-backup-approaches"></a>Approches de la sauvegarde Hyper-V
Hyper-V vous permet de sauvegarder des ordinateurs virtuels, depuis le système d’exploitation hôte, sans avoir à exécuter un logiciel de sauvegarde personnalisé à l’intérieur de la machine virtuelle.  Il existe plusieurs approches que les développeurs peuvent utiliser en fonction de leurs besoins.
## <a name="hyper-v-vss-writer"></a>Enregistreur VSS Hyper-V
Hyper-V implémente un enregistreur VSS sur toutes les versions de Windows Server où Hyper-V est pris en charge.  Cet enregistreur VSS permet aux développeurs d’utiliser l’infrastructure VSS existante pour sauvegarder des machines virtuelles.  Toutefois, il est conçu pour les opérations de sauvegarde à petite échelle où tous les ordinateurs virtuels d’un serveur sont sauvegardés simultanément.

Pour mieux comprendre cette architecture, reportez-vous à cette présentation : https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Sauvegarde basée sur le WMI Hyper-V
À partir de Windows Server 2016, Hyper-V a démarré la prise en charge de la sauvegarde via l’API WMI Hyper-V.  Cette approche utilise toujours le service VSS à l’intérieur de la machine virtuelle à des fins de sauvegarde, mais n’utilise plus VSS dans le système d’exploitation hôte.  Au lieu de cela, une combinaison de points de référence et de suivi des modifications résilientes (RCT) est utilisée pour permettre aux développeurs d’accéder de manière efficace aux informations sur les ordinateurs virtuels sauvegardés.  Cette approche est plus évolutive que l’utilisation de VSS dans l’hôte, mais elle n’est disponible que sur Windows Server 2016 et versions ultérieures.

Pour mieux comprendre cette architecture, reportez-vous à cette présentation : https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

Vous trouverez également un exemple d’utilisation de ces API disponibles ici : https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Méthodes de lecture des sauvegardes à partir de la sauvegarde basée sur WMI
Lors de la création de sauvegardes de machines virtuelles à l’aide d’Hyper-V WMI, il existe trois méthodes pour lire les données réelles à partir de la sauvegarde.  Chacun présente des avantages et des inconvénients uniques.
### <a name="wmi-export"></a>Exportation WMI
Les développeurs peuvent exporter les données de sauvegarde via les interfaces WMI Hyper-V (comme indiqué dans l’exemple ci-dessus).  Hyper-V compilera les modifications apportées à un disque dur virtuel et copiera le fichier à l’emplacement demandé.  Cette méthode est facile à utiliser, fonctionne pour tous les scénarios et est accessible à distance.  Toutefois, le disque dur virtuel généré crée souvent une grande quantité de données à transférer sur le réseau.
### <a name="win32-apis"></a>API Win32
Les développeurs peuvent utiliser les API SetVirtualDiskInformation, GetVirtualDiskInformation et QueryChangesVirtualDisk sur l’ensemble d’API Win32 du disque dur virtuel comme indiqué ici : https://docs.microsoft.com/windows/desktop/api/_vhd/ noter que pour utiliser ces API, vous devez toujours utiliser le WMI Hyper-V pour créer des points de référence sur les ordinateurs virtuels associés.  Ces API Win32 permettent ensuite un accès efficace aux données de l’ordinateur virtuel sauvegardé.  Les API Win32 présentent plusieurs limitations :
* Ils sont accessibles uniquement localement
* Ne prend pas en charge la lecture de données à partir de fichiers de disque dur virtuel partagés
* Elles retournent des adresses de données relatives à la structure interne du disque dur virtuel

### <a name="remote-shared-virtual-disk-protocol"></a>Protocole de disque virtuel partagé distant
Enfin, si un développeur a besoin d’accéder efficacement aux informations de données de sauvegarde à partir d’un fichier de disque dur virtuel partagé, il devra utiliser le protocole de disque virtuel partagé distant.  Ce protocole est documenté [ici](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15).

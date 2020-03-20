# <a name="hyper-v-backup-approaches"></a>Approches de la sauvegarde Hyper-V
Hyper-V vous permet de sauvegarder des machines virtuelles, à partir du système d'exploitation hôte, sans qu'il soit nécessaire d'exécuter un logiciel de sauvegarde personnalisé sur la machine virtuelle.  Plusieurs approches sont à la disposition des développeurs en fonction de leurs besoins.
## <a name="hyper-v-vss-writer"></a>Enregistreur VSS Hyper-V
Hyper-V implémente un enregistreur VSS sur toutes les versions de Windows Server où Hyper-V est pris en charge.  Cet enregistreur VSS permet aux développeurs d'utiliser l'infrastructure VSS existante pour sauvegarder des machines virtuelles.  Il est toutefois conçu pour les opérations de sauvegarde à petite échelle où toutes les machines virtuelles d'un serveur sont sauvegardées simultanément.

Pour mieux comprendre cette architecture, reportez-vous à la présentation suivante : https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Sauvegarde basée sur WMI Hyper-V
Hyper-V a commencé à prendre en charge la sauvegarde via l'API WMI Hyper-V à partir de Windows Server 2016.  Cette approche utilise toujours VSS sur la machine virtuelle à des fins de sauvegarde, mais n'utilise plus VSS dans le système d'exploitation hôte.  À la place, une combinaison de points de référence et de suivi de modifications durables est utilisée pour permettre aux développeurs d'accéder efficacement aux informations relatives aux machines virtuelles sauvegardées.  Cette approche est plus évolutive que l'utilisation de VSS sur l'hôte, mais elle n'est disponible que sous Windows Server 2016 et versions ultérieures.

Pour mieux comprendre cette architecture, reportez-vous à la présentation suivante : https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

Vous trouverez également un exemple d'utilisation de ces API ici : https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Méthodes de lecture des sauvegardes à partir d'une sauvegarde basée sur WMI
Lors de la création de sauvegardes de machines virtuelles à l'aide de WMI Hyper-V, trois méthodes sont disponibles pour lire les données de sauvegarde.  Chacune présente des avantages et des inconvénients.
### <a name="wmi-export"></a>Exportation WMI
Les développeurs peuvent exporter les données de sauvegarde par le biais des interfaces WMI Hyper-V (comme indiqué dans l'exemple ci-dessus).  Hyper-V compilera les modifications sur un disque dur virtuel et copiera le fichier à l'emplacement demandé.  Simple d'utilisation, cette méthode fonctionne pour tous les scénarios et est accessible à distance.  Toutefois, le disque dur virtuel généré crée souvent une grande quantité de données à transférer sur le réseau.
### <a name="win32-apis"></a>API Win32
Les développeurs peuvent utiliser les API SetVirtualDiskInformation, GetVirtualDiskInformation et QueryChangesVirtualDisk sur l'ensemble d'API Win32 du disque dur virtuel, comme indiqué ici : https://docs.microsoft.com/windows/desktop/api/_vhd/ Notez que pour utiliser ces API, WMI Hyper-V doit encore être utilisé afin de créer des points de référence sur les machines virtuelles associées.  Ces API Win32 permettent ensuite un accès efficace aux données de la machine virtuelle sauvegardée.  Les API Win32 présentent différentes limitations :
* Elles ne sont accessibles que localement
* Elles ne prennent pas en charge la lecture de données à partir de fichiers de disques durs virtuels partagés
* Elles renvoient des adresses de données relatives à la structure interne du disque dur virtuel

### <a name="remote-shared-virtual-disk-protocol"></a>Protocole RSVD (Remote Shared Virtual Disk)
Enfin, si un développeur a besoin d'accéder efficacement à des informations sur les données de sauvegarde à partir d'un fichier de disque dur virtuel partagé, il devra utiliser le protocole RSVD.  Ce protocole est décrit [ici](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15).

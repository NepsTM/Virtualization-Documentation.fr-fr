# <a name="hyper-v-backup-approaches"></a>Approches de sauvegarde Hyper-V
Hyper-V vous permet de sauvegarder des machines virtuelles à partir du système d’exploitation hôte, sans avoir besoin d’exécuter un logiciel de sauvegarde personnalisé à l’intérieur de l’ordinateur virtuel.  Plusieurs approches peuvent être utilisées par les développeurs selon leurs besoins.
## <a name="hyper-v-vss-writer"></a>Scripteur VSS Hyper-V
Hyper-V implémente un writer VSS sur toutes les versions de Windows Server sur lesquelles Hyper-V est pris en charge.  Ce writer VSS permet aux développeurs d’utiliser l’infrastructure VSS existante pour sauvegarder des machines virtuelles.  En revanche, il est conçu pour des opérations de sauvegarde de petite envergure dans lesquelles toutes les machines virtuelles d’un serveur sont sauvegardées en même temps.

Pour mieux comprendre cette architecture, reportez-vous à la présentation suivante:https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Sauvegarde basée sur WMI Hyper-V
À partir de Windows Server 2016, Hyper-V a commencé à prendre en charge la sauvegarde via l’API WMI Hyper-V.  Cette approche utilise toujours le SERVICEVSS dans l’ordinateur virtuel à des fins de sauvegarde, mais n’utilise plus VSS dans le système d’exploitation hôte.  Au lieu de cela, une combinaison de points de référence et de suivi des modifications résilientes (RCT) est utilisée pour permettre aux développeurs d’accéder aux informations relatives aux machines virtuelles de façon efficace.  Cette approche est d’une plus grande évolutivité que l’utilisation de VSS dans l’hôte, mais elle n’est disponible que sur Windows Server 2016 et les versions ultérieures.

Pour mieux comprendre cette architecture, reportez-vous à la présentation suivante:https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

Il existe également un exemple d’utilisation de ces API disponibles à l’adresse suivante:https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Méthodes pour lire des sauvegardes à partir d’une sauvegarde basée sur WMI
Lors de la création de sauvegardes d’ordinateur virtuel à l’aide de la technologie WMI Hyper-V, il existe trois méthodes pour lire les données réelles à partir de la sauvegarde.  Chacun présente des avantages et des inconvénients uniques.
### <a name="wmi-export"></a>Exportation WMI
Les développeurs peuvent exporter les données de sauvegarde via les interfaces WMI Hyper-V (comme utilisé dans l’exemple ci-dessus).  Hyper-V compile les modifications dans un disque dur virtuel et copie le fichier à l’emplacement demandé.  Cette méthode est facile à utiliser et fonctionne pour tous les scénarios et est accessible à distance.  Toutefois, le disque dur virtuel généré génère souvent un grand volume de données à transférer sur le réseau.
### <a name="win32-apis"></a>API Win32
Les développeurs peuvent utiliser les API SetVirtualDiskInformation, GetVirtualDiskInformation et QueryChangesVirtualDisk sur l’API Win32 de disque dur virtuel définies comme suit https://docs.microsoft.com/windows/desktop/api/_vhd/ : Remarque: pour utiliser ces API, le WMI Hyper-V doit toujours être utilisé pour créer une référence. points sur les machines virtuelles associées.  Ces API Win32 permettent alors d’accéder efficacement aux données de l’ordinateur virtuel sauvegardé.  Les API Win32 présentent plusieurs limitations:
*   Ils ne peuvent être consultés qu’en local
*   Les fichiers ne prenant pas en charge la lecture de données provenant de fichiers de disque dur virtuel partagés
*   Ils renvoient des adresses de données relatives à la structure interne du disque dur virtuel.

### <a name="remote-shared-virtual-disk-protocol"></a>Protocole de disque virtuel partagé distant
Enfin, si un développeur a besoin d’accéder efficacement aux informations de sauvegarde à partir d’un fichier de disque dur virtuel partagé, il doit utiliser le protocole de disque virtuel partagé distant.  Ce protocole est documenté [ici](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15).

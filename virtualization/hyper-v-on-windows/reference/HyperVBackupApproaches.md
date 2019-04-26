# <a name="hyper-v-backup-approaches"></a>Approches de sauvegarde Hyper-V
Hyper-V vous permet de sauvegarde machines virtuelles, à partir du système d’exploitation hôte, sans avoir besoin d’exécuter un logiciel de sauvegarde personnalisé à l’intérieur de la machine virtuelle.  Il existe plusieurs approches qui sont disponibles pour les développeurs à utiliser en fonction de leurs besoins.
## <a name="hyper-v-vss-writer"></a>Enregistreur VSS Hyper-V
Hyper-V implémente un rédacteur VSS sur toutes les versions de Windows Server dans lequel Hyper-V est pris en charge.  Ce writer VSS permet aux développeurs d’utiliser l’infrastructure VSS existante pour les machines virtuelles sauvegarde.  Toutefois, il est conçu pour les opérations de sauvegarde à petite échelle où toutes les machines virtuelles sur un serveur sont sauvegardées simultanément.

Pour mieux comprendre cette architecture, reportez-vous à cette présentation:https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Sauvegarde de sur WMI Hyper-V
À compter de Windows Server 2016, Hyper-V démarré prise en charge de sauvegarde par le biais de l’API de WMI Hyper-V.  Cette approche utilise toujours VSS à l’intérieur de la machine virtuelle pour des raisons de sauvegarde, mais n’est plus utilise VSS dans le système d’exploitation hôte.  Au lieu de cela, une combinaison de points de référence et résiliente (RCT) de suivi des modifications est utilisée pour permettre aux développeurs d’accéder aux informations sur sauvegardées machines virtuelles de manière efficace.  Cette approche est plus évolutive que l’utilisation de VSS dans l’hôte, mais il est uniquement disponible sur Windows Server 2016 et versions ultérieures.

Pour mieux comprendre cette architecture, reportez-vous à cette présentation:https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

Il existe également un exemple sur l’utilisation de ces API disponibles ici:https://msconfiggallery.cloudapp.net/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Méthodes permettant de lire des sauvegardes à partir de la sauvegarde sur WMI
Lorsque vous créez des sauvegardes de machines virtuelles à l’aide de WMI Hyper-V, il existe trois méthodes permettant de lire des données réelles à partir de la sauvegarde.  Chacune présente des avantages et ses inconvénients.
### <a name="wmi-export"></a>Exportation WMI
Les développeurs peuvent exporter les données de sauvegarde via les interfaces WMI Hyper-V (comme dans l’exemple ci-dessus).  Hyper-V sera compiler les modifications dans un disque dur virtuel et copiez le fichier à l’emplacement demandé.  Cette méthode est facile à utiliser, fonctionne pour tous les scénarios et est accessible à distance.  Toutefois, le disque dur virtuel généré souvent crée une grande quantité de données à transférer via le réseau.
### <a name="win32-apis"></a>API Win32
Les développeurs peuvent utiliser la SetVirtualDiskInformation, GetVirtualDiskInformation et APIs QueryChangesVirtualDisk sur l’API Win32 de disque dur virtuel définie comme indiqué ici: https://docs.microsoft.com/en-us/windows/desktop/api/_vhd/ Remarque que pour utiliser ces API, WMI Hyper-V toujours doit être utilisé pour créer la référence points sur les machines virtuelles associées.  Ces API Win32 puis accordez pour un accès efficace pour les données de la machine virtuelle sauvegardée.  Les API Win32 ont plusieurs limitations:
*   Ils sont uniquement accessibles localement
*   Ne prend pas en charge les données à partir de lecture partagé les fichiers de disque dur virtuel
*   Elles retournent des adresses de données qui sont par rapport à la structure interne du disque dur virtuel

### <a name="remote-shared-virtual-disk-protocol"></a>Protocole de disque virtuel partagé distant
Enfin, si un développeur doit efficacement accéder aux informations de données de sauvegarde à partir d’un fichier de disque dur virtuel partagé – ils devrez utiliser le protocole de disque virtuel partagé à distance.  Ce protocole est documenté ici:https://msdn.microsoft.com/en-us/library/dn393384.aspx

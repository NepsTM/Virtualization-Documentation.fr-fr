# <a name="hyper-v-backup-approaches"></a>Approches de sauvegarde Hyper-V
Hyper-V vous permet de sauvegarde des ordinateurs virtuels, à partir du système d’exploitation hôte, sans avoir à exécuter le logiciel de sauvegarde personnalisé à l’intérieur de l’ordinateur virtuel.  Il existe plusieurs approches sont disponibles pour les développeurs à utiliser en fonction de leurs besoins.
## <a name="hyper-v-vss-writer"></a>Enregistreur VSS de Hyper-V
Hyper-V implémente un rédacteur sur toutes les versions de Windows Server où Hyper-V est pris en charge.  Cet enregistreur VSS permet aux développeurs d’utiliser l’infrastructure VSS sauvegarde des ordinateurs virtuels.  Toutefois, il est conçu pour les opérations de sauvegarde à petite échelle où tous les ordinateurs virtuels sur un serveur sont sauvegardés simultanément.

Pour mieux comprendre cette architecture, reportez-vous à la présentation:https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-V WMI en fonction de sauvegarde
À partir de Windows Server 2016, Hyper-V démarré prise en charge de sauvegarde par le biais de l’API WMI Hyper-V.  Cette approche utilise toujours VSS à l’intérieur de l’ordinateur virtuel à des fins de sauvegarde, mais ne les utilise plus VSS dans le système d’exploitation hôte.  Au lieu de cela, une combinaison de points de référence et résistantes (RCT) de suivi des modifications est utilisée pour permettre aux développeurs d’accéder aux informations sur les sauvegarder des ordinateurs virtuels de manière efficace.  Cette approche est plus évolutive que l’utilisation de VSS dans l’hôte, mais il est uniquement disponible sur Windows Server 2016 et les versions ultérieures.

Pour mieux comprendre cette architecture, reportez-vous à la présentation:https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

Il existe également un exemple sur la façon d’utiliser ces API disponible ici:https://msconfiggallery.cloudapp.net/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Méthodes pour la lecture des sauvegardes de WMI en fonction de sauvegarde
Lorsque vous créez des sauvegardes de machine virtuelle à l’aide de WMI Hyper-V, il existe trois méthodes pour lire les données réelles à partir de la sauvegarde.  Chacun a des avantages et ses inconvénients.
### <a name="wmi-export"></a>Exportation WMI
Les développeurs peuvent exporter les données de sauvegarde via les interfaces WMI Hyper-V (comme dans l’exemple ci-dessus).  Hyper-V sera compiler les modifications dans un disque dur virtuel et copiez le fichier à l’emplacement demandé.  Cette méthode est facile à utiliser, fonctionne pour tous les scénarios et est accessible à distance.  Toutefois, le disque dur virtuel généré souvent crée une grande quantité de données à transférer via le réseau.
### <a name="win32-apis"></a>API Win32
Les développeurs peuvent utiliser la SetVirtualDiskInformation, GetVirtualDiskInformation et QueryChangesVirtualDisk APIs sur l’API Win32 de disque dur virtuel défini comme indiqué ici: https://docs.microsoft.com/en-us/windows/desktop/api/_vhd/ que pour utiliser ces API, Hyper-V WMI toujours doit être utilisé pour créer la référence de Note points sur les machines virtuelles associées.  Ces API Win32 puis autoriser pour faciliter l’accès aux données de l’ordinateur virtuel sauvegardée.  Les API Win32 ont plusieurs limitations:
*   Ils sont uniquement accessibles localement
*   Ne prend pas en charge la lecture des données à partir de partagé des fichiers de disque dur virtuel
*   Elles retournent des adresses de données sont par rapport à la structure du disque dur virtuel interne

### <a name="remote-shared-virtual-disk-protocol"></a>Protocole de disque virtuel partagé distant
Enfin, si un développeur doit efficacement accéder aux informations de données de sauvegarde à partir d’un fichier de disque dur virtuel partagé – ils vous devrez utiliser le protocole de disque virtuel partagé à distance.  Ce protocole est décrit ici:https://msdn.microsoft.com/en-us/library/dn393384.aspx

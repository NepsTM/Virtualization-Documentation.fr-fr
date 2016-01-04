# Migrer et mettre à niveau des machines virtuelles

Si vous déplacez vers votre hôte Windows 10 des machines virtuelles créées à l’origine avec Hyper-V dans Windows 8.1 ou une version antérieure, vous ne pouvez pas utiliser les nouvelles fonctionnalités de machine virtuelle à moins de mettre à jour manuellement la version de configuration des machines virtuelles.

Pour mettre à jour la version de configuration, arrêtez la machine virtuelle, puis sélectionnez Mettre à niveau la configuration de machine virtuelle dans le Gestionnaire Hyper-V. Vous pouvez également ouvrir une invite de commandes Windows PowerShell avec élévation de privilèges et taper :

 ```PowerShell
Update-VmVersion <vmname> | <vmobject>
 ```

## Comment vérifier la version de configuration des machines virtuelles exécutées sur Hyper-V ?

Pour rechercher la version de configuration, ouvrez une invite de commandes Windows PowerShell avec élévation de privilèges et exécutez la commande suivante :

**Get-VM * | Format-Table Name, Version**

L’invite de commandes PowerShell produit l’exemple de sortie suivant :

```
Name        State       CPUUsage(%) MemoryAssigned(M)   Uptime              Status                  Version

Atlantis    Running         0       1024                00:04:20.5910000    Operating normally      5.0

SGC VM      Running         0       538                 00:02:44.8350000    Operating normally      6.2
```


## Que se passe-t-il si je ne mets pas à jour la version de configuration ?

Si vous avez créé des machines virtuelles avec une version antérieure d’Hyper-V, certaines fonctionnalités peuvent ne pas fonctionner avec ces machines virtuelles tant que vous ne mettez pas à jour la version des machines virtuelles.

Version minimale de configuration de machine virtuelle pour les nouvelles fonctionnalités Hyper-V :

| **Nom de la fonctionnalité**| **Version minimale de machine virtuelle**| |
| :------------------------------------- | -----------------: ||
| Ajout/suppression de mémoire à chaud| 6.0| |
| Ajout/suppression de cartes réseau à chaud| 5.0| |
| Démarrage sécurisé pour les machines virtuelles Linux| 6.0| |
| Points de contrôle de production| 6.0| |
| PowerShell Direct| 6.2| |
| Module de plateforme sécurisée virtuelle (vTPM)| 6.2| |
| Regroupement de machines virtuelles| 6.2| |



## Version de configuration de machine virtuelle

Quand vous déplacez ou importez une machine virtuelle vers un hôte exécutant Hyper-V sur Windows 10 à partir d’un hôte exécutant Windows 8.1, le fichier de configuration de la machine virtuelle n’est pas automatiquement mis à niveau. Vous pouvez ainsi redéplacer la machine virtuelle vers un hôte exécutant Windows 8.1. Pour bénéficier des nouvelles fonctionnalités de la machine virtuelle, vous devez tout d’abord mettre manuellement à jour la version de configuration de la machine virtuelle.

Celle-ci correspond à la version d’Hyper-V compatible avec la configuration, l’état de mise en mémoire et les fichiers de capture instantanée de la machine virtuelle. Les machines virtuelles avec la version de configuration 5 sont compatibles avec Windows 8.1 et peuvent s’exécuter sur Windows 8.1 et Windows 10. Les machines virtuelles avec la version de configuration 6 sont compatibles avec Windows 10, mais ne peuvent pas s’exécuter sur Windows 8.1.

Vous n’avez pas besoin de mettre à niveau toutes vos machines virtuelles simultanément. Vous pouvez choisir de mettre à niveau certaines machines virtuelles quand cela est nécessaire. Toutefois, vous n’avez pas accès aux nouvelles fonctionnalités de machine virtuelle à moins de mettre à jour manuellement la version de configuration de chaque machine virtuelle.


----------------

**Important **

• Après avoir mis à niveau la version de configuration de la machine virtuelle, vous ne pouvez pas déplacer la machine virtuelle vers un hôte exécutant Windows 8.1.

• Vous ne pouvez pas passer de la version de configuration 6 de machine virtuelle à la version de configuration 5.

• Vous devez éteindre la machine virtuelle pour mettre à niveau sa configuration.

• Après la mise à niveau, la machine virtuelle utilise le nouveau format du fichier de configuration. Pour plus d’informations, voir Nouveau format du fichier de configuration de machine virtuelle.

--------






## Que se passe-t-il quand je mets à niveau la version d’une machine virtuelle ?

Quand vous mettez à niveau manuellement la version de configuration d’une machine virtuelle vers la version 6.x, vous modifiez la structure de fichiers utilisée pour stocker les fichiers de configuration et de point de contrôle des machines virtuelles.

Les machines virtuelles mises à niveau utilisent un nouveau format de fichier de configuration qui est conçu pour accroître les performances de lecture et d’écriture des données de configuration de machine virtuelle. La mise à niveau permet également de réduire le risque de corruption des données en cas de défaillance du stockage.

Le tableau suivant répertorie les emplacements des fichiers binaires et les informations d’extension pour une machine virtuelle mise à niveau.

| **Fichiers de configuration et de point de contrôle de machine virtuelle (version 6.x)**| **Description**| |
|:---------------|:----------------||
| **Configuration de machine virtuelle**| Les informations de configuration sont stockées dans un format de fichier binaire qui utilise l’extension .vmcx.| |
| **État d’exécution de machine virtuelle**| Les données sur l’état d’exécution sont stockées dans un format de fichier binaire qui utilise l’extension .vmrs.| |
| **Disque dur virtuel de machine virtuelle**| Fichiers de disque dur virtuel pour la machine virtuelle.Ils utilisent les extensions de fichier .vhd ou .vhdx.| |
| **Fichiers de disque dur virtuel automatiques**| Fichiers de disque de différenciation utilisés pour les points de contrôle de machine virtuelle.Ils utilisent les extensions de fichier .avhdx.| |
| **Fichiers de point de contrôle**| Les points de contrôle sont stockés dans plusieurs fichiers de point de contrôle.Chaque point de contrôle crée un fichier de configuration et un fichier d’état d’exécution.Les fichiers de point de contrôle utilisent les extensions de fichier .vmrs et .vmcx.Ces nouveaux formats de fichier sont également utilisés pour les points de contrôle de production et les points de contrôle standard.| |

Une fois que vous avez mis à niveau la configuration de machine virtuelle vers une version 6.x, vous ne pouvez plus passer de la version 6.x à la version 5.

Vous devez éteindre la machine virtuelle pour mettre à niveau sa configuration.

Le tableau suivant répertorie les emplacements de fichier par défaut pour les machines virtuelles nouvelles ou mises à niveau.

| **Fichiers de machine virtuelle (Version 6.x)**| **Description**| |
|:-----|:-----||
| **Fichier de configuration de machine virtuelle**| C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines| |
| **Fichier d’état d’exécution de machine virtuelle**| C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines| |
| **Fichiers de point de contrôle (.vmrs, .vmcx)**| C:\ProgramData\Microsoft\Windows\Snapshots| |
| **Fichier de disque dur virtuel (.vhd/.vdhx)**| C:\Users\Public\Documents\Hyper-V\Virtual Hard Disks| |
| **Fichiers de disque dur virtuel automatiques (.avhdx)**| C:\Users\Public\Documents\Hyper-V\Virtual Hard Disks| |









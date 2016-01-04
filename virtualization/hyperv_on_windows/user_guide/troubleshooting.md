# Résoudre les problèmes liés à Hyper-V sur Windows 10

## Je viens d’effectuer la mise à jour vers Windows 10 et je ne peux plus me connecter à mon hôte de niveau inférieur (Windows 8.1 ou Windows Server 2012 R2)

Dans Windows 10, le Gestionnaire Hyper-V se trouve dans WinRM pour la gestion à distance. Cela signifie que vous devez à présent activer la gestion à distance sur l’hôte distant afin d’utiliser le Gestionnaire Hyper-V pour le gérer.

Pour plus d’informations, voir [Gestion d’hôtes Hyper-V distants](remote_host_management.md).

## J’ai modifié le type de point de contrôle, mais c’est encore le mauvais type de point de contrôle qui est pris

Si vous prenez le point de contrôle de VMConnect et que vous modifiez le type de point de contrôle dans le Gestionnaire Hyper-V, le type du point de contrôle pris est celui qui a été spécifié lors de l’ouverture de VMConnect.

Fermez et rouvrez VMConnect pour que le bon type de point de contrôle soit pris.

## Quand j’essaie de créer un disque dur virtuel sur un lecteur flash, un message d’erreur s’affiche

Hyper-V ne prend pas en charge les lecteurs de disque au format FAT/FAT32, car ces systèmes de fichiers ne fournissent pas de listes de contrôle d’accès et ne prennent pas en charge les fichiers de plus de 4 Go. Les disques au format ExFAT fournissent uniquement des listes de contrôle d’accès avec des fonctionnalités limitées. Ils ne sont donc pas non plus pris en charge pour des raisons de sécurité.
Le message d’erreur affiché dans PowerShell est le suivant : « Le système n’a pas pu créer ’\[chemin du disque dur virtuel\]’. Impossible de terminer l’opération demandée du fait d’une limitation du système de fichiers (0x80070299). »

Utilisez plutôt un lecteur au format NTFS.

## J’obtiens le message suivant pendant l’installation : « Impossible d’installer Hyper-V : le processeur ne prend pas en charge la traduction d’adresse de second niveau (SLAT). »

Hyper-V a besoin de SLAT pour exécuter des machines virtuelles. Si votre ordinateur ne prend pas en charge SLAT, vous ne pouvez pas l’utiliser en tant qu’hôte pour des machines virtuelles.

Si vous essayez d’installer uniquement les outils de gestion, désélectionnez **Plateforme Hyper-V** dans **Programmes et fonctionnalités** > **Activer ou désactiver des fonctionnalités Windows**.





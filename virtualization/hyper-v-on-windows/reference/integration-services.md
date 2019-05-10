---
title: Services d’intégration Hyper-V
description: Informations de référence pour les services d’intégration Hyper-V
keywords: windows 10, hyper-v, services d’intégration, composants d’intégration
author: scooley
ms.date: 05/25/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 18930864-476a-40db-aa21-b03dfb4fda98
ms.openlocfilehash: 762b82f3714651ffb488f682581680c9526404a8
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621207"
---
# <a name="hyper-v-integration-services"></a>Services d’intégration Hyper-V

Les services d’intégration (souvent appelés composants d’intégration) sont des services qui permettent à la machine virtuelle de communiquer avec l’hôte Hyper-V. Plusieurs de ces services sont pratiques, mais d’autres peuvent être très importants dans le fonctionnement de la machine virtuelle.

Cet article est une référence pour chaque service d’intégration disponible dans Windows.  Il représente également un point de départ pour toutes les informations relatives à des services d’intégration spécifiques ou à leur historique.

**Guides de l’utilisateur:**  
* [Gestion des services d’intégration](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/Manage-Hyper-V-integration-services)


## <a name="quick-reference"></a>Référence rapide

| Nom | Nom du service Windows | Nom de démon Linux |  Description | Impact de sa désactivation sur la machine virtuelle |
|:---------|:---------|:---------|:---------|:---------|
| [Service Pulsation Microsoft Hyper-V](#hyper-v-heartbeat-service) |  vmicheartbeat | hv_utils | Signale que la machine virtuelle fonctionne correctement. | Varie |
| [Service Arrêt de l’invité Microsoft Hyper-V](#hyper-v-guest-shutdown-service) | vmicshutdown | hv_utils |  Permet à l’hôte de déclencher l’arrêt des machines virtuelles. | **Importante** |
| [Service Synchronisation date/heure Microsoft Hyper-V](#hyper-v-time-synchronization-service) | vmictimesync | hv_utils | Synchronise l’horloge de la machine virtuelle avec celle de l’ordinateur hôte. | **Importante** |
| [Service Échange de données Microsoft Hyper-V](#hyper-v-data-exchange-service-kvp) | vmickvpexchange | hv_kvp_daemon | Fournit un moyen d’échanger des métadonnées de base entre la machine virtuelle et l’hôte. | Moyen |
| [Requête du service VSS Microsoft Hyper-V](#hyper-v-volume-shadow-copy-requestor) | vmicvss | hv_vss_daemon | Permet au service VSS de sauvegarder la machine virtuelle sans l’arrêter. | Varie |
| [Interface de services d’invité Hyper-V](#hyper-v-powershell-direct-service) | vmicguestinterface | hv_fcopy_daemon | Fournit une interface pour que l’hôte Hyper-V copie des fichiers vers ou depuis la machine virtuelle. | Faible |
| [Service PowerShell Direct Hyper-V](#hyper-v-powershell-direct-service) | vmicvmsession | non disponibles | Fournit un moyen de gérer la machine virtuelle avec PowerShell sans connexion réseau. | Faible |  


## <a name="hyper-v-heartbeat-service"></a>Service Pulsation Microsoft Hyper-V

**Nom du service Windows:** vmicheartbeat  
**Nom de démon Linux:** hv_utils  
**Description:** indique à l’hôte Hyper-V que la machine virtuelle dispose d’un système d’exploitation et a démarré correctement.  
**Ajout dans:** Windows Server2012, Windows8  
**Impact:** quand il est désactivé, la machine virtuelle ne peut pas signaler que le système d’exploitation à l’intérieur de la machine virtuelle fonctionne correctement.  Cela peut affecter certains types de diagnostics d’analyse et côté hôte.  

Le service Pulsation permet de répondre à des questions de base comme «est-ce que la machine virtuelle a démarré?».  

Quand Hyper-V signale que l’état d’une machine virtuelle est «en cours d’exécution» (voir l’exemple ci-dessous), cela signifie que Hyper-V a réservé des ressources pour une machine virtuelle, mais pas qu’un système d’exploitation est installé ou en train de fonctionner.  C’est là où le service Pulsation devient utile.  Le service Pulsation indique à Hyper-V que le système d’exploitation à l’intérieur de la machine virtuelle a démarré.  

### <a name="check-heartbeat-with-powershell"></a>Vérifier les pulsations avec PowerShell

Exécutez [Get-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps) en tant qu’administrateur pour afficher les pulsations d’une machine virtuelle:
``` PowerShell
Get-VM -VMName $VMName | select Name, State, Status
```

Votre sortie doit ressembler à ceci:
```
Name    State    Status
----    -----    ------
DemoVM  Running  Operating normally
```

Le champ `Status` est déterminé par le service Pulsation.



## <a name="hyper-v-guest-shutdown-service"></a>Service Arrêt de l’invité Microsoft Hyper-V

**Nom du service Windows:** vmicshutdown  
**Nom de démon Linux:** hv_utils  
**Description:** permet à l’hôte Hyper-V de demander l’arrêt de la machine virtuelle.  L’hôte peut toujours forcer l’arrêt de la machine virtuelle, mais cela revient à utiliser le bouton marche/arrêt au lieu de sélectionner Arrêter.  
**Ajout dans:** Windows Server2012, Windows8  
**Impact:** **Impact élevé** Quand il est désactivé, l’hôte ne peut pas déclencher un arrêt convivial à l’intérieur de la machine virtuelle.  Tous les arrêts sera un disque dur hors tension, qui pourrait entraîner une altération des données perte ou de données.  


## <a name="hyper-v-time-synchronization-service"></a>Service Synchronisation date/heure Microsoft Hyper-V

**Nom du service Windows:** vmictimesync  
**Nom de démon Linux:** hv_utils  
**Description:** synchronise l’horloge système de la machine virtuelle avec celle de l’ordinateur physique.  
**Ajout dans:** Windows Server2012, Windows8  
**Impact:** **Impact élevé** Quand il est désactivé, l’horloge de la machine virtuelle dérive de manière irrégulière.  


## <a name="hyper-v-data-exchange-service-kvp"></a>Service Échange de données Microsoft Hyper-V

**Nom du service Windows:** vmickvpexchange  
**Nom de démon Linux:** hv_kvp_daemon  
**Description:** fournit un mécanisme pour échanger des métadonnées de base entre la machine virtuelle et l’hôte.  
**Ajout dans:** Windows Server2012, Windows8  
**Impact:** quand il est désactivé, les machines virtuelles exécutant Windows8 ou Windows Server2012 ou version antérieure ne reçoivent pas les mises à jour des services d’intégration Hyper-V.  La désactivation de l’échange de données peut également affecter certains types de diagnostics d’analyse et côté hôte.  

Le service Échange de données (parfois appelé paire clé/valeur) partage de petites quantités d’informations sur l’ordinateur entre la machine virtuelle et l’hôte Hyper-V à l’aide de paires clé/valeur (KVP) via le Registre Windows.  Le même mécanisme peut également servir à partager des données personnalisées entre la machine virtuelle et l’hôte.

Les paires clé/valeur sont constituées d’une «clé» et d’une «valeur». La clé et la valeur sont toutes deux des chaînes; aucun autre type de données n’est pris en charge. Lors de la création ou modification d’une paire clé/valeur, elle est visible pour l’invité et pour l’hôte. Les informations sur la paire clé/valeur sont transférées sur le bus VMBus Hyper-V et ne nécessitent aucun type de connexion réseau entre l’invité et l’hôte Hyper-V. 

Le service Échange de données est un excellent outil pour conserver les informations sur la machine virtuelle; pour le partage de données interactif ou le transfert de données, utilisez [PowerShell Direct](#hyper-v-powershell-direct-service). 


**Guides de l’utilisateur:**  
* [Utilisation de paires clé/valeur pour partager des informations entre l’hôte et l’invité sur Hyper-V](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn798287(v=ws.11)).  


## <a name="hyper-v-volume-shadow-copy-requestor"></a>Requête du service VSS Microsoft Hyper-V

**Nom du service Windows:** vmicvss  
**Nom de démon Linux:** hv_vss_daemon  
**Description:** autorise le service VSS à sauvegarder des données et applications sur la machine virtuelle.  
**Ajout dans:** Windows Server2012, Windows8  
**Impact:** quand il est désactivé, la machine virtuelle ne peut pas être sauvegardée pendant qu’elle est en cours d’exécution (avec VSS).  

Le service d’intégration Requête du service VSS est nécessaire pour le service VSS ([VSS](https://docs.microsoft.com/windows/desktop/VSS/overview-of-processing-a-backup-under-vss)).  Le service VSS capture et copie des images pour la sauvegarde sur les systèmes en cours d’exécution, notamment les serveurs, sans trop dégrader les performances et la stabilité des services fournis.  Ce service d’intégration rend cela possible en coordonnant les charges de travail de la machine virtuelle avec le processus de sauvegarde de l’hôte.

En savoir plus sur le service VSS [ici](https://docs.microsoft.com/previous-versions/windows/desktop/virtual/backing-up-and-restoring-virtual-machines).


## <a name="hyper-v-guest-service-interface"></a>Interface de services d’invité Hyper-V

**Nom du service Windows:** vmicguestinterface  
**Nom de démon Linux:** hv_fcopy_daemon  
**Description:** fournit une interface pour que l’hôte Hyper-V copie de façon bidirectionnelle des fichiers vers ou depuis la machine virtuelle.  
**Ajout dans:** Windows Server2012R2, Windows8.1  
**Impact:** quand il est désactivé, l’hôte ne peut pas effectuer la copie de fichiers vers et depuis l’invité avec `Copy-VMFile`.  En savoir plus sur l’[applet de commande Copy-VMFile](https://docs.microsoft.com/powershell/module/hyper-v/copy-vmfile?view=win10-ps).  

**Remarques:**  
Désactivé par défaut.  Voir [PowerShell Direct avec Copy-Item](../user-guide/powershell-direct.md#copy-files-with-new-pssession-and-copy-item). 


## <a name="hyper-v-powershell-direct-service"></a>Service PowerShell Direct Hyper-V

**Nom du service Windows:** vmicvmsession  
**Nom de démon Linux:** non applicable  
**Description:** fournit un mécanisme de gestion de machine virtuelle avec PowerShell via la session de machine virtuelle sans réseau virtuel.    
**Ajout dans:** Windows ServerTP3, Windows10  
**Impact:** la désactivation de ce service empêche l’hôte d’être en mesure de se connecter à la machine virtuelle avec PowerShell Direct.  

**Remarques:**  
Le nom du service était à l’origine Service de session d’ordinateur virtuel Hyper-V.  
PowerShell Direct est en cours de développement actif et disponible uniquement sur Windows10 et Windows Server Technical Preview3 ou hôtes/invités ultérieurs.

Grâce à PowerShell Direct, vous pouvez gérer PowerShell dans une machine virtuelle à partir de l’hôte Hyper-V, indépendamment de la configuration réseau ou des paramètres de gestion à distance sur l’hôte Hyper-V ou la machine virtuelle. Pour les administrateurs Hyper-V, cela facilite l’automatisation et la définition par script des tâches de gestion et de configuration.

[En savoir plus sur PowerShell Direct](../user-guide/powershell-direct.md).  

**Guides de l’utilisateur:**  
* [Exécution de script dans une machine virtuelle](../user-guide/powershell-direct.md#run-a-script-or-command-with-invoke-command)
* [Copie de fichiers vers et depuis une machine virtuelle](../user-guide/powershell-direct.md#copy-files-with-new-pssession-and-copy-item)

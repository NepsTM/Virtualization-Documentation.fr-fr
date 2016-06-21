---
title: &1152148439 Gestion des services d’intégration Hyper-V
description: Gestion des services d’intégration Hyper-V
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &1773816246 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 9cafd6cb-dbbe-4b91-b26c-dee1c18fd8c2
---

# Gestion des services d’intégration Hyper-V

Les services d’intégration (souvent appelés composants d’intégration) sont des services qui permettent à la machine virtuelle de communiquer avec l’hôte Hyper-V. Plusieurs de ces services sont pratiques (comme la copie de fichier de l’invité), mais d’autres peuvent être très importants dans le fonctionnement du système d’exploitation invité (synchronisation date/heure).

Cet article décrit en détail comment gérer les services d’intégration à l’aide du Gestionnaire Hyper-V et de PowerShell dans Windows 10. Pour plus d’informations sur chaque service d’intégration, voir <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Services d’intégration</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

## Activer ou désactiver les services d’intégration à l’aide du Gestionnaire Hyper-V

1. Sélectionnez une machine virtuelle et ouvrez les paramètres.
  <g id="1" ctype="x-linkText"></g>

2. Dans la fenêtre de paramètres de la machine virtuelle, accédez à l’onglet Services d’intégration sous Gestion.

  <g id="1" ctype="x-linkText"></g>

  Dans cet onglet sont affichés tous les services d’intégration disponibles sur cet hôte Hyper-V. Il est important de noter que le système d’exploitation invité peut ou ne peut pas prendre en charge la totalité des services d’intégration répertoriés.

## Activer ou désactiver les services d’intégration à l’aide de PowerShell

Les services d’intégration peuvent également être activés et désactivés avec PowerShell en exécutant [<g id="2" ctype="x-code">Enable-VMIntegrationService</g>] (https://technet.microsoft.com/en-us/library/hh848500.aspx) et [<g id="4" ctype="x-code">Disable-VMIntegrationService</g>] (https://technet.microsoft.com/en-us/library/hh848488.aspx).

Dans cet exemple, nous activons et désactivons le service d’intégration de copie de fichier de l’invité sur la machine virtuelle « demovm » citée plus haut.

1. Afficher les services d’intégration en cours d’exécution

  ``` PowerShell
  Get-VMIntegrationService -VMName "demovm"
  ```

  La sortie doit ressembler à ceci :
  ``` PowerShell
  VMName      Name                    Enabled PrimaryStatusDescription SecondaryStatusDescription
  ------      ----                    ------- ------------------------ --------------------------
  demovm      Guest Service Interface False   OK
  demovm      Heartbeat               True    OK                       OK
  demovm      Key-Value Pair Exchange True    OK
  demovm      Shutdown                True    OK
  demovm      Time Synchronization    True    OK
  demovm      VSS                     True    OK
  ```

2. Activer le service d’intégration <g id="2" ctype="x-code">Interface de services d’invité</g>

   ``` PowerShell
   Enable-VMIntegrationService -VMName "demovm" -Name "Guest Service Interface"
   ```

   Si vous exécutez la commande <g id="2" ctype="x-code">Get-VMIntegrationService - VMName "demovm"</g>, vous constatez que le service d’intégration Interface de services d’invité est activé.

3. Désactiver le service d’intégration <g id="2" ctype="x-code">Interface de services d’invité</g>

   ``` PowerShell
   Disable-VMIntegrationService -VMName "demovm" -Name "Guest Service Interface"
   ```

Les services d’intégration ont été conçus de telle sorte qu’ils ont besoin d’être activés dans l’hôte et l’invité pour fonctionner. Bien que tous les services d’intégration soient activés par défaut sur les systèmes d’exploitation Windows invités, ils peuvent être désactivés. Pour en savoir plus, voir la section suivante.


## Gérer les services d’intégration à partir du système d’exploitation invité (Windows)

> <g id="1" ctype="x-strong">Remarque :</g> la désactivation des services d’intégration peut avoir un impact considérable sur la capacité de l’hôte à gérer votre machine virtuelle. Les services d’intégration doivent être activés sur l’hôte et l’invité pour fonctionner.

Les services d’intégration apparaissent en tant que services dans Windows. Pour activer ou désactiver des services d’intégration à partir de la machine virtuelle, ouvrez le Gestionnaire des services Windows.

<g id="1" ctype="x-linkText"></g>

Recherchez les services dont le nom contient Hyper-V. Cliquez avec le bouton droit sur le service que vous voulez activer ou désactiver, et démarrez ou arrêtez le service.

Vous pouvez aussi afficher tous les services d’intégration avec PowerShell, pour ce faire, exécutez la commande suivante :

```PowerShell
Get-Service -Name vm*
```

qui retourne une liste qui ressemble à ceci :

```PowerShell
Status   Name               DisplayName
------   ----               -----------
Running  vmicguestinterface Hyper-V Guest Service Interface
Running  vmicheartbeat      Hyper-V Heartbeat Service
Running  vmickvpexchange    Hyper-V Data Exchange Service
Running  vmicrdv            Hyper-V Remote Desktop Virtualizati...
Running  vmicshutdown       Hyper-V Guest Shutdown Service
Running  vmictimesync       Hyper-V Time Synchronization Service
Stopped  vmicvmsession      Hyper-V VM Session Service
Running  vmicvss            Hyper-V Volume Shadow Copy Requestor
```

Démarrez ou arrêtez des services à l’aide de [<g id="2" ctype="x-code">Start-Service</g>](https://technet.microsoft.com/en-us/library/hh849825.aspx) ou [<g id="4" ctype="x-code">Stop-Service</g>](https://technet.microsoft.com/en-us/library/hh849790.aspx).

Par exemple, pour désactiver PowerShell Direct, vous pouvez exécuter <g id="2" ctype="x-code">Stop-Service -Name vmicvmsession</g>.

Par défaut, tous les services d’intégration sont activés dans le système d’exploitation invité.

## Gérer les services d’intégration à partir du système d’exploitation invité (Linux)

Les services d’intégration Linux sont généralement fournis par le noyau Linux.

Vérifiez que le pilote et les démons des services d’intégration sont en cours d’exécution en exécutant les commandes suivantes dans votre système de d’exploitation invité Linux.

1. Le pilote des services d’intégration Linux est appelé « hv_utils ». Exécutez la commande suivante pour vérifier qu’il est chargé.

  ``` BASH
  lsmod | grep hv_utils
  ```

  La sortie doit ressembler à ceci :

  ``` BASH
  Module                  Size   Used by
  hv_utils               20480   0
  hv_vmbus               61440   8 hv_balloon,hyperv_keyboard,hv_netvsc,hid_hyperv,hv_utils,hyperv_fb,hv_storvsc
  ```

2. Exécutez la commande suivante dans votre système d’exploitation invité Linux pour vérifier que les démons nécessaires sont en cours d’exécution.

  ``` BASH
  ps -ef | grep hv
  ```

  La sortie doit ressembler à ceci :

  ``` BASH
  root       236     2  0 Jul11 ?        00:00:00 [hv_vmbus_con]
  root       237     2  0 Jul11 ?        00:00:00 [hv_vmbus_ctl]
  ...
  root       252     2  0 Jul11 ?        00:00:00 [hv_vmbus_ctl]
  root      1286     1  0 Jul11 ?        00:01:11 /usr/lib/linux-tools/3.13.0-32-generic/hv_kvp_daemon
  root      9333     1  0 Oct12 ?        00:00:00 /usr/lib/linux-tools/3.13.0-32-generic/hv_kvp_daemon
  root      9365     1  0 Oct12 ?        00:00:00 /usr/lib/linux-tools/3.13.0-32-generic/hv_vss_daemon
  scooley  43774 43755  0 21:20 pts/0    00:00:00 grep --color=auto hv          
  ```

  Pour afficher les démons disponibles, exécutez la commande suivante :
  ``` BASH
  compgen -c hv_
  ```

  La sortie doit ressembler à ceci :

  ``` BASH
  hv_vss_daemon
  hv_get_dhcp_info
  hv_get_dns_info
  hv_set_ifconfig
  hv_kvp_daemon
  hv_fcopy_daemon     
  ```

  Processus de services intégration qui peuvent s’afficher :
  * **<g id="2" ctype="x-code">hv_vss_daemon</g>** : ce démon est nécessaire pour créer des sauvegardes dynamiques de machines virtuelles Linux.
  * **<g id="2" ctype="x-code">hv_kvp_daemon</g>**: ce démon permet de définir et d’interroger des paires clé-valeur intrinsèques et extrinsèques.
  * **<g id="2" ctype="x-code">hv_fcopy_daemon</g>** : ce démon implémente un service de copie de fichiers entre l’hôte et l’invité.

> <g id="1" ctype="x-strong">Remarque :</g> si les démons de services d’intégration ci-dessus ne sont pas disponibles, ils ne sont peut-être pas pris en charge sur votre système ou installés. Pour en savoir plus sur la distribution, cliquez <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">ici</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

Dans cet exemple, nous allons arrêter et démarrer le démon KVP <g id="2" ctype="x-code">hv_kvp_daemon</g>.

Arrêtez le processus du démon à l’aide du PID (ID de processus) situé dans la deuxième colonne de la sortie ci-dessus. Vous pouvez aussi rechercher le processus adéquat à l’aide de <g id="2" ctype="x-code">pidof</g>. Étant donné que les démons Hyper-V s’exécutent en tant que racine, vous avez besoin d’autorisations racine.

``` BASH
sudo kill -15 `pidof hv_kvp_daemon`
```

Si vous réexécutez <g id="2" ctype="x-code">ps -ef | hv</g>, vous constatez que tous les processus <g id="4" ctype="x-code">hv_kvp_daemon</g> ont disparu.

Pour redémarrer le démon, exécutez-le en tant que racine.

``` BASH
sudo hv_kvp_daemon
```

Si vous réexécutez <g id="2" ctype="x-code">ps -ef | hv</g>, vous découvrez un processus <g id="4" ctype="x-code">hv_kvp_daemon</g> avec un nouvel ID de processus.


## Maintenance des services d’intégration

Maintenez les services d’intégration à jour pour bénéficier des meilleures performances et fonctionnalités de machine virtuelle.

<g id="1" ctype="x-strong">Pour les machines virtuelles qui s’exécutent sur des hôtes Windows 10 :</g>

> <g id="1" ctype="x-strong">Remarque :</g> Le fichier image ISO vmguest.iso n’est plus obligatoire pour mettre à jour les composants d’intégration. Il n’est donc pas fourni avec Hyper-V sur Windows 10.

| Système d'exploitation invité| Mécanisme de mise à jour| Remarques|
|:---------|:---------|:---------|
| Windows 10| Windows Update| |
| Windows 8.1| Windows Update| |
| Windows 8| Windows Update| Nécessite le service d’intégration Échange de données.<g id="2" ctype="x-strong">*</g>|
| Windows 7| Windows Update| Nécessite le service d’intégration Échange de données.<g id="2" ctype="x-strong">*</g>|
| Windows Vista (SP2)| Windows Update| Nécessite le service d’intégration Échange de données.<g id="2" ctype="x-strong">*</g>|
| -| | |
| Windows Server 2012 R2| Windows Update| |
| Windows Server 2012| Windows Update| Nécessite le service d’intégration Échange de données.<g id="2" ctype="x-strong">*</g>|
| Windows Server 2008 R2 (SP1)| Windows Update| Nécessite le service d’intégration Échange de données.<g id="2" ctype="x-strong">*</g>|
| Windows Server 2008 (SP2)| Windows Update| Support étendu uniquement dans Server 2016 (<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">en savoir plus</g><g id="2CapsExtId3" ctype="x-title"></g></g>.|
| Windows Home Server 2011| Windows Update| Pas de support dans Server 2016 (<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">en savoir plus</g><g id="2CapsExtId3" ctype="x-title"></g></g>.|
| Windows Small Business Server 2011| Windows Update| Pas de support standard (<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">en savoir plus</g><g id="2CapsExtId3" ctype="x-title"></g></g>.|
| -| | |
| Invités Linux| gestionnaire de package| Les composants d’intégration pour Linux sont intégrés à la distribution, mais des mises à jour facultatives peuvent être disponibles.<g id="1" ctype="x-strong">****</g>|

<g id="1" ctype="x-strong">\*</g> Si le service d’intégration Échange de données ne peut pas être activé, les composants d’intégration pour ces invités sont disponibles <g id="3CapsExtId1" ctype="x-link"><g id="3CapsExtId2" ctype="x-linkText">ici</g><g id="3CapsExtId3" ctype="x-title"></g></g> en tant que fichier cabinet (cab) dans le Centre de téléchargement. Les instructions permettant d’appliquer un fichier cab sont disponibles <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">ici</g><g id="2CapsExtId3" ctype="x-title"></g></g>.


<g id="1" ctype="x-strong">Pour les machines virtuelles qui s’exécutent sur des hôtes Windows 8.1 :</g>

| Système d’exploitation invité| Mécanisme de mise à jour| Remarques|
|:---------|:---------|:---------|
| Windows 10| Windows Update| |
| Windows 8.1| Windows Update| |
| Windows 8| Disque des services d’intégration| |
| Windows 7| Disque des services d’intégration| |
| Windows Vista (SP2)| Disque des services d’intégration| |
| Windows XP (SP2, SP3)| Disque des services d’intégration| |
| -| | |
| Windows Server 2012 R2| Windows Update| |
| Windows Server 2012| Disque des services d’intégration| |
| Windows Server 2008 R2| Disque des services d’intégration| |
| Windows Server 2008 (SP2)| Disque des services d’intégration| |
| Windows Home Server 2011| Disque des services d’intégration| |
| Windows Small Business Server 2011| Disque des services d’intégration| |
| Windows Server 2003 R2 (SP2)| Disque des services d’intégration| |
| Windows Server 2003 (SP2)| Disque des services d’intégration| |
| -| | |
| Invités Linux| gestionnaire de package| Les composants d’intégration pour Linux sont intégrés à la distribution, mais des mises à jour facultatives peuvent être disponibles.<g id="1" ctype="x-strong">****</g>|


<g id="1" ctype="x-strong">Pour les machines virtuelles qui s’exécutent sur des hôtes Windows 8 :</g>

| Système d’exploitation invité| Mécanisme de mise à jour| Remarques|
|:---------|:---------|:---------|
| Windows 8.1| Windows Update| |
| Windows 8| Disque des services d’intégration| |
| Windows 7| Disque des services d’intégration| |
| Windows Vista (SP2)| Disque des services d’intégration| |
| Windows XP (SP2, SP3)| Disque des services d’intégration| |
| -| | |
| Windows Server 2012 R2| Windows Update| |
| Windows Server 2012| Disque des services d’intégration| |
| Windows Server 2008 R2| Disque des services d’intégration| |
| Windows Server 2008 (SP2)| Disque des services d’intégration| |
| Windows Home Server 2011| Disque des services d’intégration| |
| Windows Small Business Server 2011| Disque des services d’intégration| |
| Windows Server 2003 R2 (SP2)| Disque des services d’intégration| |
| Windows Server 2003 (SP2)| Disque des services d’intégration| |
| -| | |
| Invités Linux| gestionnaire de package| Les composants d’intégration pour Linux sont intégrés à la distribution, mais des mises à jour facultatives peuvent être disponibles.<g id="1" ctype="x-strong">****</g>|


Les instructions relatives à la mise à jour avec le disque des services d’intégration pour Windows 8 et Windows 8.1 sont disponibles <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">ici</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

 <g id="1" ctype="x-strong">\****</g> D’autres informations sur les invités Linux sont disponibles <g id="3CapsExtId1" ctype="x-link"><g id="3CapsExtId2" ctype="x-linkText">ici</g><g id="3CapsExtId3" ctype="x-title"></g></g>.






<!--HONumber=May16_HO1-->



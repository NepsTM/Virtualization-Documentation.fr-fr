---
title: "Gérer des hôtes Hyper-V distants avec le Gestionnaire Hyper-V"
description: "Gérer des hôtes Hyper-V distants avec le Gestionnaire Hyper-V"
keywords: "Windows 10, Hyper-V"
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 2d34e98c-6134-479b-8000-3eb360b8b8a3
translationtype: Human Translation
ms.sourcegitcommit: 07a07c790484c05ea49229a770ef75c80fad3cfa
ms.openlocfilehash: 8a84da80199479907c3bf4cf0c7b1cfb1b44bf9d

---

# Gérer des hôtes Hyper-V distants avec le Gestionnaire Hyper-V

Le Gestionnaire Hyper-V est un outil intégré à Windows pour diagnostiquer et gérer votre hôte Hyper-V local et un petit nombre d’hôtes distants.  Cet article décrit les étapes de configuration pour se connecter aux hôtes Hyper-V à l’aide du Gestionnaire Hyper-V dans toutes les configurations prises en charge.

> Le Gestionnaire Hyper-V est disponible dans **Programmes et fonctionnalités** en tant qu’**Outils de gestion Hyper-V** sur [n’importe quel système d’exploitation Windows incluant Hyper-V](../quick_start/walkthrough_compatibility.md#OperatingSystemRequirements).  La plateforme Hyper-V n’a pas besoin d’être activée pour gérer les hôtes distants.

Pour vous connecter à un hôte Hyper-V dans le Gestionnaire Hyper-V, assurez-vous que **Gestionnaire Hyper-V** est sélectionné dans le volet gauche, puis sélectionnez **Se connecter au serveur...** dans le volet droit.

![](media/HyperVManager-ConnectToHost.png)

## Combinaisons d’hôtes Hyper-V prises en charge par le Gestionnaire Hyper-V
Le Gestionnaire Hyper-V dans Windows 10 vous permet de gérer les hôtes Hyper-V suivants :
* Windows 10
* Windows 8.1
* Windows 8
* Windows Server 2016 + Windows Server Core, Nano Server et Hyper-V Server
* Windows Server 2012 R2 + Windows Server Core, Datacenter et Hyper-V Server
* Windows Server 2012 + Windows Server Core, Datacenter et Hyper-V Server

Le Gestionnaire Hyper-V dans Windows 8.1 et Windows Server 2012 R2 vous permet de gérer les hôtes suivants :
* Windows 8.1
* Windows 8
* Windows Server 2012 R2 + Windows Server Core, Datacenter et Hyper-V Server
* Windows Server 2012 + Windows Server Core, Datacenter et Hyper-V Server

Le Gestionnaire Hyper-V dans Windows 8 et Windows Server 2012 vous permet de gérer les hôtes suivants :
* Windows 8
* Windows Server 2012 + Windows Server Core, Datacenter et Hyper-V Server

Hyper-V est désormais disponible dans Windows 8.  Avant Windows 8.1/Windows Server 2012, le Gestionnaire Hyper-V ne gérait que les versions correspondantes d’Hyper-V.

> **Remarque :** La fonctionnalité du Gestionnaire Hyper-V correspond à la fonctionnalité disponible pour la version que vous gérez.  En d’autres termes, si vous gérez un hôte Windows Server 2012 distant à partir de Windows Server 2012 R2, les nouveaux outils du Gestionnaire Hyper-V de Windows Server 2012 R2 ne sont pas disponibles.

## Gérer localhost ##
Pour ajouter localhost au Gestionnaire Hyper-V en tant qu’hôte Hyper-V, dans la boîte de dialogue **Sélectionner un ordinateur**, sélectionnez **Ordinateur local**.

![](media/HyperVManager-ConnectToLocalHost.png)

Si une connexion ne peut pas être établie :
*  Assurez-vous que le rôle de la plateforme Hyper-V est activé.  
  Pour savoir si Hyper-V est pris en charge, voir la [section du guide pas à pas relative à la vérification de la compatibilité](../quick_start/walkthrough_compatibility.md).
*  Vérifiez que votre compte d’utilisateur fait partie du groupe Administrateurs Hyper-V.


## Gérer un autre hôte Hyper-V dans le même domaine ##

Pour ajouter un hôte Hyper-V distant au Gestionnaire Hyper-V, dans la boîte de dialogue **Sélectionner un ordinateur**, sélectionnez **Autre ordinateur**, puis entrez le nom d’hôte, le NetBIOS ou le nom de domaine complet de l’hôte distant dans le champ de texte.

![](media/HyperVManager-ConnectToRemoteHost.png)

Pour gérer les hôtes Hyper-V distants, la gestion à distance doit être activée sur l’ordinateur local et l’hôte distant.

Pour cela, accédez à `System Properties -> Remote Management Settings` ou exécutez la commande PowerShell suivante en tant qu’administrateur :  

``` PowerShell
winrm quickconfig
```

Si votre compte d’utilisateur actuel correspond à un compte Administrateur Hyper-V sur l’hôte distant, continuez et appuyez sur **OK** pour établir la connexion.  

> Il s’agit de la seule façon de gérer un hôte distant dans le Gestionnaire Hyper-V dans Windows 8 ou Windows 8.1.


Windows 10 a largement étendu les combinaisons possibles des types de connexions à distance.  
Vous pouvez désormais vous connecter à un hôte distant Windows 10 ou version ultérieure à l’aide du nom d’hôte ou de l’adresse IP.  Le Gestionnaire Hyper-V prend aussi désormais en charge la possibilité d’entrer d’autres informations d’identification de l’utilisateur.  


### Se connecter à l’hôte distant en tant qu’autre utilisateur
> Cela est possible uniquement quand vous vous connectez à un hôte distant Windows 10 ou Windows Server 2016 Technical Preview 3 ou version ultérieure

Dans Windows 10, si vous n’avez pas ouvert de session avec le compte d’utilisateur approprié pour l’hôte distant, vous pouvez vous connecter en tant qu’autre utilisateur avec d’autres informations d’identification.

Pour spécifier les informations d’identification de l’hôte Hyper-V distant, sélectionnez **Se connecter en tant qu’autre utilisateur : ** dans la boîte de dialogue **Sélectionner un ordinateur**, puis sélectionnez **Définir l’utilisateur...**.

![](media/HyperVManager-ConnectToRemoteHostAltCreds.png)


### Se connecter à l’hôte distant à l’aide de l’adresse IP
> Cela est possible uniquement quand vous vous connectez à un hôte distant Windows 10 ou Windows Server 2016 Technical Preview 3 ou version ultérieure

Il est parfois plus facile de se connecter à l’aide de l’adresse IP que du nom d’hôte.  Windows 10 vous permet de le faire.

Pour vous connecter à l’aide de l’adresse IP, entrez l’adresse IP dans le champ de texte **Autre ordinateur**.


## Gérer un hôte Hyper-V en dehors de votre domaine (ou sans domaine) ##
> Cela est possible uniquement quand vous vous connectez à un hôte distant Windows 10 ou Windows Server 2016 Technical Preview 3 ou version ultérieure

Sur l’hôte Hyper-V à gérer, exécutez les commandes suivantes en tant qu’administrateur :

1.  [Enable-PSRemoting](https://technet.microsoft.com/en-us/library/hh849694.aspx)
  * [Enable-PSRemoting](https://technet.microsoft.com/en-us/library/hh849694.aspx) crée les règles de pare-feu nécessaires pour les zones *privées* du réseau. Vous devez activer les règles pour CredSSP et WinRM pour autoriser l’accès sur les zones publiques.
2.  [Enable-WSManCredSSP](https://technet.microsoft.com/en-us/library/hh849872.aspx) -Role server

Sur le PC de gestion, exécutez la commande suivante en tant qu’administrateur :

1. Set-Item WSMan:\localhost\Client\TrustedHosts -value "fqdn-of-hyper-v-host"
  * Vous pouvez aussi permettre à tous les hôtes d’être approuvés pour la gestion avec la commande suivante :
  * Set-Item WSMan:\localhost\Client\TrustedHosts -value * -force
2. [Enable-WSManCredSSP](https://technet.microsoft.com/en-us/library/hh849872.aspx) -Role client -DelegateComputer "fqdn-of-hyper-v-host"
  * Vous pouvez aussi permettre à tous les hôtes d’être approuvés pour la gestion avec la commande suivante :
  * [Enable-WSManCredSSP](https://technet.microsoft.com/en-us/library/hh849872.aspx) -Role client -DelegateComputer *
3. En outre, vous devrez peut-être configurer la stratégie de groupe suivante : ** Configuration de l’ordinateur | Modèles d’administration | Système | Délégation d’informations d’identification | Autoriser la délégation des nouvelles informations d’identification avec l’authentification du serveur NTLM uniquement **
    * Cliquez sur **Activer** et ajoutez *wsman/fqdn-of-hyper-v-host*
    * Vous pouvez aussi permettre à tous les hôtes d’être approuvés pour la gestion en ajoutant _wsman/*_



<!--HONumber=Aug16_HO5-->



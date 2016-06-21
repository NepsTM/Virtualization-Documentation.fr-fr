---
title: &2102857188 Gérer des hôtes Hyper-V distants avec le Gestionnaire Hyper-V
description: Gérer des hôtes Hyper-V distants avec le Gestionnaire Hyper-V
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &270281986 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 2d34e98c-6134-479b-8000-3eb360b8b8a3
---

# Gérer des hôtes Hyper-V distants avec le Gestionnaire Hyper-V

Le Gestionnaire Hyper-V est un outil intégré à Windows pour diagnostiquer et gérer votre hôte Hyper-V local et un petit nombre d’hôtes distants. Cet article décrit les étapes de configuration pour se connecter aux hôtes Hyper-V à l’aide du Gestionnaire Hyper-V dans toutes les configurations prises en charge.

> Le Gestionnaire Hyper-V est disponible dans <g id="2" ctype="x-strong">Programmes et fonctionnalités</g> en tant qu’<g id="4" ctype="x-strong">Outils de gestion Hyper-V</g> sur <g id="6CapsExtId1" ctype="x-link"><g id="6CapsExtId2" ctype="x-linkText">n’importe quel système d’exploitation Windows incluant Hyper-V</g><g id="6CapsExtId3" ctype="x-title"></g></g>. La plateforme Hyper-V n’a pas besoin d’être activée pour gérer les hôtes distants.

Pour vous connecter à un hôte Hyper-V dans le Gestionnaire Hyper-V, assurez-vous que <g id="2" ctype="x-strong">Gestionnaire Hyper-V</g> est sélectionné dans le volet gauche, puis sélectionnez <g id="4" ctype="x-strong">Se connecter au serveur...</g> dans le volet droit.

<g id="1" ctype="x-linkText"></g>

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

Hyper-V est désormais disponible dans Windows 8. Avant Windows 8.1/Windows Server 2012, le Gestionnaire Hyper-V ne gérait que les versions correspondantes d’Hyper-V.

> <g id="1" ctype="x-strong">Remarque :</g> la fonctionnalité du Gestionnaire Hyper-V correspond à la fonctionnalité disponible pour la version que vous gérez. En d’autres termes, si vous gérez un hôte Windows Server 2012 distant à partir de Windows Server 2012 R2, les nouveaux outils du Gestionnaire Hyper-V de Windows Server 2012 R2 ne sont pas disponibles.

## Gérer localhost

Pour ajouter localhost au Gestionnaire Hyper-V en tant qu’hôte Hyper-V, dans la boîte de dialogue <g id="4" ctype="x-strong">Sélectionner un ordinateur</g>, sélectionnez <g id="2" ctype="x-strong">Ordinateur local</g>.

<g id="1" ctype="x-linkText"></g>

Si une connexion ne peut pas être établie :
*  Assurez-vous que le rôle de la plateforme Hyper-V est activé.  
  Pour savoir si Hyper-V est pris en charge, voir la <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">section du guide pas à pas relative à la vérification de la compatibilité</g><g id="2CapsExtId3" ctype="x-title"></g></g>.
*  Vérifiez que votre compte d’utilisateur fait partie du groupe Administrateurs Hyper-V.


## Gérer un autre hôte Hyper-V dans le même domaine

Pour ajouter un hôte Hyper-V distant au Gestionnaire Hyper-V, dans la boîte de dialogue <g id="4" ctype="x-strong">Sélectionner un ordinateur</g>, sélectionnez <g id="2" ctype="x-strong">Autre ordinateur</g>, puis entrez le nom d’hôte, le NetBIOS ou le nom de domaine complet de l’hôte distant dans le champ de texte.

<g id="1" ctype="x-linkText"></g>

Pour gérer les hôtes Hyper-V distants, la gestion à distance doit être activée sur l’ordinateur local et l’hôte distant.

Pour cela, accédez à <g id="2" ctype="x-code">Propriétés système -> Paramètres d’administration à distance</g>, ou exécutez la commande PowerShell suivante en tant qu’administrateur :

``` PowerShell
winrm quickconfig
```

Si votre compte d’utilisateur actuel correspond à un compte Administrateur Hyper-V sur l’hôte distant, continuez et appuyez sur <g id="2" ctype="x-strong">OK</g> pour établir la connexion.

> Il s’agit de la seule façon de gérer un hôte distant dans le Gestionnaire Hyper-V dans Windows 8 ou Windows 8.1.


Windows 10 a largement étendu les combinaisons possibles des types de connexions à distance.  
Vous pouvez désormais vous connecter à un hôte distant Windows 10 ou version ultérieure à l’aide du nom d’hôte ou de l’adresse IP. Le Gestionnaire Hyper-V prend aussi désormais en charge la possibilité d’entrer d’autres informations d’identification de l’utilisateur.


### Se connecter à l’hôte distant en tant qu’autre utilisateur

> Cela est possible uniquement quand vous vous connectez à un hôte distant Windows 10 ou Windows Server 2016 Technical Preview 3 ou version ultérieure

Dans Windows 10, si vous n’avez pas ouvert de session avec le compte d’utilisateur approprié pour l’hôte distant, vous pouvez vous connecter en tant qu’autre utilisateur avec d’autres informations d’identification.

Pour spécifier les informations d’identification de l’hôte Hyper-V distant, sélectionnez <g id="2" ctype="x-strong">Se connecter en tant qu’autre utilisateur : ** dans la boîte de dialogue **Sélectionner un ordinateur</g>, puis sélectionnez <g id="4" ctype="x-strong">Définir l’utilisateur...</g>.

<g id="1" ctype="x-linkText"></g>


### Se connecter à l’hôte distant à l’aide de l’adresse IP

> Cela est possible uniquement quand vous vous connectez à un hôte distant Windows 10 ou Windows Server 2016 Technical Preview 3 ou version ultérieure

Il est parfois plus facile de se connecter à l’aide de l’adresse IP que du nom d’hôte. Windows 10 vous permet de le faire.

Pour vous connecter à l’aide de l’adresse IP, entrez l’adresse IP dans le champ de texte <g id="2" ctype="x-strong">Autre ordinateur</g>.


## Gérer un hôte Hyper-V en dehors de votre domaine (ou sans domaine)

> Cela est possible uniquement quand vous vous connectez à un hôte distant Windows 10 ou Windows Server 2016 Technical Preview 3 ou version ultérieure

Sur l’hôte Hyper-V à gérer, exécutez les commandes suivantes en tant qu’administrateur :

1.  <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Enable-PSRemoting</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Enable-PSRemoting</g><g id="1CapsExtId3" ctype="x-title"></g></g> crée les règles de pare-feu nécessaires pour les zones <g id="3" ctype="x-em">privées</g> du réseau. Vous devez activer les règles pour CredSSP et WinRM pour autoriser l’accès sur les zones publiques.
2. Set-Item WSMan:\localhost\Client\TrustedHosts -value "fqdn-of-managing-pc"
  * Vous pouvez aussi permettre à tous les hôtes d’être approuvés pour la gestion avec la commande suivante :
  * Set-Item WSMan:\localhost\Client\TrustedHosts -value * -force
3. <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Enable-WSManCredSSP</g><g id="1CapsExtId3" ctype="x-title"></g></g> -Role client -DelegateComputer "fqdn-of-managing-pc"
  * Vous pouvez aussi permettre à tous les hôtes d’être approuvés pour la gestion avec la commande suivante :
  * <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Enable-WSManCredSSP</g><g id="1CapsExtId3" ctype="x-title"></g></g> -Role client -DelegateComputer *






<!--HONumber=May16_HO1-->



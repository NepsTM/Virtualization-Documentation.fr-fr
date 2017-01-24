---
title: "Créez vos propres services d’intégration"
description: "Services d’intégration Windows 10."
keywords: "Windows 10, Hyper-V"
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ef8f18c-3d76-4c06-87e4-11d8d4e31aea
translationtype: Human Translation
ms.sourcegitcommit: 54eff4bb74ac9f4dc870d6046654bf918eac9bb5
ms.openlocfilehash: 9e4be610f02e12f48fb88464eb8075b97996d5b2

---

# Créez vos propres services d’intégration

À partir de Windows 10, tout le monde peut rendre un service très similaire aux services d’intégration Hyper-V prédéfinis en utilisant un nouveau canal de communication par socket entre l’hôte Hyper-V et les machines virtuelles qui s’y exécutent.  Grâce à ces sockets Hyper-V, les services peuvent s’exécuter indépendamment de la pile réseau et toutes les données restent sur la même mémoire physique.

Ce document décrit la création d’une application simple basée sur les sockets Hyper-V et explique comment commencer à les utiliser.

[PowerShell Direct](../user-guide/powershell-direct.md) est un exemple d’application (ici un service Windows prédéfini) qui utilise des sockets Hyper-V pour communiquer.

**Système d’exploitation hôte pris en charge**
* Windows 10 build 14290 et ultérieures
* Windows Server Technical Preview 4 et versions ultérieures
* Versions futures (après Server 2016)

**Système d’exploitation invité pris en charge**
* Windows 10
* Windows Server Technical Preview 4 et versions ultérieures
* Versions futures (après Server 2016)
* Invités Linux avec services d’intégration Linux (voir [Machines virtuelles Linux et FreeBSD prises en charge pour Hyper-V sur Windows](https://technet.microsoft.com/library/dn531030(ws.12).aspx))

**Fonctionnalités et limitations**  
* Prend en charge les actions du mode utilisateur ou du mode noyau  
* Flux de données uniquement      
* Aucune mémoire de bloc (ce qui n’est pas optimal pour la sauvegarde/vidéo)   

--------------

## Mise en route
Pour le moment, les sockets Hyper-V sont disponibles en code natif (C/C++).  

Pour écrire une application simple, vous avez besoin des éléments suivants :
* Compilateur C.  Si vous n’en avez pas, voir [Visual Studio Community](https://aka.ms/vs)
* Ordinateur exécutant Hyper-V et une machine virtuelle.  
  * Le système d’exploitation hôte et invité (machine virtuelle) doit être Windows 10, Windows Server Technical Preview 3, ou version ultérieure.
* [SDK Windows 10](http://aka.ms/flightingSDK) installé sur l’hôte Hyper-V

**Détails sur le SDK Windows**

Liens vers le SDK Windows :
* [SDK Windows 10 pour la version préliminaire Windows Insiders](http://aka.ms/flightingSDK)
* [SDK Windows 10](https://dev.windows.com/en-us/downloads/windows-10-sdk)

L’API pour les sockets Hyper-V est disponible depuis Windows 10 build 14290 : le téléchargement de la version d’évaluation correspond à la dernière build d’évaluation Fast Track interne.  
Si vous êtes confronté à un comportement étrange, faites-le nous savoir dans le [forum TechNet](https://social.technet.microsoft.com/Forums/windowsserver/en-US/home "TechNet Forums").  Dans votre billet, indiquez :
* le comportement inattendu ; 
* les numéros de build et de système d’exploitation pour l’hôte, l’invité et le Kit de développement logiciel.  
  
  Le numéro de build du Kit de développement logiciel est visible dans le titre du programme d’installation du Kit de développement logiciel :  
  ![](./media/flightingSDK.png)


## Inscrire une nouvelle application
Pour utiliser des sockets Hyper-V, l’application doit être inscrite auprès du Registre de l’hôte Hyper-V.

En inscrivant le service dans le Registre, vous obtenez :
*  la gestion WMI pour activer, désactiver et répertorier les services disponibles ;
*  l’autorisation de communiquer directement avec les machines virtuelles.

La commande PowerShell suivante inscrit une nouvelle application appelée « HV Socket Demo ».  Elle doit être exécutée en tant qu’administrateur.  Instructions du manuel ci-dessous.

``` PowerShell
$friendlyName = "HV Socket Demo"

# Create a new random GUID and add it to the services list then add the name as a value

$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ((New-Guid).Guid)

$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```

** Emplacement du Registre et informations **  

``` 
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```  
Vous voyez plusieurs GUID à cet emplacement du Registre.  Il s’agit des services que nous fournissons.

Informations dans le Registre pour chaque service :
* `Service GUID`   
    * `ElementName (REG_SZ)` nom convivial du service

Pour inscrire votre propre service, créez une clé de Registre à l’aide de vos propre GUID et nom convivial.

Le nom convivial est associé à votre nouvelle application.  Il apparaît dans les compteurs de performance et d’autres emplacements où un GUID n’est pas approprié.

L’entrée de Registre doit ressembler à ceci :
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName REG_SZ  VM Session Service
    YourGUID\
        ElementName REG_SZ  Your Service Friendly Name
```

> ** Conseil : ** pour générer un GUID dans PowerShell et le copier dans le Presse-papiers, exécutez :  
``` PowerShell
(New-Guid).Guid | clip.exe
```

## Création d’un socket Hyper-V

Dans le cas le plus simple, la définition d’un socket exige une famille d’adresses, un type de connexion et un protocole.

Voici une [définition de socket](
https://msdn.microsoft.com/en-us/library/windows/desktop/ms740506(v=vs.85).aspx
) simple

``` C
SOCKET WSAAPI socket(
  _In_ int af,
  _In_ int type,
  _In_ int protocol
);
``` 

Pour un socket Hyper-V :
* Famille d’adresses : `AF_HYPERV`
* Type : `SOCK_STREAM`
* Protocole : `HV_PROTOCOL_RAW`


Voici un exemple de déclaration/d’instanciation :  
``` C
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);
```


## Liaison à un socket Hyper-V

La fonction bind associe un socket à des informations de connexion.

La définition de fonction est copiée ci-dessous par souci de commodité. Pour en savoir plus sur bind, cliquez [ici](https://msdn.microsoft.com/en-us/library/windows/desktop/ms737550.aspx).

``` C
int bind(
  _In_ SOCKET                s,
  _In_ const struct sockaddr *name,
  _In_ int                   namelen
);
```

Contrairement à l’adresse de socket (sockaddr) pour une famille d’adresses IP standard (`AF_INET`) qui se compose de l’adresse IP de l’ordinateur hôte et d’un numéro de port sur cet hôte, l’adresse de socket pour `AF_HYPERV` utilise les ID de la machine virtuelle et de l’application définis ci-dessus pour établir une connexion. 

Comme les sockets Hyper-V ne dépendent pas, entre autres, d’une pile réseau, de TCP/IP ni de DNS, le point de terminaison de socket exigeait un format autre qu’un nom d’hôte ou qu’une adresse IP qui décrive toujours sans ambiguïté la connexion.

Voici la définition de l’adresse d’un socket Hyper-V :

``` C
struct SOCKADDR_HV
{
     ADDRESS_FAMILY Family;
     USHORT Reserved;
     GUID VmId;
     GUID ServiceId;
};
```

À la place d’une adresse IP ou d’un nom d’hôte, les points de terminaison AF_HYPERV reposent largement sur deux GUID :  
* ID de machine virtuelle : il s’agit de l’ID unique affecté par machine virtuelle.  L’ID d’une machine virtuelle est accessible à l’aide de l’extrait de code PowerShell suivant.  
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* ID de service : GUID, [décrit ci-dessus](#RegisterANewApplication), avec lequel l’application est inscrite dans le Registre de l’hôte Hyper-V.

Il existe également un ensemble de caractères génériques VMID disponibles quand une connexion n’est pas propre à une machine virtuelle spécifique.
 
### Caractères génériques VMID

| Nom | GUID | Description |
|:-----|:-----|:-----|
| HV_GUID_ZERO | 00000000-0000-0000-0000-000000000000 | Les écouteurs doivent établir une liaison à ce VMID pour accepter la connexion de toutes les partitions. |
| HV_GUID_WILDCARD | 00000000-0000-0000-0000-000000000000 | Les écouteurs doivent établir une liaison à ce VMID pour accepter la connexion de toutes les partitions. |
| HV_GUID_BROADCAST | FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF | |  
| HV_GUID_CHILDREN | 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd | Adresse générique pour les enfants. Les écouteurs doivent établir une liaison à ce VMID pour accepter la connexion de ses enfants. |
| HV_GUID_LOOPBACK | e0e16197-dd56-4a10-9195-5ee7a155a838 | Adresse de bouclage. L’utilisation de ce VMID permet de se connecter à la même partition que le connecteur. |
| HV_GUID_PARENT | a42e7cda-d03f-480c-9cc2-a4de20abb878 | Adresse parente. L’utilisation de ce VMID permet de se connecter à la partition parente du connecteur.* |


***HV_GUID_PARENT**  
Le parent d’une machine virtuelle est son hôte.  Le parent d’un conteneur est l’hôte du conteneur.  
Une connexion à partir d’un conteneur exécuté dans une machine virtuelle permet de se connecter à la machine virtuelle qui héberge le conteneur.  
L’écoute sur ce VMID permet d’accepter une connexion depuis :  
(Intérieur des conteneurs) : hôte de conteneur.  
(Intérieur de machine virtuelle : hôte de conteneur/aucun conteneur) : hôte de machine virtuelle.  
(Extérieur de machine virtuelle : hôte de conteneur/aucun conteneur) : aucune prise en charge.

## Commandes de socket prises en charge

Socket()  
Bind()  
Connect()  
Send()  
Listen()  
Accept()  

[API WinSock complète](https://msdn.microsoft.com/en-us/library/windows/desktop/ms741394.aspx)



<!--HONumber=Jan17_HO2-->



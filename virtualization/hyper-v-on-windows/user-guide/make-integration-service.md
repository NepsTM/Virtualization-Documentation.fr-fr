---
title: "Créez vos propres services d’intégration"
description: "Services d’intégration Windows10."
keywords: windows10, hyper-v, HVSocket, AF_HYPERV
author: scooley
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 1ef8f18c-3d76-4c06-87e4-11d8d4e31aea
ms.openlocfilehash: d50648efcaac40d6a60430b44c070717adf31b4d
ms.sourcegitcommit: d5f30aa1bdfb34dd9e1909d73b5bd9f4153d6b46
ms.translationtype: HT
ms.contentlocale: fr-FR
---
# <a name="make-your-own-integration-services"></a>Créez vos propres services d’intégration

Depuis la Mise à jour anniversaire Windows10, tous les utilisateurs peuvent créer des applications qui communiquent entre l’hôte Hyper-V et ses ordinateurs virtuels à l’aide de sockets Hyper-V (socket Windows doté d’une nouvelle famille d’adresses et d’un point de terminaison spécialisé pour cibler des ordinateurs virtuels).  Toute la communication sur les sockets Hyper-V s’exécute sans l’aide de la mise en réseau, et toutes les données restent sur la même mémoire physique.   Les applications utilisant des sockets Hyper-V sont semblables aux services d’intégration de Hyper-V.

Ce document décrit la création d’un programme simple qui repose sur les sockets Hyper-V.

**Système d’exploitation hôte pris en charge**
* Windows10 et versions ultérieures
* Windows Server2016 et versions ultérieures

**Système d’exploitation invité pris en charge**
* Windows10 et versions ultérieures
* Windows Server2016 et versions ultérieures
* Invités Linux avec services d’intégration Linux (voir [Machines virtuelles Linux et FreeBSD prises en charge pour Hyper-V sur Windows](https://technet.microsoft.com/library/dn531030.aspx))

**Fonctionnalités et limitations**  
* Prend en charge les actions du mode utilisateur ou du mode noyau  
* Flux de données uniquement      
* Aucune mémoire de bloc (ce qui n’est pas optimal pour la sauvegarde/vidéo) 

--------------

## <a name="getting-started"></a>Prise en main

Configuration requise:
* CompilateurC/C++.  Si vous n’en avez pas, voir [Visual Studio Community](https://aka.ms/vs)
* [Kit de développement logiciel (SDK) Windows10](https://developer.microsoft.com/windows/downloads/windows-10-sdk) -- préinstallé dans Visual Studio2015 avec Update3 et versions ultérieures.
* Un ordinateur exécutant l’un des systèmes d’exploitation hôtes indiqués ci-dessus avec au moins un ordinateur virtuel. -- cela convient pour le test de votre application.

> **Remarque:** l’API des sockets Hyper-V a été mise à la disposition du public dans Windows10 peu de temps après.  Les applications qui utilisent HVSocket s’exécutent sur n’importe quel hôte ou invité Windows10, mais elles ne peuvent être développées qu’avec un SDK Windows ultérieur à la build14290.  

## <a name="register-a-new-application"></a>Inscrire une nouvelle application
Pour utiliser des sockets Hyper-V, l’application doit être inscrite auprès du Registre de l’hôte Hyper-V.

En inscrivant le service dans le Registre, vous obtenez:
*  la gestion WMI pour activer, désactiver et répertorier les services disponibles;
*  l’autorisation de communiquer directement avec les machines virtuelles.

La commande PowerShell suivante inscrit une nouvelle application appelée «HV Socket Demo».  Elle doit être exécutée en tant qu’administrateur.  Instructions du manuel ci-dessous.

``` PowerShell
$friendlyName = "HV Socket Demo"

# Create a new random GUID.  Add it to the services list
$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ((New-Guid).Guid)

# Set a friendly name 
$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```


**Emplacement du Registre et informations:**  
``` 
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```  
Vous voyez plusieurs GUID à cet emplacement du Registre.  Il s’agit des services que nous fournissons.

Informations dans le Registre pour chaque service:
* `Service GUID`   
    * `ElementName (REG_SZ)` nom convivial du service

Pour inscrire votre propre service, créez une clé de Registre à l’aide de vos propre GUID et nom convivial.

Le nom convivial est associé à votre nouvelle application.  Il apparaît dans les compteurs de performance et d’autres emplacements où un GUID n’est pas approprié.

L’entrée de Registre doit ressembler à ceci:
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName    REG_SZ    VM Session Service
    YourGUID\
        ElementName    REG_SZ    Your Service Friendly Name
```

> **Conseil: ** pour générer un GUID dans PowerShell et le copier dans le Presse-papiers, exécutez:  
``` PowerShell
(New-Guid).Guid | clip.exe
```

## <a name="create-a-hyper-v-socket"></a>Créer un socket Hyper-V

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

Pour un socket Hyper-V:
* Famille d’adresses: `AF_HYPERV`
* Type: `SOCK_STREAM`
* Protocole: `HV_PROTOCOL_RAW`


Voici un exemple de déclaration/d’instanciation:  
``` C
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);
```


## <a name="bind-to-a-hyper-v-socket"></a>Lier à un socket Hyper-V

La fonction bind associe un socket à des informations de connexion.

La définition de fonction est copiée ci-dessous par souci de commodité. Pour en savoir plus sur bind, cliquez [ici](https://msdn.microsoft.com/en-us/library/windows/desktop/ms737550.aspx).

``` C
int bind(
  _In_ SOCKET                s,
  _In_ const struct sockaddr *name,
  _In_ int                   namelen
);
```

Contrairement à l’adresse de socket (sockaddr) pour une famille d’adressesIP standard (`AF_INET`) qui se compose de l’adresseIP de l’ordinateur hôte et d’un numéro de port sur cet hôte, l’adresse de socket pour `AF_HYPERV` utilise lesID de la machine virtuelle et de l’application définis ci-dessus pour établir une connexion. 

Comme les sockets Hyper-V ne dépendent pas, entre autres, d’une pile réseau, de TCP/IP ni de DNS, le point de terminaison de socket exigeait un format autre qu’un nom d’hôte ou qu’une adresseIP qui décrive toujours sans ambiguïté la connexion.

Voici la définition de l’adresse d’un socket Hyper-V:

``` C
struct SOCKADDR_HV
{
     ADDRESS_FAMILY Family;
     USHORT Reserved;
     GUID VmId;
     GUID ServiceId;
};
```

À la place d’une adresseIP ou d’un nom d’hôte, les points de terminaison AF_HYPERV reposent largement sur deux GUID:  
* IDde machine virtuelle: il s’agit de l’ID unique affecté par machine virtuelle.  L’ID d’une machine virtuelle est accessible à l’aide de l’extrait de code PowerShell suivant.  
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* IDde service: GUID, [décrit ci-dessus](#RegisterANewApplication), avec lequel l’application est inscrite dans le Registre de l’hôte Hyper-V.

Il existe également un ensemble de caractères génériques VMID disponibles quand une connexion n’est pas propre à une machine virtuelle spécifique.
 
### <a name="vmid-wildcards"></a>Caractères génériques VMID

| Nom | GUID | Description |
|:-----|:-----|:-----|
| HV_GUID_ZERO | 00000000-0000-0000-0000-000000000000 | Les écouteurs doivent établir une liaison à ce VMID pour accepter la connexion de toutes les partitions. |
| HV_GUID_WILDCARD | 00000000-0000-0000-0000-000000000000 | Les écouteurs doivent établir une liaison à ce VMID pour accepter la connexion de toutes les partitions. |
| HV_GUID_BROADCAST | FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF | |  
| HV_GUID_CHILDREN | 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd | Adresse générique pour les enfants. Les écouteurs doivent établir une liaison à ce VMID pour accepter la connexion de ses enfants. |
| HV_GUID_LOOPBACK | e0e16197-dd56-4a10-9195-5ee7a155a838 | Adresse de bouclage. L’utilisation de ce VMID permet de se connecter à la même partition que le connecteur. |
| HV_GUID_PARENT | a42e7cda-d03f-480c-9cc2-a4de20abb878 | Adresse parente. L’utilisation de ce VMID permet de se connecter à la partition parente du connecteur.* |


\* `HV_GUID_PARENT`  
Le parent d’une machine virtuelle est son hôte.  Le parent d’un conteneur est l’hôte du conteneur.  
Une connexion à partir d’un conteneur exécuté dans une machine virtuelle permet de se connecter à la machine virtuelle qui héberge le conteneur.  
L’écoute sur ce VMID permet d’accepter une connexion depuis:  
(Intérieur des conteneurs): hôte de conteneur.  
(Intérieur de machine virtuelle: hôte de conteneur/aucun conteneur): hôte de machine virtuelle.  
(Extérieur de machine virtuelle: hôte de conteneur/aucun conteneur): aucune prise en charge.

## <a name="supported-socket-commands"></a>Commandes de socket prises en charge

Socket()  
Bind()  
Connect()  
Send()  
Listen()  
Accept()  

## <a name="useful-links"></a>Liens utiles
[API WinSock complète](https://msdn.microsoft.com/en-us/library/windows/desktop/ms741394.aspx)

[Informations de référence sur les services d’intégration Hyper-V](../reference/integration-services.md)
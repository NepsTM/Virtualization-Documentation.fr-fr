---
title: Configurer un réseau NAT
description: Configurer un réseau NAT
keywords: Windows10, Hyper-V
author: jmesser81
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1f8a691c-ca75-42da-8ad8-a35611ad70ec
ms.openlocfilehash: 0c365b9351ee09c946e1711f3a3a5e82eb71c785
ms.sourcegitcommit: 4090d158dd3573ea90799de5b014c131a206b000
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2018
ms.locfileid: "6121619"
---
# <a name="set-up-a-nat-network"></a>Configurer un réseau NAT

Hyper-V sur Windows10 permet la traduction d’adresses réseau (NAT) natives pour un réseau virtuel.

Ce guide vous aidera tout au long des processus suivants:
* Création d’un réseau NAT
* Connexion d’une machine virtuelle existante à votre nouveau réseau
* Vérification que la machine virtuelle est correctement connectée

Configuration requise:
* Mise à jour anniversaire Windows10 ou ultérieure
* Hyper-V est activé (instructions [ici](../quick-start/enable-hyper-v.md))

> **Remarque:** pour le moment, vous pouvez créer un seul réseau NAT par hôte. Pour plus d’informations sur l’implémentation, les fonctionnalités et les limitations de NAT Windows (WinNAT), reportez-vous au billet de blog [Fonctionnalités et limitations de WinNAT](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations/)

## <a name="nat-overview"></a>Vue d’ensemble de NAT
NAT permet à une machine virtuelle d’accéder à des ressources réseau à l’aide de l’adresseIP de l’ordinateur hôte et d’un port via un commutateur virtuel Hyper-V interne.

La traduction d’adresses réseau (NAT) est un mode de mise en réseau conçu pour conserver des adressesIP en mappant une adresseIP et un port externes à un ensemble beaucoup plus vaste d’adressesIP internes.  Fondamentalement, NAT utilise une table de flux pour acheminer le trafic à partir d’une adresseIP (hôte) externe et d’un numéro de port vers l’adresseIP interne appropriée associée à un point de terminaison sur le réseau (machine virtuelle, ordinateur, conteneur, etc.)

De plus, NAT autorise que plusieurs machines virtuelles hébergent des applications qui exigent des ports de communication (internes) identiques en les mappant à des ports externes uniques.

Pour toutes ces raisons, la mise en réseau NAT est très répandue pour la technologie de conteneur (voir [Mise en réseau de conteneur](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/management/container_networking)).


## <a name="create-a-nat-virtual-network"></a>Créer un réseau virtuel NAT
Examinons la configuration d’un nouveau réseau NAT.

1.  Ouvrez une console PowerShell en tant qu’administrateur.  

2. Créez un commutateur interne.

  ``` PowerShell
  New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
  ```

3. Recherchez l’index d’interface du commutateur virtuel que vous venez de créer.

    Pour trouver l’index d’interface, exécutez `Get-NetAdapter`

    Votre sortie doit ressembler à ceci:

    ```
    PS C:\> Get-NetAdapter

    Name                  InterfaceDescription               ifIndex Status       MacAddress           LinkSpeed
    ----                  --------------------               ------- ------       ----------           ---------
    vEthernet (intSwitch) Hyper-V Virtual Ethernet Adapter        24 Up           00-15-5D-00-6A-01      10 Gbps
    Wi-Fi                 Marvell AVASTAR Wireless-AC Net...      18 Up           98-5F-D3-34-0C-D3     300 Mbps
    Bluetooth Network ... Bluetooth Device (Personal Area...      21 Disconnected 98-5F-D3-34-0C-D4       3 Mbps

    ```

    Le commutateur interne a un nom semblable à `vEthernet (SwitchName)` et la description d’interface `Hyper-V Virtual Ethernet Adapter`. Notez son `ifIndex` afin de l’utiliser dans l’étape suivante.

4. Configurez la passerelle NAT à l’aide de [New-NetIPAddress](https://docs.microsoft.com/powershell/module/nettcpip/New-NetIPAddress).  

  Voici la commande générique:
  ``` PowerShell
  New-NetIPAddress -IPAddress <NAT Gateway IP> -PrefixLength <NAT Subnet Prefix Length> -InterfaceIndex <ifIndex>
  ```

  Pour configurer la passerelle, vous aurez besoin de certaines informations concernant votre réseau:  
  * **IPAddress**: l’adresseIP de la passerelle NAT spécifie l’adresse IPv4 ou IPv6 à utiliser en tant qu’adresseIP de la passerelle NAT.  
    La forme générique est a.b.c.1 (par exemple, 172.16.0.1).  Même si la position finale ne doit pas obligatoirement être.1, c’est généralement le cas (en fonction de la longueur du préfixe).

    Une adresseIP de passerelle courante est 192.168.0.1  

  * **PrefixLength**: la longueur du préfixe de sous-réseau NAT définit la taille du sous-réseau local NAT (masque de sous-réseau).
    La longueur du préfixe de sous-réseau est une valeur entière comprise entre0 et32.

    0 mappe la totalité d’Internet; 32 permet une seule adresseIP mappée.  Les valeurs courantes sont comprises entre24 et12 en fonction du nombre d’adressesIP attachées à NAT.

    24 est une valeur PrefixLength courante: il s’agit d’un masque de sous-réseau 255.255.255.0

  * **InterfaceIndex** --ifIndex est l’index d’interface du commutateur virtuel défini à l’étape précédente.

  Exécutez la commande suivante pour créer une passerelle NAT:

  ``` PowerShell
  New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
  ```

5. Configurez le réseau NAT à l’aide de [New-NetNat](https://docs.microsoft.com/powershell/module/netnat/New-NetNat).  

  Voici la commande générique:

  ``` PowerShell
  New-NetNat -Name <NATOutsideName> -InternalIPInterfaceAddressPrefix <NAT subnet prefix>
  ```

  Pour configurer la passerelle, vous devez fournir des informations relatives au réseau et à la passerelle NAT:  
  * **Name**: NATOutsideName décrit le nom du réseau NAT.  Cette valeur permet de supprimer le réseau NAT.

  * **InternalIPInterfaceAddressPrefix**: le préfixe de sous-réseau NAT décrit à la fois le préfixeIP de la passerelle NAT ci-dessus et la longueur du préfixe de sous-réseau NAT ci-dessus.

    La forme générique est a.b.c.0/Longueur du préfixe de sous-réseau NAT

    Conformément à ce qui précède, pour cet exemple, nous utiliserons 192.168.0.0/24

  Pour notre exemple, exécutez la commande suivante pour configurer le réseau NAT:

  ``` PowerShell
  New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.0.0/24
  ```

Félicitations!  Vous avez maintenant un réseau NAT virtuel!  Pour ajouter une machine virtuelle, pour le réseau NAT, suivez [ces instructions](#connect-a-virtual-machine).

## <a name="connect-a-virtual-machine"></a>Connecter une machine virtuelle

Pour connecter une machine virtuelle à votre nouveau réseau NAT, connectez le commutateur interne que vous avez créé à la première étape de la section de [configuration d’un réseau NAT](#create-a-nat-virtual-network) à votre machine virtuelle à l’aide du menu Paramètres de la machine virtuelle.

Étant donné que WinNAT par lui-même ne peut pas allouer ni affecter des adressesIP à un point de terminaison (par exemple, une machine virtuelle), vous devez le faire manuellement à partir de la machine virtuelle elle-même, autrement dit définir une adresseIP dans la plage de préfixe interne NAT, définir une adresseIP de passerelle par défaut et définir des informations sur le serveur DNS. Vous devez être attentif quand le point de terminaison est associé à un conteneur. Dans ce cas, le service HNS (Host Network Service) alloue et utilise le service HCS (Host Compute Service) pour affecter l’adresseIP, l’adresseIP de passerelle et les informations DNS directement au conteneur.


## <a name="configuration-example-attaching-vms-and-containers-to-a-nat-network"></a>Exemple de configuration: Association de machines virtuelles et de conteneurs à un réseau NAT

_Si vous devez associer plusieurs machines virtuelles et conteneurs à un seul réseau NAT, vous devrez vous assurer que le préfixe de sous-réseau interne NAT est suffisamment important pour englober les plagesIP affectées par différents services ou applications (par exemple, Docker pour Windows et conteneur Windows – HNS). Cela nécessite une affectation d’adressesIP au niveau de l’application et une configuration réseau ou une configuration manuelle à exécuter par un administrateur et qui ne doit pas réutiliser les affectations d’adressesIP existantes sur le même hôte._

### <a name="docker-for-windows-linux-vm-and-windows-containers"></a>Docker pour Windows (machine virtuelle Linux) et conteneurs Windows
La solution ci-dessous permet à la fois à Docker pour Windows (machine virtuelle Linux exécutant des conteneurs Linux) et aux conteneurs Windows de partager la même instance WinNAT à l’aide de commutateurs virtuels internes distincts. La connectivité entre les conteneurs Windows et Linux fonctionne.

L’utilisateur a connecté des machines virtuelles à un réseau NAT via un commutateur virtuel interne nommé «VMNAT» et veut maintenant installer la fonctionnalité de conteneur Windows avec le moteur Docker
```
PS C:\> Get-NetNat “VMNAT”| Remove-NetNat (this will remove the NAT but keep the internal vSwitch).
Install Windows Container Feature
DO NOT START Docker Service (daemon)
Edit the arguments passed to the docker daemon (dockerd) by adding –fixed-cidr=<container prefix> parameter. This tells docker to create a default nat network with the IP subnet <container prefix> (e.g. 192.168.1.0/24) so that HNS can allocate IPs from this prefix.
PS C:\> Start-Service Docker; Stop-Service Docker
PS C:\> Get-NetNat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> Start-Service docker
```
Docker/HNS affectera des adresses IP aux conteneurs Windows et administrateur sera affecter des adresses IP aux machines virtuelles à partir de l’ensemble des deux.

L’utilisateur a installé la fonctionnalité de conteneur Windows avec le moteur Docker en cours d’exécution et veut maintenant connecter les machines virtuelles au réseau NAT
```
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
PS C:\> Get-NetNat | Remove-NetNat (this will remove the NAT but keep the internal vSwitch)
Edit the arguments passed to the docker daemon (dockerd) by adding -b “none” option to the end of docker daemon (dockerd) command to tell docker not to create a default NAT network.
PS C:\> New-ContainerNetwork –name nat –Mode NAT –subnetprefix <container prefix> (create a new NAT and internal vSwitch – HNS will allocate IPs to container endpoints attached to this network from the <container prefix>)
PS C:\> Get-Netnat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> New-VirtualSwitch -Type internal (attach VMs to this new vSwitch)
PS C:\> Start-Service docker
```
Docker/HNS affectera des adresses IP aux conteneurs Windows et administrateur sera affecter des adresses IP aux machines virtuelles à partir de l’ensemble des deux.

Au final, vous devez disposer de deux commutateurs de machine virtuelle internes et d’un réseau NAT partagé entre eux.

## <a name="multiple-applications-using-the-same-nat"></a>Plusieurs applications utilisant la même NAT

Certains scénarios exigent que plusieurs applications ou services utilisent la même NAT. Dans ce cas, le workflow suivant doit être suivi afin que plusieurs applications/services puissent utiliser un préfixe de sous-réseau interne NAT plus grand

**_À titre d’exemple, nous allons décrire en détail la machine virtuelle Windows Docker4/Docker Beta/Linux qui coexiste avec la fonctionnalité de conteneur Windows sur le même hôte. Ce flux de travail est susceptible d’être modifié_**

1. C:\> net stop docker
2. Arrêter la machine virtuelle Docker4Windows MobyLinux
3. PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
4. PS C:\> Get-NetNat | Remove-NetNat  
   *Supprime tous les réseaux de conteneur existants (par exemple, supprime vSwitch, supprime NetNat, nettoie)*  

5. New-ContainerNetwork -Name nat -Mode NAT –subnetprefix 10.0.76.0/24 (ce sous-réseau sera utilisé pour la fonctionnalité de conteneurs Windows) *Crée un commutateur virtuel interne nommé nat*  
   *Crée un réseau NAT nommé «nat» avec le préfixeIP 10.0.76.0/24*  

6. Remove-NetNAT  
   *Supprime les réseaux NAT DockerNAT et nat (conserve les commutateurs virtuels internes)*  

7. New-NetNat -Name DockerNAT -InternalIPInterfaceAddressPrefix 10.0.0.0/17 (cela crée un réseau NAT plus large à partager entre D4W et les conteneurs)  
   *Crée un réseau NAT nommé DockerNAT avec un plus grand préfixe 10.0.0.0/17*  

8. Run Docker4Windows (MobyLinux.ps1)  
   *Crée un commutateur virtuel interne DockerNAT*  
   *Crée un réseau NAT nommé «DockerNAT» avec un préfixeIP 10.0.75.0/24*  

9. Net start docker  
   *Docker utilisera par défaut le réseau NAT défini par l’utilisateur pour se connecter à des conteneurs Windows*  

Au final, vous devez avoir deux commutateurs virtuels internes: l’un nommé DockerNAT et l’autre nommé nat. Vous avez un seul réseau NAT (10.0.0.0/17), ce que vous avez vérifié en exécutant la commande Get-NetNat. Les adressesIP des conteneurs Windows seront affectées par le service de réseau hôte (HNS, Host Network Service) Windows à partir du sous-réseau 10.0.76.0/24. Basées sur le script MobyLinux.ps1 existant, les adressesIP pour Windows Docker4 seront affectées à partir du sous-réseau 10.0.75.0/24.


## <a name="troubleshooting"></a>Résolution des problèmes

### <a name="multiple-nat-networks-are-not-supported"></a>Les réseaux NAT multiples ne sont pas pris en charge  
Ce guide suppose qu’il n’y a pas d’autre NAT sur l’hôte. Toutefois, les applications ou services qui nécessitent l’utilisation d’un NAT peuvent en créer un au moment de l’installation. Étant donné que Windows (WinNAT) ne prend en charge qu’un seul préfixe de sous-réseau NAT interne, si vous essayez de créer plusieurs NAT, le système est placé dans un état inconnu.

Pour voir si c’est le problème, vérifiez que vous avez un seul NAT:
``` PowerShell
Get-NetNat
```

Si un NAT existe déjà, supprimez-le.
``` PowerShell
Get-NetNat | Remove-NetNat
```
Vérifiez que vous avez uniquement un commutateur de machine virtuelle «interne» pour l’application ou la fonctionnalité (par exemple, des conteneurs Windows). Enregistrez le nom du commutateur virtuel
``` PowerShell
Get-VMSwitch
```

Vérifiez s’il existe des adressesIP privées (par exemple, l’adresseIP de la passerelle par défaut NAT, généralement *.1) de l’ancien NAT toujours affectées à un adaptateur
``` PowerShell
Get-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)"
```

Si une ancienne adresseIP privée est en cours d’utilisation, supprimez-la
``` PowerShell
Remove-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)" -IPAddress <IPAddress>
```

**Suppression des NAT multiples**  
Nous avons vu des rapports concernant plusieurs réseaux NAT créés par inadvertance. Cela est dû à un bogue dans les builds récentes (notamment Windows Server2016 Technical Preview5 et Windows10 Insider Preview). Si vous voyez plusieurs réseaux NAT après avoir exécuté docker network ls ou Get-ContainerNetwork, exécutez ce qui suit à partir d’une invite PowerShell avec élévation des privilèges:

```
PS> $KeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
PS> $keys = get-childitem $KeyPath
PS> foreach($key in $keys)
PS> {
PS>    if ($key.GetValue("FriendlyName") -eq 'nat')
PS>    {
PS>       $newKeyPath = $KeyPath+"\"+$key.PSChildName
PS>       Remove-Item -Path $newKeyPath -Recurse
PS>    }
PS> }
PS> remove-netnat -Confirm:$false
PS> Get-ContainerNetwork | Remove-ContainerNetwork
PS> Get-VmSwitch -Name nat | Remove-VmSwitch (_failure is expected_)
PS> Stop-Service docker
PS> Set-Service docker -StartupType Disabled
Reboot Host
PS> Get-NetNat | Remove-NetNat
PS> Set-Service docker -StartupType automaticac
PS> Start-Service docker 
```

Consultez ce [guide d’installation concernant l’utilisation du même NAT par plusieurs applications](#multiple-applications-using-the-same-nat) pour recréer votre environnement NAT, si nécessaire. 

## <a name="references"></a>Références
En savoir plus sur les [réseaux NAT](https://en.wikipedia.org/wiki/Network_address_translation)

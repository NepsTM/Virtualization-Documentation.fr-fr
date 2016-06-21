---
title: Configurer un réseau NAT
description: Configurer un réseau NAT
keywords: windows 10, hyper-v
author: jmesser81
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1f8a691c-ca75-42da-8ad8-a35611ad70ec
---

# Configurer un réseau NAT

Hyper-V sur Windows 10 permet la traduction d’adresses réseau (NAT) natives pour un réseau virtuel.

Ce guide vous aidera tout au long des processus suivants :
* Création d’un réseau NAT
* Connexion d’une machine virtuelle existante à votre nouveau réseau
* Vérification que la machine virtuelle est correctement connectée

Configuration requise :
* Windows build 14295 ou ultérieures
* Le rôle Hyper-V est activé (instructions [ici](../quick_start/walkthrough_create_vm.md))

> **Remarque :** Actuellement, Hyper-V vous permet uniquement de créer un réseau NAT.

## Vue d’ensemble de NAT
NAT permet à une machine virtuelle d’accéder à des ressources réseau à l’aide de l’adresse IP de l’ordinateur hôte et d’un port.

La traduction d’adresses réseau (NAT) est un mode de mise en réseau conçu pour conserver des adresses IP en mappant une adresse IP et un port externes à un ensemble beaucoup plus vaste d’adresses IP internes.  Fondamentalement, un commutateur NAT utilise une table de mappage NAT pour acheminer le trafic à partir d’une adresse IP et d’un numéro de port vers l’adresse IP interne appropriée associée à un appareil sur le réseau (machine virtuelle, ordinateur, conteneur, etc.).

De plus, NAT autorise que plusieurs machines virtuelles hébergent des applications qui exigent des ports de communication (internes) identiques en les mappant à des ports externes uniques.

Pour toutes ces raisons, la mise en réseau NAT est très répandue pour la technologie de conteneur (voir [Mise en réseau de conteneur](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/management/container_networking)).


## Créer un réseau virtuel NAT
Examinons la configuration d’un nouveau réseau NAT.

1.  Ouvrez une console PowerShell en tant qu’administrateur.  

2. Créer un commutateur interne  

  ``` PowerShell
  New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
  ```

3. Configurez la passerelle NAT à l’aide de [New-NetIPAddress](https://technet.microsoft.com/en-us/library/hh826150.aspx).  

  Voici la commande générique :
  ``` PowerShell
  New-NetIPAddress -IPAddress <NAT Gateway IP> -PrefixLength <NAT Subnet Prefix Length> -InterfaceIndex <ifIndex>
  ```

  Pour configurer la passerelle, vous aurez besoin de certaines informations concernant votre réseau :  
  * **IPAddress** : l’adresse IP de la passerelle NAT spécifie l’adresse IPv4 ou IPv6 à utiliser en tant qu’adresse IP de la passerelle NAT.  
    La forme générique est a.b.c.1 (par exemple, 172.16.0.1).  Même si la position finale ne doit pas obligatoirement être .1, c’est généralement le cas (en fonction de la longueur du préfixe).

    Une adresse IP de passerelle courante est 192.168.0.1  

  * **PrefixLength** : la longueur du préfixe de sous-réseau NAT définit la taille du sous-réseau local NAT (masque de sous-réseau).
    La longueur du préfixe de sous-réseau est une valeur entière comprise entre 0 et 32.

    0 mappe la totalité d’Internet ; 32 permet une seule adresse IP mappée.  Les valeurs courantes sont comprises entre 24 et 12 en fonction du nombre d’adresses IP attachées à NAT.

    24 est une valeur PrefixLength courante : il s’agit d’un masque de sous-réseau 255.255.255.0

  * **InterfaceIndex** : ifIndex est l’index d’interface du commutateur virtuel créé précédemment.

    Pour trouver l’index d’interface, exécutez `Get-NetAdapter`

    Votre sortie doit ressembler à ceci :

    ```
    PS C:\> Get-NetAdapter

    Name                  InterfaceDescription               ifIndex Status       MacAddress           LinkSpeed
    ----                  --------------------               ------- ------       ----------           ---------
    vEthernet (intSwitch) Hyper-V Virtual Ethernet Adapter        24 Up           00-15-5D-00-6A-01      10 Gbps
    Wi-Fi                 Marvell AVASTAR Wireless-AC Net...      18 Up           98-5F-D3-34-0C-D3     300 Mbps
    Bluetooth Network ... Bluetooth Device (Personal Area...      21 Disconnected 98-5F-D3-34-0C-D4       3 Mbps

    ```

    Le commutateur interne a un nom semblable à `vEthernet (SwitchName)` et la description d’interface `Hyper-V Virtual Ethernet Adapter`.

  Exécutez la commande suivante pour créer une passerelle NAT :

  ``` PowerShell
  New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
  ```

4. Configurez le réseau NAT à l’aide de [New-NetNat](https://technet.microsoft.com/en-us/library/dn283361(v=wps.630).aspx).  

  Voici la commande générique :

  ``` PowerShell
  New-NetNat -Name <NATOutsideName> -InternalIPInterfaceAddressPrefix <NAT subnet prefix>
  ```

  Pour configurer la passerelle, vous devez fournir des informations relatives au réseau et à la passerelle NAT :  
  * **Name** : NATOutsideName décrit le nom du réseau NAT.  Cette valeur permet de supprimer le réseau NAT.

  * **InternalIPInterfaceAddressPrefix** : le préfixe de sous-réseau NAT décrit à la fois le préfixe IP de la passerelle NAT ci-dessus et la longueur du préfixe de sous-réseau NAT ci-dessus.

    La forme générique est a.b.c.0/Longueur du préfixe de sous-réseau NAT

    Conformément à ce qui précède, pour cet exemple, nous utiliserons 192.168.0.0/24

  Pour notre exemple, exécutez la commande suivante pour configurer le réseau NAT :

  ``` PowerShell
  New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.0.0/24
  ```

Félicitations !  Vous avez maintenant un réseau NAT virtuel !  Pour ajouter une machine virtuelle, pour le réseau NAT, suivez [ces instructions](setup_nat_network.md#connect-a-virtual-machine).

## Connecter une machine virtuelle

Pour connecter une machine virtuelle à votre nouveau réseau NAT, connectez le commutateur interne que vous avez créé à la première étape de la section de [configuration d’un réseau NAT](setup_nat_network.md#create-a-nat-virtual-network) à votre machine virtuelle à l’aide du menu Paramètres de la machine virtuelle.


## Résolution des problèmes

Ce flux de travail suppose qu’il n’y a aucune autre NAT sur l’hôte. Cependant, plusieurs applications ou services nécessitent parfois l’utilisation d’une NAT. Étant donné que Windows (WinNAT) ne prend en charge qu’un seul préfixe de sous-réseau NAT interne, si vous essayez de créer plusieurs NAT, le système est placé dans un état inconnu.

### Étapes de résolution des problèmes
1. Vérifiez que vous avez une seule NAT

  ``` PowerShell
  Get-NetNat
  ```
2. Si une NAT existe déjà, supprimez-la

  ``` PowerShell
  Get-NetNat | Remove-NetNat
  ```

3. Vérifiez que vous avez un seul commutateur virtuel (vmSwitch) « interne » pour la NAT. Enregistrez le nom du commutateur virtuel pour l’étape 4

  ``` PowerShell
  Get-VMSwitch
  ```

4. Vérifiez s’il existe des adresses IP privées (par exemple, l’adresse IP de la passerelle NAT par défaut ; généralement *.1) de l’ancienne NAT toujours affectée à un adaptateur

  ``` PowerShell
  Get-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)"
  ```

5. Si une ancienne adresse IP privée est en cours d’utilisation, supprimez-la  
   ``` PowerShell
  Remove-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)" -IPAddress <IPAddress>
  ```

## Plusieurs applications utilisant la même NAT

Certains scénarios exigent que plusieurs applications ou services utilisent la même NAT. Dans ce cas, le workflow suivant doit être suivi afin que plusieurs applications/services puissent utiliser un préfixe de sous-réseau interne NAT plus grand

**_À titre d’exemple, nous allons décrire en détail la machine virtuelle Windows Docker 4/Docker Beta/Linux qui coexiste avec la fonctionnalité de conteneur Windows sur le même hôte. Ce workflow est susceptible d’être modifié_**

1. C:\> net stop docker
2. Stop Docker4Windows MobyLinux VM
3. PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
4. PS C:\> Get-NetNat | Remove-NetNat  
   *Supprime tous les réseaux de conteneur existants (par exemple, supprime vSwitch, supprime NetNat, nettoie)*  

5. New-ContainerNetwork -Name nat -Mode NAT –subnetprefix 10.0.76.0/24 (ce sous-réseau sera utilisé pour la fonctionnalité de conteneurs Windows) *Crée un commutateur virtuel interne nommé nat*  
   *Crée un réseau NAT nommé « nat » avec le préfixe IP 10.0.76.0/24*  

6. Remove-NetNAT  
   *Supprime les réseaux NAT DockerNAT et nat (conserve les commutateurs virtuels internes)*  

7. New-NetNat -Name DockerNAT -InternalIPInterfaceAddressPrefix 10.0.0.0/17 (cela crée un réseau NAT plus large à partager entre D4W et les conteneurs)  
   *Crée un réseau NAT nommé DockerNAT avec un plus grand préfixe 10.0.0.0/17*  

8. Run Docker4Windows (MobyLinux.ps1)  
   *Crée un commutateur virtuel interne DockerNAT*  
   *Crée un réseau NAT nommé « DockerNAT » avec un préfixe IP 10.0.75.0/24*  

9. Net start docker  
   *Docker utilisera par défaut le réseau NAT défini par l’utilisateur pour se connecter à des conteneurs Windows*  

Au final, vous devez avoir deux commutateurs virtuels internes : l’un nommé DockerNAT et l’autre nommé nat. Vous avez un seul réseau NAT (10.0.0.0/17), ce que vous avez vérifié en exécutant la commande Get-NetNat. Les adresses IP des conteneurs Windows seront affectées par le service de réseau hôte (HNS, Host Network Service) Windows à partir du sous-réseau 10.0.76.0/24. Basées sur le script MobyLinux.ps1 existant, les adresses IP pour Windows Docker 4 seront affectées à partir du sous-réseau 10.0.75.0/24.


## Références
En savoir plus sur les [réseaux NAT](https://en.wikipedia.org/wiki/Network_address_translation)


<!--HONumber=May16_HO5-->



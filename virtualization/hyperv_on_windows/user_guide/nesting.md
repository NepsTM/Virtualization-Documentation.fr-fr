---
title: "Virtualisation imbriquée"
description: "Virtualisation imbriquée"
keywords: "Windows 10, Hyper-V"
author: theodthompson
manager: timlt
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
translationtype: Human Translation
ms.sourcegitcommit: 4eb3e733990ca86f28620e23bf408ff71b879173
ms.openlocfilehash: c94b7f2c9c90eaf2834a1d0c54fb1f9b2a8f9669

---

# Exécuter Hyper-V dans une machine virtuelle avec la virtualisation imbriquée

La virtualisation imbriquée est une fonctionnalité qui vous permet d’exécuter Hyper-V à l’intérieur d’une machine virtuelle Hyper-V. En d’autres termes, avec la virtualisation imbriquée, un hôte Hyper-V peut être virtualisé. Certains cas d’utilisation de la virtualisation imbriquée consistent à exécuter un conteneur Hyper-V dans un hôte de conteneur virtualisé, configurer un laboratoire Hyper-V dans un environnement virtualisé ou tester des scénarios sur plusieurs machines sans besoin de matériel. Ce document décrit en détail la configuration matérielle et logicielle requise, les étapes de configuration et les limitations. 

## Conditions préalables

- Un hôte Hyper-V exécutant Windows Server 2016 ou la mise à jour anniversaire Windows 10.
- Une machine virtuelle Hyper-V exécutant Windows Server 2016 ou la mise à jour anniversaire Windows 10.
- Une machine virtuelle Hyper-V avec une configuration 8.0 ou version ultérieure.
- Processeur Intel avec la technologie VT-x et EPT.

## Configurer la virtualisation imbriquée

1. Créer une machine virtuelle. Consultez la configuration requise ci-dessus pour les versions de système d’exploitation et les machines virtuelles.
2. Pendant que la machine virtuelle est à l’état DÉSACTIVÉ, exécutez la commande suivante sur l’hôte Hyper-V physique. Cela permet d’activer la virtualisation imbriquée de la machine virtuelle.
```none
    Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. Démarrez l'ordinateur virtuel.
4. Installez Hyper-V sur la machine virtuelle, comme vous le feriez sur un serveur physique. Pour plus d’informations sur l’installation d’Hyper-V, consultez [Installer Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install).

## Désactiver la virtualisation imbriquée
Vous pouvez désactiver la virtualisation imbriquée d’une machine virtuelle à l’arrêt à l’aide de la commande PowerShell suivante :
```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## Mémoire dynamique et redimensionnement de la mémoire d’exécution
Hyper-V est en cours d’exécution dans une machine virtuelle, la machine virtuelle doit être désactivée pour ajuster sa mémoire. Cela signifie que même si la mémoire dynamique est activée, la quantité de mémoire ne varie pas. Pour les machines virtuelles dont la mémoire dynamique n’est pas activée, toute tentative d’ajustement de la quantité de mémoire en fonctionnement échouera. 

Notez que la simple activation de la virtualisation imbriquée n’a aucun effet sur la mémoire dynamique ou le redimensionnement de la mémoire runtime. L’incompatibilité se produit uniquement lorsque Hyper-V s’exécute dans la machine virtuelle Hyper-V.

## Options de mise en réseau
Il existe deux options pour la mise en réseau des machines virtuelles imbriquées : l’usurpation des adresses MAC et le mode NAT.

### Usurpation des adresses MAC
Pour que les paquets réseau puissent être acheminés via deux commutateurs virtuels, l’usurpation des adresses MAC doit être activée sur le premier niveau du commutateur virtuel. Pour cela, exécutez la commande PowerShell suivante.

```none
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```
### Traduction d’adresses réseau
La deuxième option s’appuie sur la traduction d’adresses réseau (NAT). Cette approche est idéale pour les cas où l’usurpation des adresses MAC n’est pas possible, comme dans un environnement de cloud public.

Tout d’abord, un commutateur NAT virtuel doit être créé dans la machine virtuelle hôte (machine virtuelle « intermédiaire »). Notez que les adresses IP ne sont qu’un exemple et peuvent varier entre les environnements :
```none
new-vmswitch -name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```
Affectez ensuite une adresse IP à la carte réseau :
```none
get-netadapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```
Une adresse IP et une passerelle doivent être affectées à chaque machine virtuelle imbriquée. Notez que l’adresse IP de la passerelle doit pointer vers la carte NAT de l’étape précédente. Vous pouvez également affecter un serveur DNS :
```none
get-netadapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## Applications de virtualisation tierces
Les applications de virtualisation autres que Hyper-V ne sont pas prises en charge sur les machines virtuelles Hyper-V et sont susceptibles d’échouer. Cela inclut tout logiciel qui requiert des extensions matérielles pour la virtualisation.



<!--HONumber=Sep16_HO3-->



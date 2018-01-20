---
title: "Virtualisation imbriquée"
description: "Virtualisation imbriquée"
keywords: Windows10, Hyper-V
author: johncslack
ms.date: 12/18/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.openlocfilehash: 82c8ba6b09b6d1bfde7217eeaf16cfb7967d4f62
ms.sourcegitcommit: 59541f11d481d8df341597bd73ce7fac14f442ee
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/05/2018
---
# <a name="run-hyper-v-in-a-virtual-machine-with-nested-virtualization"></a>Exécuter Hyper-V dans une machine virtuelle avec la virtualisation imbriquée

La virtualisation imbriquée est une fonctionnalité qui vous permet d’exécuter Hyper-V à l’intérieur d’une machine virtuelle (VM) Hyper-V. Ceci est utile pour exécuter un émulateur de téléphone Visual Studio sur une machine virtuelle ou pour tester des configurations qui nécessitent généralement plusieurs hôtes.

![](./media/HyperVNesting.png)

## <a name="prerequisites"></a>Conditions préalables

* L’hôte et l’invité Hyper-V doivent tous deux exécuter Windows Server2016/la Mise à jour anniversaire Windows10 ou version ultérieure.
* Configuration de machine virtuelle version8.0 ou ultérieure.
* Processeur Intel avec la technologie VT-x et EPT -- l’imbrication est actuellement **exclusivement Intel**.
* Il existe certaines différences avec le réseau virtuel pour les machines virtuelles de second niveau. Voir «Mise en réseau de machines virtuelles imbriquées».


## <a name="configure-nested-virtualization"></a>Configurer la virtualisation imbriquée

1. Créer une machine virtuelle. Consultez la configuration requise ci-dessus pour les versions de système d’exploitation et les machines virtuelles.
2. Pendant que la machine virtuelle est à l’état DÉSACTIVÉ, exécutez la commande suivante sur l’hôte Hyper-V physique. Cela permet d’activer la virtualisation imbriquée de la machine virtuelle.

```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. Démarrez l'ordinateur virtuel.
4. Installez Hyper-V sur la machine virtuelle, comme vous le feriez sur un serveur physique. Pour plus d’informations sur l’installation d’Hyper-V, consultez [Installer Hyper-V](../quick-start/enable-hyper-v.md).

## <a name="disable-nested-virtualization"></a>Désactiver la virtualisation imbriquée
Vous pouvez désactiver la virtualisation imbriquée d’une machine virtuelle à l’arrêt à l’aide de la commande PowerShell suivante:
```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## <a name="dynamic-memory-and-runtime-memory-resize"></a>Mémoire dynamique et redimensionnement de la mémoire d’exécution
Hyper-V est en cours d’exécution dans une machine virtuelle, la machine virtuelle doit être désactivée pour ajuster sa mémoire. Cela signifie que même si la mémoire dynamique est activée, la quantité de mémoire ne varie pas. Pour les machines virtuelles dont la mémoire dynamique n’est pas activée, toute tentative d’ajustement de la quantité de mémoire en fonctionnement échouera. 

Notez que la simple activation de la virtualisation imbriquée n’a aucun effet sur la mémoire dynamique ou le redimensionnement de la mémoire runtime. L’incompatibilité se produit uniquement lorsque Hyper-V s’exécute dans la machine virtuelle Hyper-V.

## <a name="networking-options"></a>Options de mise en réseau

Il existe deux options pour la mise en réseau des machines virtuelles imbriquées: 

1. Usurpation des adresses MAC
2. Mise en réseau NAT

### <a name="mac-address-spoofing"></a>Usurpation des adresses MAC
Pour que les paquets réseau puissent être acheminés via deux commutateurs virtuels, l’usurpation des adresses MAC doit être activée sur le premier niveau du commutateur virtuel. Pour cela, exécutez la commande PowerShell suivante.

``` PowerShell
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="network-address-translation-nat"></a>Traduction d’adresses réseau (NAT)
La deuxième option s’appuie sur la traduction d’adresses réseau (NAT). Cette approche est idéale pour les cas où l’usurpation des adresses MAC n’est pas possible, comme dans un environnement de cloud public.

Tout d’abord, un commutateur NAT virtuel doit être créé dans la machine virtuelle hôte (machine virtuelle «intermédiaire»). Notez que les adressesIP ne sont qu’un exemple et peuvent varier entre les environnements:

``` PowerShell
New-VMSwitch -Name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```

Affectez ensuite une adresseIP à la carte réseau:

``` PowerShell
Get-NetAdapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```

Une adresseIP et une passerelle doivent être affectées à chaque machine virtuelle imbriquée. Notez que l’adresseIP de la passerelle doit pointer vers la carte NAT de l’étape précédente. Vous pouvez également affecter un serveur DNS:

``` PowerShell
Get-NetAdapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## <a name="how-nested-virtualization-works"></a>Fonctionnement de la virtualisation imbriquée

Les processeurs modernes incluent des fonctionnalités matérielles qui rendent la virtualisation plus rapide et plus sûre. Hyper-V s’appuie sur ces extensions de processeur pour exécuter des machines virtuelles (par ex. Intel VT-x et AMD-V). En règle générale, une fois que Hyper-V démarre, il empêche les autres logiciels d’utiliser ces fonctionnalités de processeur.  Cela empêche les machines virtuelles invitées d’exécuter Hyper-V.

Grâce à la virtualisation imbriquée, cette prise en charge matérielle est disponible pour les machines virtuelles invitées.

Le schéma ci-dessous illustre Hyper-V sans imbrication.  L’hyperviseur Hyper-V prend le contrôle total des fonctionnalités de virtualisation matérielle (flèche orange) et ne les expose pas au système d’exploitation invité.

![](./media/HVNoNesting.png)

Par contre, le schéma au-dessous illustre Hyper-V avec la virtualisation imbriquée activée. Dans ce cas, Hyper-V expose les extensions de virtualisation matérielle à ses machines virtuelles. Quand l’imbrication est activée, une machine virtuelle invitée peut installer son propre hyperviseur et exécuter ses propres machines virtuelles invitées.

![](./media/HVNesting.png)

## <a name="3rd-party-virtualization-apps"></a>Applications de virtualisation tierces

Les applications de virtualisation autres que Hyper-V ne sont pas prises en charge sur les machines virtuelles Hyper-V et sont susceptibles d’échouer. Cela inclut tout logiciel qui requiert des extensions matérielles pour la virtualisation.

---
title: "Virtualisation imbriquée"
description: "Virtualisation imbriquée"
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
translationtype: Human Translation
ms.sourcegitcommit: 26a8adb426a7cf859e1a9813da2033e145ead965
ms.openlocfilehash: d17413fc572e59ec21ff513ef5de994c6716aa08

---

# Exécuter Hyper-V dans une machine virtuelle avec la virtualisation imbriquée

La virtualisation imbriquée est une fonctionnalité qui vous permet d’exécuter Hyper-V à l’intérieur d’une machine virtuelle Hyper-V. En d’autres termes, avec la virtualisation imbriquée, un hôte Hyper-V peut être virtualisé. Certains cas d’utilisation de la virtualisation imbriquée consistent à exécuter un conteneur Hyper-V dans un hôte de conteneur virtualisé, configurer un laboratoire Hyper-V dans un environnement virtualisé ou tester des scénarios sur plusieurs machines sans besoin de matériel. Ce document décrit en détail la configuration matérielle et logicielle requise, les étapes de configuration et la résolution des problèmes. Si vous exécutez Hyper-V sur la version d’évaluation de Windows Insider 14361 ou une version ultérieure, voir [Version d’évaluation de la virtualisation imbriquée pour les Windows Insiders : builds 14361 et ultérieures](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting#nested-virtualization-preview-for-windows-insiders-builds-14361-).

## Conditions préalables

- Hôte Hyper-V exécutant une build Windows Insiders (Windows Server 2016 ou Windows 10) exécutant la build 10565 ou version ultérieure.
- Les deux hyperviseurs (parent et enfant) doivent exécuter des builds Windows identiques (10565 ou version ultérieure).
- Processeur Intel avec la technologie Intel VT-x et EPT.

## Configurer la virtualisation imbriquée

Créez d’abord une machine virtuelle, mais **n’activez pas la machine virtuelle**. Pour plus d’informations, consultez [Créer une machine virtuelle](../quick_start/walkthrough_create_vm.md).

Une fois que la machine virtuelle a été créée, exécutez la commande suivante sur l’hôte Hyper-V physique. Cela permet d’activer la virtualisation imbriquée sur la machine virtuelle.

```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
Lors de l’exécution d’un hôte Hyper-V imbriqué, la mémoire dynamique doit être désactivée sur la machine virtuelle. Cette configuration peut être effectuée sur les propriétés de la machine virtuelle ou en utilisant la commande PowerShell suivante.
```none
Set-VMMemory -VMName <VMName> -DynamicMemoryEnabled $false
```

Une fois ces étapes terminées, la machine virtuelle peut être démarrée et Hyper-V installé. Pour plus d’informations sur l’installation d’Hyper-V, consultez [Installer Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install).

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


## Problèmes connus

- Les hôtes sur lesquels la fonctionnalité Device Guard est activée ne peuvent pas exposer les extensions de virtualisation aux invités.
- Il n’est pas possible d’activer simultanément la sécurité basée sur la virtualisation et la virtualisation imbriquée sur les machines virtuelles. Vous devez d’abord désactiver la sécurité basée sur la virtualisation pour pouvoir utiliser la virtualisation imbriquée.
- Une fois la virtualisation imbriquée activée sur une machine virtuelle, les fonctionnalités suivantes ne sont plus compatibles avec cette machine virtuelle.  
  * Redimensionnement de la mémoire d’exécution, et mémoire dynamique
  * Points de contrôle
  * Une machine virtuelle sur laquelle Hyper-V est activé ne peut pas faire l’objet d’une migration dynamique.

## FAQ et résolution des problèmes

Ma machine virtuelle ne démarre pas, que dois-je faire ?

1. Assurez-vous que la mémoire dynamique est désactivée.
2. Vérifiez que vous disposez d’un processeur Intel compatible.
3. Exécutez [ce script PowerShell](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1) sur votre machine hôte à partir d’une invite de commandes avec élévation de privilèges.

## Commentaires

Indiquez tout autre problème dans l’application Commentaires sur Windows, les [forums sur la virtualisation](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv) ou sur [GitHub](https://github.com/Microsoft/Virtualization-Documentation).

##Version d’évaluation de la virtualisation imbriquée pour les Windows Insiders : builds 14361 et ultérieures
Quelques mois plus tôt, nous avons annoncé une première version d’évaluation de la virtualisation imbriquée Hyper-V avec la build 10565. Nous avons été ravis de voir l’intérêt que cette fonctionnalité a généré et sommes heureux de partager une mise à jour avec les Windows Insiders.

###Nouvelle version de machine virtuelle nécessaire pour la virtualisation imbriquée
À compter de la build 14361, la version 8.0 est obligatoire pour les machines virtuelles sur lesquelles la virtualisation imbriquée est activée. Une mise à jour de version est donc nécessaire pour les machines virtuelles sur lesquelles la virtualisation imbriquée est activée et qui ont été créées sur des hôtes plus anciens. 

####Mettre à jour la version de la machine virtuelle
Pour continuer à utiliser la virtualisation imbriquée, vous devez mettre à jour la version de la machine virtuelle vers la version 8.0. Cela signifie que l’état enregistré doit être supprimé et que la machine virtuelle doit être arrêtée. L’applet de commande PowerShell suivante met à jour la version de la machine virtuelle :
```none
Update-VMVersion -Name <VMName>
```
####Désactiver la virtualisation imbriquée
Si vous ne souhaitez pas mettre à jour la machine virtuelle, vous pouvez désactiver la virtualisation imbriquée pour que la machine virtuelle puisse démarrer :
```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

###Nouveau comportement pour la machine virtuelle version 8.0 
Il existe plusieurs modifications du fonctionnement des machines virtuelles sur lesquelles la virtualisation imbriquée est activée dans cette version d’évaluation :
-   La création et l’application des points de contrôle fonctionnent désormais pour les machines virtuelles sur lesquelles la virtualisation imbriquée est activée.
-   Vous pouvez maintenant enregistrer et démarrer des machines virtuelles sur lesquelles la virtualisation imbriquée est activée.
-   Les machines virtuelles sur lesquelles la virtualisation imbriquée est activée peuvent maintenant s’exécuter sur des hôtes sur lesquels la sécurité basée sur la virtualisation est activée (dont Device Guard et Credential Guard).
-   Nous avons amélioré les messages d’erreur pour les limitations existantes.

###Limitations fonctionnelles
-   La virtualisation imbriquée est conçue pour exécuter Hyper-V dans une machine virtuelle Hyper-V. Les applications de virtualisation tierces ne sont pas prises en charge et risquent d’échouer dans des machines virtuelles Hyper-V.
-   La mémoire dynamique n’est pas compatible avec la virtualisation imbriquée. Quand Hyper-V est en cours d’exécution à l’intérieur d’une machine virtuelle, celle-ci ne peut pas modifier sa mémoire lors de l’exécution. 
-   Le redimensionnement de la mémoire d’exécution n’est pas compatible avec la virtualisation imbriquée. Le redimensionnement de la mémoire d’une machine virtuelle dans laquelle Hyper-V est en cours d’exécution échoue. 
-   La virtualisation imbriquée est uniquement prise en charge sur les systèmes Intel.

###Problème connu
Il existe un problème connu dans la build 14361 où les machines virtuelles de génération 2 ne démarrent pas avec l’erreur suivante :
```none
“Cannot modify property without enabling VirtualizationBasedSecurityOptOut”
```
Ce problème peut être temporairement résolu en désactivant la virtualisation imbriquée ou en refusant la sécurité basée sur la virtualisation :
```none
Set-VMSecurity -VMName <vmname> -VirtualizationBasedSecurityOptOut $true
```

###Nous sommes à votre écoute
Comme toujours, continuez à nous envoyer vos commentaires avec l’application Commentaires sur Windows. Si vous avez des questions, entrez un problème dans la page [GitHub](https://github.com/Microsoft/Virtualization-Documentation) de notre documentation. 



<!--HONumber=Jul16_HO1-->



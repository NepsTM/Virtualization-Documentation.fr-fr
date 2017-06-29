---
title: Portage d&quot;Hyper-V WMIv1 sur WMIv2
description: "Découvrez comment porter Hyper-V WMIv1 sur WMIv2"
keywords: windows10, hyper-v, WMIv1, WMIv2, WMI, Msvm_VirtualSystemGlobalSettingData, root\virtualization
author: scooley
ms.date: 04/13/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: b13a3594-d168-448b-b0a1-7d77153759a8
ms.openlocfilehash: e2d6faabe77346199a5d292fcfd92cdfd63909b8
ms.sourcegitcommit: a424c11258e47f224e14c4349b852b9e37b7604f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/10/2017
---
# <a name="move-from-hyper-v-wmi-v1-to-wmi-v2"></a>Passer d’Hyper-V WMIv1 à WMIv2

WindowsManagementInstrumentation (WMI) est l’interface de gestion sous-jacente des applets de commande PowerShell du Gestionnaire Hyper-V et d'Hyper-V.  Bien que la plupart des gens utilisent nos applets de commande PowerShell ou le Gestionnaire Hyper-V, les développeurs avaient parfois besoin d'utiliser WMI directement.  

Il y a eu deux espaces de noms WMI Hyper-V (ou des versions de l’API WMI Hyper-V).
* L'espace de noms WMIv1 (root\virtualization), qui a été introduit dans Windows Server2008 et qui fut disponible pour la dernière fois dans Windows Server2012
* L'espace de noms WMIv2 (root\virtualization\v2), qui a été introduit dans Windows Server2012

Ce document contient des références à des ressources permettant de convertir du code qui communique avec notre ancien espace de noms WMI vers le nouveau.  Dans un premier temps, cet article servira de référentiel d'informations sur les API et d’exemples de code ou de scripts pouvant être utilisés pour aider à porter des programmes ou scripts qui utilisent les API de WMI Hyper-V depuis l’espace de noms v1 vers l’espace de noms v2.

## <a name="msdn-samples"></a>Exemples sur MSDN

[Exemple de migration d'une machine virtuelle Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-machine-aef356ee)  
[Exemple de Fibre Channel virtuelle Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-Fiber-35d27dcd)  
[Exemple de machines virtuelles Hyper-V planifiées](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-planned-virtual-8c7b7499)  
[Exemple d’analyse de l’intégrité des applications Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-application-health-dc0294f2)  
[Exemple de gestion de disque dur virtuel](http://code.msdn.microsoft.com/windowsdesktop/Virtual-hard-disk-03108ed3)  
[Exemple de réplication Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-replication-sample-d2558867)  
[Exemple de métriques d’Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-metrics-sample-2dab2cb1)  
[Exemple de mémoire dynamique Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-dynamic-memory-9b0b1d05)  
[Pilote de filtre d’extension de commutateur extensible Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-Extensible-Virtual-e4b31fbb)  
[Exemple de mise en réseau Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-networking-sample-7c47e6f5)  
[Exemple de gestion de pool de ressources Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-resource-pool-df906d95)  
[Exemple d'instantané de récupération Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-recovery-snapshot-ea72320c)  

## <a name="samples-from-blogs"></a>Exemples tirés de blogs

[Ajout d’une carte réseau à une machine virtuelle à l’aide de l'espace de noms WMI Hyper-VV2](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/adding-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Connexion d’une carte réseau de machine virtuelle à un commutateur à l’aide de l'espace de noms WMI Hyper-VV2](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/connecting-a-vm-network-adapter-to-a-switch-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Modification de l’adresse MAC d'une carte réseau à l’aide de l'espace de noms WMI Hyper-VV2](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/changing-the-mac-address-of-nic-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Retrait d’une carte réseau d'une machine virtuelle à l’aide de l'espace de noms WMI Hyper-VV2](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Connexion d'un disque dur virtuel à une machine virtuelle à l’aide de l'espace de noms WMI Hyper-VV2](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/attaching-a-vhd-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Suppression d’un disque dur virtuel à partir d’une machine virtuelle à l’aide de l'espace de noms WMI Hyper-VV2](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-vhd-from-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Création d’une machine virtuelle à l’aide de l'espace de noms WMI Hyper-VV2](http://blogs.msdn.com/b/virtual_pc_guy/archive/2013/06/20/creating-a-virtual-machine-with-wmi-v2.aspx)


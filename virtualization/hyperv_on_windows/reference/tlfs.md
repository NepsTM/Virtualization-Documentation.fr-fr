---
title: "Spécifications de l’hyperviseur"
description: "Spécifications de l’hyperviseur"
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
translationtype: Human Translation
ms.sourcegitcommit: e62373c30c6e233ab1db94720c5b2a1512dfb936
ms.openlocfilehash: 49fd4f7f5a035c68975e3c38b2daabeed8e7dd5f

---

# Spécifications de l’hyperviseur

## Spécification fonctionnelle générale de l’hyperviseur

La spécification fonctionnelle générale de l’hyperviseur Hyper-V décrit le comportement de l’hyperviseur visible de l’extérieur par d’autres composants du système d’exploitation. Cette spécification est destinée aux développeurs des systèmes d’exploitation invités.
  
> Elle est fournie dans le cadre de Microsoft Open Specification Promise.  Consultez les informations suivantes pour plus de détails sur [Microsoft Open Specification Promise](https://msdn.microsoft.com/en-us/openspecifications).  

#### Télécharger
Version | Document
--- | ---
Windows Server 2012 R2 (révision B) | [Hypervisor Top Level Functional Specification v4.0b.pdf (Spécification fonctionnelle générale de l’hyperviseur)](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 R2 | [Hypervisor Top Level Functional Specification v4.0.pdf (Spécification fonctionnelle générale de l’hyperviseur)](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf (Spécification fonctionnelle générale de l’hyperviseur)](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf (Spécification fonctionnelle générale de l’hyperviseur)](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## Configuration requise pour l’implémentation de l’interface de l’hyperviseur Microsoft

Les systèmes d’exploitation Windows nécessitent un ensemble limité d’interfaces hyperviseur pour s’exécuter sur une machine virtuelle invitée (également appelée interface « HV#1 »). Plusieurs fonctionnalités facultatives peuvent également être implémentées par un hyperviseur compatible avec Microsoft. Ces options changent le comportement de Windows sur une machine virtuelle. Le document « Requirements for Implementing the Microsoft Hypervisor Interface.pdf » (Configuration requise pour l’implémentation de l’interface de l’hyperviseur Microsoft) aborde les fonctionnalités requises et facultatives qui sont implémentées par un hyperviseur compatible avec Microsoft.

#### Télécharger

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf (Configuration requise pour l’implémentation de l’interface de l’hyperviseur Microsoft)](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)


<!--HONumber=Jun16_HO5-->



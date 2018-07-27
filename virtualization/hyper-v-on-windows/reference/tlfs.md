---
title: Spécifications de l’hyperviseur
description: Spécifications de l’hyperviseur
keywords: Windows10, Hyper-V
author: allenma
ms.date: 06/26/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
ms.openlocfilehash: 26eaca5a583f8b2e11ca1e05e2fa9a4366fd8837
ms.sourcegitcommit: 594cc1728347646609ae1952ecc6c97fc659d0a9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/07/2018
ms.locfileid: "2226205"
---
# <a name="hypervisor-specifications"></a>Spécifications de l’hyperviseur

## <a name="hypervisor-top-level-functional-specification"></a>Spécification fonctionnelle générale de l’hyperviseur

La spécification fonctionnelle générale de l’hyperviseur Hyper-V décrit le comportement de l’hyperviseur visible de l’extérieur par d’autres composants du système d’exploitation. Cette spécification est destinée aux développeurs des systèmes d’exploitation invités.
  
> Elle est fournie dans le cadre de Microsoft Open Specification Promise.  Consultez les informations suivantes pour plus de détails sur [Microsoft Open Specification Promise](https://msdn.microsoft.com/en-us/openspecifications).  

#### <a name="download"></a>Télécharger
Version | Document
--- | ---
WindowsServer2016 (révisionC) | [Hypervisor Top Level Functional Specification v5.0c.pdf (Spécification fonctionnelle générale de l’hyperviseur)](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/live/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0C.pdf)
WindowsServer2012R2 (révisionB) | [Hypervisor Top Level Functional Specification v4.0b.pdf (Spécification fonctionnelle générale de l’hyperviseur)](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server2012 | [Hypervisor Top Level Functional Specification v3.0.pdf (Spécification fonctionnelle générale de l’hyperviseur)](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server2008R2 | [Hypervisor Top Level Functional Specification v2.0.pdf (Spécification fonctionnelle générale de l’hyperviseur)](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## <a name="requirements-for-implementing-the-microsoft-hypervisor-interface"></a>Configuration requise pour l’implémentation de l’interface de l’hyperviseurMicrosoft

La spécification fonctionnelle générale décrit chaque aspect de l’architecture d’hyperviseur spécifique àMicrosoft, qui est déclarée en tant qu’interface «HV#1» pour les ordinateurs virtuels invités.  Toutefois, toutes les interfaces décrites dans cette spécification ne doivent pas être implémentées par l’hyperviseur tiers désireux de déclarer la conformité avec la spécification d’hyperviseurHV #1 deMicrosoft. Le document «Configuration requise pour l’implémentation de l’interface de l’hyperviseur Microsoft» décrit l’ensemble minimal d’interfaces d’hyperviseur devant être implémenté par tout hyperviseur revendiquant la compatibilité avec l’interfaceHV#1 deMicrosoft.

#### <a name="download"></a>Télécharger

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf (Configuration requise pour l’implémentation de l’interface de l’hyperviseur Microsoft)](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)

---
title: Spouleur d’impression dans les conteneurs Windows
description: Décrit le comportement de travail actuel pour le service de spouleur d’impression dans les conteneurs Windows.
keywords: docker, conteneurs, imprimante, spouleur
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999096"
---
# <a name="print-spooler-in-windows-containers"></a>Spouleur d’impression dans les conteneurs Windows

Les applications dotées d’une dépendance sur les services d’impression peuvent être conteneurs avec succès avec les conteneurs Windows. Des exigences spéciales doivent être satisfaites pour pouvoir activer la fonctionnalité de service de l’imprimante. Ce guide décrit comment configurer correctement votre déploiement.

> [!IMPORTANT]
> Lorsque l’accès aux services d’impression dans les conteneurs fonctionne correctement, la fonctionnalité est limitée dans le formulaire. certaines actions liées à l’impression risquent de ne pas fonctionner. Par exemple, les applications qui ont une dépendance à l’installation de pilotes d’imprimante dans l’hôte ne peuvent pas être dépendantes dans la mesure où l' **installation de pilotes à partir d’un conteneur n’est pas prise en charge**. Ouvrez un commentaire ci-dessous si vous trouvez une fonctionnalité d’impression non prise en charge que vous voulez prendre en charge dans les conteneurs.

## <a name="setup"></a>Configuration

* L’hôte doit être Windows Server 2019 ou Windows 10 professionnel, version d’octobre 2018 ou une mise à jour.
* L’image [MCR.Microsoft.com/Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) doit correspondre à l’image de base ciblée. Les autres images de base de conteneur Windows (par exemple, nano Server et Windows Server Core) ne comportent pas le rôle de serveur d’impression.

### <a name="hyper-v-isolation"></a>Isolation Hyper-V

Nous vous recommandons d’utiliser votre conteneur avec l’isolation Hyper-V. Lorsque cette fonction est exécutée dans ce mode, vous pouvez avoir autant de conteneurs que vous le souhaitez en cours d’exécution avec l’accès aux services d’impression. Vous n’avez pas besoin de modifier le service de spouleur sur l’hôte.

Vous pouvez vérifier la fonctionnalité à l’aide de la requête PowerShell suivante:

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation hyperv mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```

### <a name="process-isolation"></a>Isolement du processus

En raison de la nature du noyau partagé de conteneurs isolés du processus, le comportement actuel limite l’utilisateur à n’exécuter qu' **une seule instance** du service de spouleur d’impression sur l’hôte et sur tous ses enfants. Si le spouleur d’imprimante est en cours d’exécution sur l’hôte, vous devez arrêter le service sur l’hôte avant Attemping pour lancer le service d’imprimante dans l’invité.

> [!TIP]
> Si vous lancez un conteneur et que vous interrogez le service de spouleur à la fois dans le conteneur et dans l’hôte, les deux indiquent qu’il s’agit de l’État «en cours d’exécution». Sans être tromper: le conteneur ne sera pas en mesure d’effectuer une requête pour obtenir la liste des imprimantes disponibles. Le service de spouleur de l’hôte ne doit pas s’exécuter. 

Pour vérifier si l’hôte exécute le service d’imprimante, utilisez la requête dans PowerShell ci-dessous:

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

Pour arrêter le service de spouleur sur l’hôte, utilisez les commandes suivantes dans PowerShell ci-dessous:

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

Lancez le conteneur et vérifiez l’accès aux imprimantes.

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation process mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.


PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```
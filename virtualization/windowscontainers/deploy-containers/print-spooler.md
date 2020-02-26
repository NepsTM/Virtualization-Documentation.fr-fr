---
title: Spouleur d’impression dans les conteneurs Windows
description: Explique le comportement de travail actuel du service spouleur d’impression dans les conteneurs Windows.
keywords: ancrage, conteneurs, imprimante, spouleur
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439536"
---
# <a name="print-spooler-in-windows-containers"></a>Spouleur d’impression dans les conteneurs Windows

Les applications avec une dépendance sur les services d’impression peuvent être correctement en conteneur avec les conteneurs Windows. Des exigences particulières doivent être remplies pour activer correctement les fonctionnalités du service d’impression. Ce guide explique comment configurer correctement votre déploiement.

> [!IMPORTANT]
> Lorsque l’accès aux services d’impression s’effectue correctement dans les conteneurs, les fonctionnalités sont limitées dans le formulaire. certaines actions liées à l’impression peuvent ne pas fonctionner. Par exemple, les applications qui dépendent de l’installation de pilotes d’imprimante dans l’hôte ne peuvent pas être en conteneur, car **l’installation de pilotes à partir d’un conteneur n’est pas prise en charge**. Veuillez ouvrir un commentaire ci-dessous si vous trouvez une fonctionnalité d’impression non prise en charge que vous souhaitez prendre en charge dans les conteneurs.

## <a name="setup"></a>Programme d'installation

* L’hôte doit être Windows Server 2019 ou Windows 10 professionnel/entreprise mise à jour 2018 ou version ultérieure.
* L’image [MCR.Microsoft.com/Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) doit être l’image de base ciblée. Les autres images de base du conteneur Windows (telles que nano Server et Windows Server Core) ne transmettent pas le rôle de serveur d’impression.

### <a name="hyper-v-isolation"></a>Isolation Hyper-V

Nous vous recommandons d’exécuter votre conteneur avec l’isolation Hyper-V. Lorsqu’il est exécuté dans ce mode, vous pouvez avoir autant de conteneurs que vous souhaitez exécuter avec un accès aux services d’impression. Vous n’avez pas besoin de modifier le service spouleur sur l’ordinateur hôte.

Vous pouvez vérifier la fonctionnalité avec la requête PowerShell suivante :

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

### <a name="process-isolation"></a>Isolation des processus

En raison de la nature partagée du noyau des conteneurs isolés par processus, le comportement actuel limite l’utilisateur à exécuter **une seule instance** du service spouleur d’impression sur l’hôte et tous ses enfants de conteneur. Si le spouleur de l’ordinateur hôte est en cours d’exécution, vous devez arrêter le service sur l’ordinateur hôte avant de tentative le lancement du service d’impression dans l’invité.

> [!TIP]
> Si vous lancez un conteneur et une requête pour le service de spouleur à la fois dans le conteneur et l’hôte simultanément, les deux rapports signalent leur état comme « en cours d’exécution ». Mais ne sont pas tromper : le conteneur ne peut pas interroger la liste des imprimantes disponibles. Le service spouleur de l’hôte ne doit pas s’exécuter. 

Pour vérifier si l’ordinateur hôte exécute le service d’imprimante, utilisez la requête dans PowerShell ci-dessous :

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

Pour arrêter le service Spooler sur l’ordinateur hôte, utilisez les commandes suivantes dans PowerShell ci-dessous :

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
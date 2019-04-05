---
title: Spouleur d’impression dans les conteneurs Windows
description: Explique le comportement de travail actuel pour le service Spouleur d’impression dans les conteneurs Windows
keywords: docker, conteneurs, imprimante, le spouleur
author: cwilhit
ms.openlocfilehash: 48130bc6a826a45dfa49d0a3b4600d227f34704e
ms.sourcegitcommit: 3c81b0efd1ac2c4c93d58f16edae1044c9a5ad55
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/05/2019
ms.locfileid: "9284573"
---
# <a name="print-spooler-in-windows-containers"></a>Spouleur d’impression dans les conteneurs Windows

Les applications avec une dépendance sur les services d’impression peuvent être en conteneur gérées avec des conteneurs Windows. Il existe des exigences spéciales qui doivent être remplies pour pouvoir activer la fonctionnalité de service d’imprimante avec succès. Ce guide explique comment configurer correctement votre déploiement.

> [!IMPORTANT]
> Pendant que l’accès à l’impression fonctionnement des services correctement dans des conteneurs, la fonctionnalité est limitée sous forme; certaines actions relatives à l’impression ne peuvent pas fonctionner. Par exemple, les applications qui ont une dépendance sur l’installation des pilotes d’imprimante dans l’hôte ne peut pas être en conteneur gérées dans la mesure où **l’installation du pilote à partir d’un conteneur est non pris en charge**. Ouvrez un retour ci-dessous si vous trouvez une fonctionnalité d’impression non pris en charge que vous voulez être pris en charge dans les conteneurs.

## <a name="setup"></a>Configuration

* L’hôte doit être Windows Server 2019 ou Windows 10 Professionnel/entreprise octobre 2018 mettre à jour ou une version ultérieure.
* L’image [mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) doit être l’image de base ciblée. Autres images de base de conteneur Windows (par exemple, Nano Server et Windows Server Core) ne bénéficient pas le rôle de serveur d’impression.

### <a name="hyper-v-isolation"></a>Isolation Hyper-V

Nous vous recommandons d’exécuter votre conteneur avec l’isolation Hyper-V. Lorsque vous l’exécutez dans ce mode, vous pouvez avoir autant de conteneurs que vous le souhaitez en cours d’exécution avec accès aux services d’impression. Vous n’avez pas besoin de modifier le spouleur sur l’ordinateur hôte.

Vous pouvez vérifier la fonctionnalité avec la requête PowerShell suivante:

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

En raison de la nature de noyau partagé des conteneurs isolées du processus, le comportement actuel limite l’utilisateur qu' **une seule instance** du spouleur d’imprimante en cours d’exécution sur l’hôte et tous ses enfants du conteneur. Si l’hôte dispose le spouleur d’impression en cours d’exécution, vous devez arrêter le service sur l’ordinateur hôte avant attemping pour lancer le service d’imprimante dans l’invité.

> [!TIP]
> Si vous lancez un conteneur et interroger simultanément pour le service spouleur dans le conteneur et l’hôte, les deux signalera leur état comme «en cours d’exécution». Mais ne soyez pas attirer--le conteneur ne sera pas en mesure d’interroger pour obtenir la liste des imprimantes disponibles. Spouleur de l’hôte ne doit pas s’exécuter. 

Pour vérifier si l’ordinateur hôte est en cours d’exécution du service de l’imprimante, utilisez la requête dans PowerShell ci-dessous:

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

Pour arrêter le service spouleur sur l’ordinateur hôte, utilisez les commandes suivantes dans PowerShell ci-dessous:

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

Lancez le conteneur et vérifier l’accès aux imprimantes.

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
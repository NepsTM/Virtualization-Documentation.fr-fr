---
title: Conteneurs Hyper-V
description: Explaination de quoi les conteneurs Hyper-V diffèrent des conteneurs de processus.
keywords: docker, conteneurs
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: caaf4186f43c69dfbc35d04dd8909876ed082906
ms.sourcegitcommit: 4336d7617c30d26a987ad3450b048e17404c365d
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2019
ms.locfileid: "9000998"
---
# <a name="hyper-v-containers"></a>Conteneurs Hyper-V

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

La technologie de conteneur Windows comprend deux types distincts de conteneurs, les conteneurs Windows Server (processus conteneurs) et les conteneurs Hyper-V. Les deux types de conteneurs sont créés, sont gérés et fonctionnent de manière identique. Ils produisent et consomment aussi les mêmes images de conteneur. La différence entre eux est le niveau d’isolation créé entre le conteneur, le système d’exploitation hôte et tous les autres conteneurs exécutés sur cet hôte.

**Conteneurs Windows Server**: plusieurs instances de conteneurs peuvent s’exécuter simultanément sur un hôte avec une isolation assurée par le biais des technologies des espaces de noms, du contrôle des ressources et de l’isolation des processus.  Les conteneurs Windows Server partagent le même noyau avec l’hôte ainsi qu’entre eux.  Il s’agit environ le même que la manière dont les conteneurs exécutés sur Linux.

**Conteneurs Hyper-V** : plusieurs instances de conteneurs peuvent s’exécuter simultanément sur un hôte, cependant, chaque conteneur s’exécute à l’intérieur d’une machine virtuelle spéciale. Ceci fournit une isolation de niveau noyau entre chaque conteneur Hyper-V et l’hôte des conteneurs.

## <a name="hyper-v-container-examples"></a>Exemples de conteneur Hyper-V

### <a name="create-container"></a>Créer un conteneur

Gestion des conteneurs Hyper-V avec Docker sont presque identique à la gestion des conteneurs Windows Server. Pour créer un conteneur Hyper-V avec Docker, utilisez le `--isolation` paramètre pour définir `--isolation=hyperv`.

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>Description de l’isolation

Cet exemple illustre les différences de fonctionnalités d’isolement entre les conteneurs Hyper-V et Windows Server. 

Ici, un conteneur Windows Server est déployé et doit héberger un processus ping à long terme.

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:1809 ping localhost -t
```

À l’aide de la commande `docker top`, le processus ping est retourné comme indiqué dans le conteneur. Le processus de cet exemple a l’ID3964.

``` cmd
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

Sur l’hôte de conteneur, la commande `get-process` peut être utilisée pour retourner tous les processus ping en cours d’exécution à partir de l’hôte. Dans cet exemple, il en existe un, et l’ID de processus correspond à celui du conteneur. Le même processus est visible à la fois depuis le conteneur et l’hôte.

```
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

Par contre, cet exemple démarre également un conteneur Hyper-V avec un processus ping. 

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 ping -t localhost
```

De même, `docker top` peut servir à retourner les processus en cours d’exécution à partir du conteneur.

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

Toutefois, lors de la recherche du processus sur l’hôte de conteneur, un processus ping est introuvable et une erreur est générée.

```
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

Enfin, sur l’hôte, le processus `vmwp` est visible, qui correspond à la machine virtuelle en cours d’exécution qui encapsule le conteneur en cours d’exécution et protège les processus exécutés à partir du système d’exploitation hôte.

```
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```

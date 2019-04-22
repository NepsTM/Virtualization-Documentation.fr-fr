---
title: Isolation Hyper-V
description: Explaination de quoi l’isolation Hyper-V diffèrent des conteneurs de processus isolé.
keywords: docker, conteneurs
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 4ab473c1752c377955bb23bdf6c9ef83a3336aa8
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380123"
---
# <a name="hyper-v-isolation"></a>Isolation Hyper-V

La technologie de conteneur Windows comprend deux niveaux distincts d’isolation pour les conteneurs, processus et isolation Hyper-V. Les deux types sont créés, sont gérés et fonctionnent de manière identique. Ils produisent et consomment aussi les mêmes images de conteneur. La différence entre eux est le niveau d’isolation créé entre le conteneur, le système d’exploitation hôte et tous les autres conteneurs exécutés sur cet hôte.

**L’isolation des processus** – conteneur plusieurs instances peuvent s’exécuter simultanément sur un hôte, avec l’isolation fournie par le biais d’espace de noms, le contrôle de la ressource et l’isolation des processus technologies.  Les conteneurs partagent le même noyau avec l’hôte, ainsi que l’autre.  Il s’agit environ le même que la manière dont les conteneurs exécutés sur Linux.

**Isolation Hyper-V** : plusieurs instances de conteneurs peuvent s’exécuter simultanément sur un hôte, cependant, chaque conteneur s’exécute à l’intérieur d’une machine virtuelle spéciale. Ceci fournit une isolation au niveau du noyau entre chaque conteneur ainsi que l’hôte de conteneur.

## <a name="hyper-v-isolation-examples"></a>Exemples d’isolation Hyper-V

### <a name="create-container"></a>Créer un conteneur

Gestion des conteneurs isolé Hyper-V avec Docker sont presque identique à la gestion des conteneurs Windows Server. Pour créer un conteneur avec l’isolation Hyper-V Docker approfondi, utilisez le `--isolation` paramètre pour définir `--isolation=hyperv`.

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>Description de l’isolation

Cet exemple illustre les différences de fonctionnalités d’isolement entre l’isolation Hyper-V et Windows Server.

Ici, un processus conteneur isolé est en cours de déploiement et doit héberger un processus ping long.

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

En revanche, cet exemple démarre un conteneur isolé Hyper-V avec un processus ping également.

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

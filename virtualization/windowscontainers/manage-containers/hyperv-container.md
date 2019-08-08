---
title: Isolation Hyper-V
description: Explication de la façon dont l’isolation Hyper-V est différente de celle des conteneurs isolés de processus.
keywords: docker, conteneurs
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 092312848173102bec5a791f2c48fe8166e70d5f
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998326"
---
# <a name="hyper-v-isolation"></a>Isolation Hyper-V

La technologie conteneur Windows inclut deux niveaux d’isolement distincts pour les conteneurs, le processus et l’isolation Hyper-V. Les deux types sont créés, gérés et fonctionne de la même manière. Ils produisent et consomment aussi les mêmes images de conteneur. La différence entre eux est le niveau d’isolation créé entre le conteneur, le système d’exploitation hôte et tous les autres conteneurs exécutés sur cet hôte.

**Isolement de processus** : plusieurs instances de conteneur peuvent être exécutées simultanément sur un hôte, avec l’isolation fournie par le biais d’espaces de noms, de contrôles de ressources et de technologies d’isolation de processus.  Les conteneurs partagent le même noyau avec l’hôte, et les uns les autres.  C’est approximativement la même que la manière dont les conteneurs s’exécutent sur Linux.

**Isolement Hyper-V** : plusieurs instances de conteneur peuvent être exécutées simultanément sur un hôte, mais chaque conteneur s’exécute à l’intérieur d’une machine virtuelle spéciale. Cela fournit une isolation de niveau noyau entre chaque conteneur et l’hôte de conteneur.

## <a name="hyper-v-isolation-examples"></a>Exemples d’isolation Hyper-V

### <a name="create-container"></a>Créer un conteneur

La gestion des conteneurs isolés Hyper-V avec l’amarrage est presque identique à la gestion des conteneurs Windows Server. Pour créer un conteneur avec l’isolement Hyper-V complet, utilisez le `--isolation` paramètre à définir. `--isolation=hyperv`

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>Description de l’isolation

Cet exemple illustre les différences de fonctionnalités d’isolation entre Windows Server et l’isolation Hyper-V.

Dans cet exemple, un conteneur process isolated est en cours de déploiement et sera hébergé par un processus ping à exécution longue.

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

Pour le contraster, cet exemple démarre un conteneur Hyper-V isolé avec un processus ping.

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

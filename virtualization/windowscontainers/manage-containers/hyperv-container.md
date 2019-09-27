---
title: Modes d’isolation
description: Explication de la façon dont l’isolation Hyper-V est différente de celle des conteneurs isolés de processus.
keywords: docker, conteneurs
author: crwilhit
ms.date: 09/26/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: fa95ffe1c699a2c837076fcc1b662f6b792b7dfb
ms.sourcegitcommit: e9dda81f1f68359ece9ef132a184a30880bcdb1b
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/27/2019
ms.locfileid: "10161726"
---
# <a name="isolation-modes"></a>Modes d’isolation

Les conteneurs Windows proposent deux modes d’isolement du runtime `process` : `Hyper-V` et isolation. Les conteneurs qui s’exécutent sous les deux modes d’isolation sont créés, gérés et fonctionnent de la même manière. Ils produisent et consomment aussi les mêmes images de conteneur. La différence entre les modes d’isolation est le niveau d’isolement qui est créé entre le conteneur, le système d’exploitation hôte, et tous les autres conteneurs en cours d’exécution sur cet hôte.

## <a name="process-isolation"></a>Isolement du processus

Il s’agit du mode d’isolation «traditionnel» pour les conteneurs et est décrit dans la [vue d’ensemble des conteneurs Windows](../about/index.md). Avec l’isolement de processus, plusieurs instances de conteneur s’exécutent en même temps sur un hôte donné avec l’isolation fournie par le biais d’espaces de noms, de contrôles de ressources et de technologies d’isolation de processus. Lorsque ce mode est exécuté dans ce mode, les conteneurs partagent le même noyau avec l’hôte ainsi que les autres.  C’est approximativement la même que la manière dont les conteneurs Linux s’exécutent.

![](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Isolation Hyper-V
Ce mode d’isolation offre une plus grande sécurité et une meilleure compatibilité entre les versions d’hôte et de conteneur. Avec l’isolation Hyper-V, plusieurs instances de conteneur s’exécutent simultanément sur un hôte; Toutefois, chaque conteneur s’exécute à l’intérieur d’une machine virtuelle hautement optimisée et obtient efficacement son propre noyau. La présence de l’ordinateur virtuel fournit une isolation matérielle entre chaque conteneur et l’hôte de conteneur.

![](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>Exemples d’isolement

### <a name="create-container"></a>Créer un conteneur

La gestion des conteneurs Hyper-V avec l’amarrage est presque identique à la gestion des conteneurs isolés de processus. Pour créer un conteneur avec l’isolement Hyper-V complet, utilisez le `--isolation` paramètre à définir. `--isolation=hyperv`

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Pour créer un conteneur avec isolement de processus complet, utilisez le `--isolation` paramètre à définir. `--isolation=process`

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Les conteneurs Windows qui s’exécutent sur Windows Server s’exécutent par défaut avec l’isolement de processus. Les conteneurs Windows fonctionnant sous Windows 10 professionnel ou entreprise s’exécutent par défaut avec l’isolation Hyper-V. À partir de la mise à jour de Windows 10 d’octobre 2018, les utilisateurs exécutant un hôte Windows 10 professionnel ou entreprise peuvent exécuter un conteneur Windows avec l’isolement de processus. Les utilisateurs doivent faire une demande d’isolement de processus `--isolation=process` à l’aide de l’indicateur.

> [!WARNING]
> L’exécution avec l’isolement de processus sur Windows 10 professionnel et entreprise est destinée aux développements et aux tests. Votre hôte doit exécuter Windows 10 Build 17763 + et vous devez disposer d’une version d’amarrage du moteur 18,09 ou d’une version ultérieure.
> 
> Vous devez continuer à utiliser Windows Server en tant qu’hôte pour les déploiements de production. En utilisant cette fonctionnalité sur Windows 10 professionnel et entreprise, vous devez également vous assurer que les balises de version de votre hôte et de votre conteneur correspondent, sinon le conteneur peut ne pas démarrer ou présenter un comportement non défini.

### <a name="isolation-explanation"></a>Description de l’isolation

Cet exemple montre les différences de fonctionnalités d’isolation entre processus et isolation Hyper-V.

Dans cet exemple, un conteneur en isolement de processus est déployé et héberge un processus ping à exécution longue.

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
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

Pour contraster, cet exemple démarre un conteneur Hyper-V-solated avec un processus ping.

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
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

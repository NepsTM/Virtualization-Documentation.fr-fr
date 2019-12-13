---
title: Modes d’isolation
description: Explication de la différence entre l’isolation Hyper-V et le traitement des conteneurs isolés.
keywords: docker, conteneurs
author: crwilhit
ms.date: 09/26/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: fa95ffe1c699a2c837076fcc1b662f6b792b7dfb
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909749"
---
# <a name="isolation-modes"></a>Modes d’isolation

Les conteneurs Windows offrent deux modes distincts d’isolation du Runtime : `process` et `Hyper-V` isolation. Les conteneurs s’exécutant sous les deux modes d’isolation sont créés, gérés et fonctionnent de manière identique. Ils produisent et consomment aussi les mêmes images de conteneur. La différence entre les modes d’isolation est le degré d’isolement créé entre le conteneur, le système d’exploitation hôte et tous les autres conteneurs exécutés sur cet hôte.

## <a name="process-isolation"></a>Isolement des processus

Il s’agit du mode d’isolation « traditionnel » pour les conteneurs. c’est ce qui est décrit dans la [vue d’ensemble des conteneurs Windows](../about/index.md). Avec l’isolation des processus, plusieurs instances de conteneur s’exécutent simultanément sur un hôte donné avec l’isolation fournie par le biais des technologies d’espace de noms, de contrôle des ressources et d’isolation des processus. Lors de son exécution dans ce mode, les conteneurs partagent le même noyau avec l’hôte et l’ordinateur hôte.  Cela est quasiment identique à celui de l’exécution des conteneurs Linux.

![](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Isolation Hyper-V
Ce mode d’isolation offre une sécurité accrue et une plus grande compatibilité entre les versions d’hôte et de conteneur. Avec l’isolation Hyper-V, plusieurs instances de conteneur s’exécutent simultanément sur un hôte ; Toutefois, chaque conteneur s’exécute à l’intérieur d’une machine virtuelle hautement optimisée et reçoit efficacement son propre noyau. La présence de l’ordinateur virtuel fournit un isolement au niveau matériel entre chaque conteneur et l’hôte de conteneur.

![](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>Exemples d’isolation

### <a name="create-container"></a>Créer un conteneur

La gestion d’un conteneur isolé Hyper-V avec l’arrimeur est presque identique à la gestion des conteneurs à isolation de processus. Pour créer un conteneur avec l’isolement Hyper-V complet, utilisez le paramètre `--isolation` pour définir `--isolation=hyperv`.

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Pour créer un conteneur avec l’isolation de processus complet, utilisez le paramètre `--isolation` pour définir `--isolation=process`.

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Les conteneurs Windows s’exécutant sur Windows Server s’exécutent par défaut avec l’isolation des processus. Les conteneurs Windows s’exécutant sur Windows 10 professionnel et entreprise utilisent par défaut l’isolation Hyper-V. À compter de la mise à jour 2018 de Windows 10 octobre, les utilisateurs qui exécutent un hôte Windows 10 professionnel ou entreprise peuvent exécuter un conteneur Windows avec l’isolation des processus. Les utilisateurs doivent demander directement l’isolation du processus à l’aide de l’indicateur `--isolation=process`.

> [!WARNING]
> L’exécution avec l’isolation des processus sur Windows 10 professionnel et entreprise est destinée au développement et aux tests. Votre ordinateur hôte doit exécuter Windows 10 Build 17763 + et vous devez disposer d’une version d’ancrage avec le moteur 18,09 ou une version ultérieure.
> 
> Vous devez continuer à utiliser Windows Server comme hôte pour les déploiements de production. En utilisant cette fonctionnalité sur Windows 10 professionnel et entreprise, vous devez également vous assurer que les balises de version de l’hôte et du conteneur correspondent, sans quoi le conteneur peut ne pas démarrer ou présenter un comportement indéfini.

### <a name="isolation-explanation"></a>Description de l’isolation

Cet exemple illustre les différences de fonctionnalités d’isolation entre le processus et l’isolation Hyper-V.

Ici, un conteneur isolé du processus est déployé et hébergera un processus ping à long terme.

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

À l’aide de la commande `docker top`, le processus ping est retourné comme indiqué dans le conteneur. Le processus de cet exemple a l’ID 3964.

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

À l’inverse, cet exemple démarre un conteneur Hyper-V-solated avec un processus ping également.

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

---
title: Modes d’isolation
description: Explication de la différence entre l’isolation Hyper-V et le traitement de conteneurs isolés.
keywords: docker, conteneurs
author: cwilhit
ms.date: 09/26/2019
ms.topic: conceptual
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: c7bcb25b2c3b65be745971ae2dec4d509266a1b3
ms.sourcegitcommit: bb18e6568393da748a6d511d41c3acbe38c62668
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/12/2020
ms.locfileid: "88161948"
---
# <a name="isolation-modes"></a>Modes d’isolation

Les conteneurs Windows offrent deux modes distincts d’isolation du runtime : isolation `process` et `Hyper-V`. Les conteneurs s’exécutant sous les deux modes d’isolation sont créés, gérés et fonctionnent de manière identique. Ils produisent et consomment aussi les mêmes images de conteneur. La différence entre les modes d’isolation est le degré d’isolation créé entre le conteneur, le système d’exploitation de l’hôte et tous les autres conteneurs s’exécutant sur celui-ci.

## <a name="process-isolation"></a>Isolation des processus

Il s’agit du mode d’isolation « traditionnel » pour les conteneurs, comme décrit dans la [Vue d’ensemble des conteneurs Windows](../about/index.md). Grâce à l’isolation des processus, plusieurs instances de conteneur s’exécutent simultanément sur un hôte en bénéficiant d’une isolation assurée par des technologies d’espaces de noms, de contrôle des ressources et d’isolation des processus. En cas d’exécution dans ce mode, les conteneurs partagent un même noyau avec l’hôte ainsi qu’entre eux.  Cela se produit pratiquement de la même façon que l’exécution des conteneurs Linux.

![Diagramme montrant un conteneur rempli d’applications isolées du système d’exploitation et du matériel.](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Isolation Hyper-V
Ce mode d’isolation offre une sécurité accrue et une plus grande compatibilité entre les versions de l’hôte et du conteneur. Avec l’isolation Hyper-V, plusieurs instances de conteneur s’exécutent simultanément sur un hôte. Toutefois, chaque conteneur s’exécute à l’intérieur d’une machine virtuelle hautement optimisée, et reçoit efficacement son propre noyau. La présence de l’ordinateur virtuel fournit un isolation au niveau matériel entre les conteneurs, ainsi qu’entre l’hôte et les conteneurs.

![Diagramme d’un conteneur isolé dans un système d’exploitation d’une machine virtuelle qui s’exécute sur un système d’exploitation d’un ordinateur physique.](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>Exemples d’isolation

### <a name="create-container"></a>Créer un conteneur

La gestion de conteneurs isolés par Hyper-V avec Docker est pratiquement identique à la gestion de conteneurs isolés par processus. Pour créer un conteneur avec isolation Hyper-V via Docker, utilisez le paramètre `--isolation` pour définir `--isolation=hyperv`.

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Pour créer un conteneur avec isolation par processus via Docker, utilisez le paramètre `--isolation` pour définir `--isolation=process`.

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Les conteneurs Windows s’exécutant sur Windows Server utilisent par défaut une isolation des processus. Les conteneurs Windows s’exécutant sur Windows 10 Professionnel et Entreprise utilisent par défaut une isolation Hyper-V. Depuis la mise à jour de Windows 10 d’octobre 2018, les utilisateurs qui exécutent un hôte Windows 10 Professionnel ou Entreprise peuvent exécuter un conteneur Windows avec une isolation des processus. Les utilisateurs doivent demander directement l’isolation des processus à l’aide de l’indicateur `--isolation=process`.

> [!WARNING]
> L’exécution avec isolation des processus sur Windows 10 Professionnel et Entreprise est destinée au développement et aux tests. Votre ordinateur hôte doit exécuter Windows 10, build 17763+, et vous devez disposer d’une version de Docker avec le moteur 18.09 ou ultérieur.
>
> Vous devez continuer d'utiliser Windows Server en tant qu'hôte pour les déploiements de production. En utilisant cette fonctionnalité sur Windows 10 Professionnel et Entreprise, vous devez également vous assurer que les balises de version de votre hôte et de votre conteneur correspondent, sans quoi le conteneur risque de ne pas pouvoir démarrer ou ne pas se comporter de manière imprévisible.

### <a name="isolation-explanation"></a>Description de l’isolation

Cet exemple montre les différences de capacités d’isolation entre l’isolation des processus et l’isolation Hyper-V.

Ici, un conteneur isolé par processus est déployé, qui devra héberger un processus ping de longue durée.

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

Par contre, cet exemple démarre également un conteneur isolé par Hyper-V avec un processus ping.

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

De même, `docker top` peut servir à retourner les processus en cours d’exécution à partir du conteneur.

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

Toutefois, lors de la recherche du processus sur l’hôte du conteneur, aucun processus ping n’est trouvé et une erreur est levée.

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

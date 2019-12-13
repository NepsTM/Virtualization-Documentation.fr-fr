---
title: Plateforme de conteneur Windows
description: En savoir plus sur les nouveaux blocs de construction de conteneur disponibles dans Windows.
keywords: LCOW, conteneurs Linux, docker, conteneurs, containered, élément de rapport personnalisé, runhcs, Runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 74e22702aa4be30055b3f4f48c7fac926d793095
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909919"
---
# <a name="container-platform-tools-on-windows"></a>Outils de plateforme de conteneur sur Windows

La plateforme de conteneur Windows est en expansion ! La première partie du parcours du conteneur est maintenant la création d’autres outils de plateforme de conteneur.

* [conteneur/élément de rapport personnalisé](https://github.com/containerd/cri) -nouveauté dans Windows Server 2019/Windows 10 1809.
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) : un équivalent de l’hôte de conteneur Windows à Runc.
* [HCS](https://docs.microsoft.com/virtualization/api/) : service de calcul de l’hôte + shims pratiques pour faciliter son utilisation.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

Cet article traite de la plateforme de conteneur Windows et Linux, ainsi que de chaque outil de plateforme de conteneur.

## <a name="windows-and-linux-container-platform"></a>Plateforme de conteneur Windows et Linux

Dans les environnements Linux, les outils de gestion de conteneurs tels que docker sont basés sur un ensemble plus granulaire d’outils de conteneur : [Runc](https://github.com/opencontainers/runc) et [containered](https://containerd.io/).

![Architecture de l’amarrage sur Linux](media/docker-on-linux.png)

`runc` est un outil en ligne de commande Linux pour la création et l’exécution de conteneurs en fonction de la [spécification d’exécution du conteneur OCI](https://github.com/opencontainers/runtime-spec).

`containerd` est un démon qui gère le cycle de vie du conteneur du téléchargement et de la décompression de l’image conteneur pour l’exécution et la surveillance des conteneurs.

Sur Windows, nous avons adopté une approche différente.  Lorsque nous avons commencé à travailler avec l’arrimeur pour prendre en charge les conteneurs Windows, nous avons créé directement sur le HCS (service de calcul hôte).  [Ce](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332) billet de blog explique pourquoi nous avons créé le HCS et pourquoi nous avons initialement adopté cette approche pour les conteneurs.

![Architecture initiale du moteur de l’ancrage sur Windows](media/hcs.png)

À ce stade, l’arrimeur appelle toujours directement dans le HCS. Toutefois, les outils de gestion de conteneur qui se développent pour inclure des conteneurs Windows et l’hôte de conteneur Windows peuvent appeler dans containered et runhcs de la façon dont ils appellent sur des conteneurs et Runc sur Linux.

## <a name="runhcs"></a>runhcs

`runhcs` est une fourche de `runc`.  Comme `runc`, `runhcs` est un client de ligne de commande pour l’exécution d’applications empaquetées conformément au format OCI (Open Container initiative) et est une implémentation conforme de la spécification Open Container initiative.

Les différences fonctionnelles entre Runc et runhcs sont les suivantes :

* `runhcs` s’exécute sur Windows.  Il communique avec le [HCS](containerd.md#hcs) pour créer et gérer des conteneurs.
* `runhcs` pouvez exécuter divers types de conteneurs différents.

  * [Isolation Hyper-V](../manage-containers/hyperv-container.md) Windows et Linux
  * Conteneurs de processus Windows (l’image de conteneur doit correspondre à l’hôte de conteneur)

**Utilisation :**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` est le nom de l’instance de conteneur que vous démarrez. Le nom doit être unique sur votre hôte de conteneur.

Le répertoire du bundle (à l’aide de `-b bundle`) est facultatif.  
Comme avec Runc, les conteneurs sont configurés à l’aide d’offres groupées. L’offre groupée d’un conteneur est le répertoire contenant le fichier de spécification OCI du conteneur, « config. JSON ».  La valeur par défaut de « Bundle » est le répertoire actif.

Le fichier de spécifications OCI, « config. JSON », doit avoir deux champs pour s’exécuter correctement :

* Chemin d’accès à l’espace de travail du conteneur
* Chemin d’accès au répertoire de couche du conteneur

Les commandes de conteneur disponibles dans runhcs sont les suivantes :

* Outils de création et d’exécution d’un conteneur
  * **exécuter** crée et exécute un conteneur
  * **créer** créer un conteneur

* Outils de gestion des processus s’exécutant dans un conteneur :
  * **Démarrer** exécute le processus défini par l’utilisateur dans un conteneur créé
  * **Exec** exécute un nouveau processus à l’intérieur du conteneur
  * **suspendre la** suspension interrompt tous les processus à l’intérieur du conteneur
  * **reprendre** reprend tous les processus qui ont été suspendus précédemment
  * **PS** PS affiche les processus en cours d’exécution au sein d’un conteneur

* Outils pour gérer l’état d’un conteneur
  * **État** de sortie de l’état d’un conteneur
  * **Kill** envoie le signal spécifié (par défaut : SIGTERM) au processus init du conteneur
  * **supprimer** supprime toutes les ressources détenues par le conteneur souvent utilisées avec le conteneur détaché

La seule commande qui peut être considérée comme plusieurs conteneurs est **List**.  Elle répertorie les conteneurs en cours d’exécution ou en pause démarrés par runhcs avec la racine donnée.

### <a name="hcs"></a>HCS

Deux wrappers sont disponibles sur GitHub pour l’interface avec HCS. Étant donné que HCS est une API C, les wrappers facilitent l’appel de HCS à partir de langages de niveau supérieur.  

* [hcsshim](https://github.com/microsoft/hcsshim) -hcsshim est écrit en Go et est la base de runhcs.
Récupérez le dernier à partir de AppVeyor ou générez-le vous-même.
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet-computevirtualization est un C# wrapper pour HCS.

Si vous souhaitez utiliser l’HCS (directement ou via un Wrapper), ou si vous souhaitez créer un wrapper de rouille/Haskell/InsertYourLanguage autour du HCS, laissez un commentaire.

Pour un examen approfondi de la HCS, regardez la [Présentation DockerCon de John](https://www.youtube.com/watch?v=85nCF5S8Qok)restante.

## <a name="containerdcri"></a>conteneur/élément de rapport personnalisé

> [!IMPORTANT]
> La prise en charge de l’élément de rapport personnalisé est disponible uniquement dans le serveur 2019/Windows 10 1809 et versions ultérieures.  Nous développons tout de même activement le conteneur pour Windows.
> Dev/test uniquement.

Alors que les spécifications OCI définissent un seul conteneur, l' [élément de rapport personnalisé](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (interface runtime de conteneur) décrit les conteneurs en tant que charges de travail dans un environnement de bac à sable (sandbox) partagé appelé pod.  Les Pod peuvent contenir une ou plusieurs charges de travail de conteneur.  Les gousses permettent aux orchestrateurs de conteneurs comme Kubernetes et Service Fabric maille de gérer des charges de travail groupées qui doivent se trouver sur le même hôte avec des ressources partagées telles que la mémoire et réseaux virtuels.

conteneur/élément personnalisé active la matrice de compatibilité suivante pour les Pod :

| Système d’exploitation hôte | Image du | Isolation | Prise en charge Pod |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | Oui : prend en charge les blocs de plusieurs conteneurs véritables. |
|  | Windows Server 2019/1809 | `process`* ou `hyperv` | Oui : prend en charge les blocs de plusieurs conteneurs véritables si chaque système d’exploitation de conteneur de charge de travail correspond au système d’exploitation de machine virtuelle utilitaire. |
|  | Windows Server 2016,</br>Windows Server 1709,</br>Windows Server 1803 | `hyperv` | Partielle : prend en charge les bacs à sable (sandbox) qui peuvent prendre en charge un seul conteneur isolé par processus par machine virtuelle d’utilitaire si le système d’exploitation du conteneur correspond au système d’exploitation de la machine virtuelle. |

\*les hôtes Windows 10 prennent uniquement en charge l’isolation Hyper-V

Liens vers la spécification de l’élément de rapport personnalisé :

* Spécification [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) -Pod
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) -spécification de charge de travail

![Environnements de conteneurs basés sur des conteneurs](media/containerd-platform.png)

Alors que les runHCS et les conteneurs peuvent tous les deux gérer sur n’importe quel serveur système Windows 2016 ou version ultérieure, la prise en charge des Pod (groupes de conteneurs) nécessitait des modifications avec rupture des outils de conteneur dans Windows.  La prise en charge de l’élément de rapport personnalisé est disponible sur Windows Server 2019/Windows 10 1809 et versions ultérieures.

---
title: Plateforme de conteneur Windows
description: En savoir plus sur les nouveaux blocs de construction de conteneur disponibles dans Windows.
keywords: LCOW, conteneurs Linux, ancrage, conteneurs, conteneurs, élément de rapportateur, runhcs, Runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 3107eb48dc9c75224b0c9dd9b436af6f0f451871
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998416"
---
# <a name="container-platform-tools-on-windows"></a>Outils de plateforme de conteneur sous Windows

La plateforme de conteneur Windows s’étend! La station d’accueil fut la première partie du voyage du conteneur, nous créons maintenant d’autres outils de plateforme de conteneur.

* [conteneur/élément personnalisé](https://github.com/containerd/cri) -nouveautés dans Windows Server 2019/Windows 10 1809.
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) -homologue d’hôte de conteneur Windows Runc.
* [HCS](https://docs.microsoft.com/virtualization/api/) : les services de calcul d’hôte + les shims pratiques pour faciliter leur utilisation.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

Cet article parle de la plateforme de conteneurs Windows et Linux ainsi que de chaque outil de plateforme de conteneur.

## <a name="windows-and-linux-container-platform"></a>Plateforme de conteneurs Windows et Linux

Dans les environnements Linux, les outils de gestion des conteneurs tels que les dockers sont basés sur un ensemble plus granulaire d’outils de conteneur: [Runc](https://github.com/opencontainers/runc) et [conteneur](https://containerd.io/).

![Architecture de l’ancrage sur Linux](media/docker-on-linux.png)

`runc` est un outil de ligne de commande Linux permettant de créer et d’exécuter des conteneurs conformément à la [spécification d’exécution du conteneur OCI](https://github.com/opencontainers/runtime-spec).

`containerd` est un démon qui gère le cycle de vie d’un conteneur de télécharger et de décompresser l’image du conteneur vers l’exécution et la surveillance du conteneur.

Sur Windows, nous avons adopté une approche différente.  Lorsque nous avons commencé à utiliser l’arrimeur pour prendre en charge les conteneurs Windows, nous avons intégré directement sur le HCS (Host Compute service).  [Ce billet de blog](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332) correspond à des informations sur les raisons pour lesquelles nous avons créé le HCS et pourquoi nous avons adopté cette approche aux conteneurs au départ.

![Architecture initiale du moteur de l’ancrage sur Windows](media/hcs.png)

À ce stade, l’arrimeur continue à appeler directement dans le HCS. En revanche, les outils de gestion des conteneurs qui s’étendent à l’inclusion de conteneurs Windows et de l’hôte de conteneur Windows pouvaient appeler le conteneur et runhcs la façon dont ils appellent le conteneur et Runc sur Linux.

## <a name="runhcs"></a>runhcs

`runhcs` est une fourche de `runc`.  Par `runc`exemple `runhcs` , il s’agit d’un client de ligne de commande pour exécuter des applications empaquetées conformément au format OCI (Open Container initiative) et est une implémentation conforme de la spécification d’initiative de conteneur ouvert.

Les différences fonctionnelles entre Runc et runhcs sont les suivantes:

* `runhcs` s’exécute sur Windows.  Il communique avec le [HCS](containerd.md#hcs) pour créer et gérer des conteneurs.
* `runhcs` peut exécuter différentes sortes de conteneurs.

  * [Isolement Hyper-V](../manage-containers/hyperv-container.md) Windows et Linux
  * Conteneurs de processus Windows (l’image de conteneur doit correspondre à l’hôte de conteneur)

**Utilisation:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` correspond à votre nom de l’instance de conteneur que vous démarrez. Ce nom doit être unique sur votre hôte de conteneur.

L’annuaire d’offres ( `-b bundle`en utilisant) est facultatif.  
Comme pour Runc, les conteneurs sont configurés à l’aide d’ensembles. Un paqueteur de conteneur est le répertoire contenant le fichier de spécification OCI du conteneur, «config. JSON».  La valeur par défaut de «Bundle» est le répertoire actif.

Le fichier spec OCI, «config. JSON», doit comporter deux champs qui s’exécutent correctement:

* Un chemin d’accès à l’espace de travail du conteneur
* Chemin d’accès au répertoire des couches du conteneur

Les commandes de conteneur disponibles dans runhcs sont les suivantes:

* Outils pour créer et exécuter un conteneur
  * **exécuter** crée et exécute un conteneur
  * **créer** un conteneur

* Outils permettant de gérer les processus en cours d’exécution dans un conteneur:
  * **commencer** exécute le processus défini par l’utilisateur dans un conteneur créé
  * **Exec** exécute un nouveau processus à l’intérieur du conteneur
  * **Pause** suspendre interrompt tous les processus à l’intérieur du conteneur
  * **reprise** reprise de tous les processus déjà en pause
  * **PS** PS affiche les processus en cours d’exécution au sein d’un conteneur

* Outils permettant de gérer l’état d’un conteneur
  * **État** génère l’état d’un conteneur
  * **Kill** envoie le signal spécifié (par défaut: SIGTERM) au processus init du conteneur.
  * la **suppression** supprime toutes les ressources détenues par le conteneur souvent utilisées avec le conteneur détaché.

La seule commande qui aurait pu être considérée comme plusieurs conteneurs est **liste**.  Il recense les conteneurs en cours d’exécution ou suspendus démarrés par runhcs avec la racine spécifiée.

### <a name="hcs"></a>HCS

Nous avons deux wrappers disponibles sur GitHub pour communiquer avec HCS. Dans la mesure où HCS est une API C, les wrappers permettent d’appeler facilement HCS à partir de langues de niveau supérieur.  

* [hcsshim](https://github.com/microsoft/hcsshim) -hcsshim est écrit dans Go et il s’agit de la base pour runhcs.
Procurez-vous la dernière version de AppVeyor ou créez-la vous-même.
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet-computevirtualization est un wrapper C# pour HCS.

Si vous souhaitez utiliser le HCS (directement ou via un Wrapper), ou si vous souhaitez créer un wrapper rouille/Haskell/InsertYourLanguage dans l’HCS, laissez un commentaire.

Pour obtenir une vue d’examen plus approfondie du HCS, regardez la [Présentation DockerCon de Jean](https://www.youtube.com/watch?v=85nCF5S8Qok).

## <a name="containerdcri"></a>contenant/personnalisé

> [!IMPORTANT]
> La prise en charge de l’élément personnalisé est uniquement disponible dans Server 2019/Windows 10 1809 et les versions ultérieures.  Il est également possible de développer activement des conteneurs pour Windows.
> Dev/test only.

Tandis que les spécifications OCI définissent un conteneur unique, l' [élément de rapport personnalisé](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (interface runtime du conteneur) décrit les conteneurs comme une charge de travail dans un environnement de bac à sable partagé appelé pod.  Les Pod peuvent contenir une ou plusieurs charges de travail de conteneur.  Les gousses permettent aux de conteneurs comme Kubernetes et de maille de service de maille de traiter des charges de travail groupées qui doivent se trouver sur le même hôte avec des ressources partagées, telles que la mémoire et vNETs.

conteneur/élément d’élément personnalisé active la matrice de compatibilité suivante pour les Pod:

| Système d’exploitation hôte | Système d’exploitation du conteneur | Problèmes | Support Pod |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | Oui, prend en charge des gousses multiples. |
|  | Windows Server 2019/1809 | `process`* ou `hyperv` | Oui, prend en charge des gousses multiples de conteneurs multiples si chaque système d’exploitation de conteneur de charge de travail correspond au système d’exploitation VM d’utilitaire. |
|  | Windows Server 2016,</br>Windows Server 1709,</br>Windows Server 1803 | `hyperv` | Partielle: prend en charge les bacs à sable (sandbox) qui peuvent prendre en charge un conteneur de processus unique par ordinateur virtuel, si le système d’exploitation de conteneur correspond au système d’exploitation de l’utilitaire VM. |

Les hôtes Windows 10 prennent uniquement en charge l’isolation Hyper-V

Liens vers la spécification du rapport de rapport d’élément:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) -Pod spec
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) -caractéristiques techniques de charge de travail

![Environnements de conteneur basés sur un conteneur](media/containerd-platform.png)

Bien que runHCS et conteneur puissent gérer sur n’importe quel système Windows Server 2016 ou version ultérieure, la prise en charge de gousses (groupes de conteneurs) nécessitait de changer les outils de conteneur dans Windows.  La prise en charge de l’élément de rapport personnalisé est disponible sur Windows Server 2019/Windows 10 1809 et les versions ultérieures.

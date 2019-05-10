---
title: Plateforme de conteneur Windows
description: En savoir plus sur les nouveaux conteneur blocs de construction disponibles dans Windows.
keywords: LCOW, des conteneurs linux, docker, conteneurs, containerd, élément de rapport personnalisé, runhcs, runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 74e22702aa4be30055b3f4f48c7fac926d793095
ms.sourcegitcommit: 03e9203e9769997d8be3f66dc7935a3e5c0a83e1
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621607"
---
# <a name="container-platform-tools-on-windows"></a>Outils de plateforme de conteneur sur Windows

La plateforme de conteneur Windows est développé! Docker a été la première partie du voyage conteneur, maintenant que nous créons des autres outils de plateforme de conteneur.

* [élément de rapport containerd/personnalisé](https://github.com/containerd/cri) - nouveau dans Windows Server 2019/Windows 10 1809.
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) - un équivalent d’hôte de conteneur Windows pour runc.
* [hcs](https://docs.microsoft.com/virtualization/api/) - le Service de calcul hôte + un shim pratique pour rendre plus facile à utiliser.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

Cet article parler de la plateforme de conteneur Windows et Linux, ainsi que chaque outil de plate-forme de conteneur.

## <a name="windows-and-linux-container-platform"></a>Plateforme de conteneur Windows et Linux

Dans les environnements Linux, les outils de gestion de conteneur comme Docker reposent sur un ensemble plus précis des outils de conteneur: [runc](https://github.com/opencontainers/runc) et [containerd](https://containerd.io/).

![Architecture de docker sur Linux](media/docker-on-linux.png)

`runc` est un outil de ligne de commande Linux pour la création et de conteneurs en fonction de la [spécification du runtime OCI conteneur](https://github.com/opencontainers/runtime-spec)en cours d’exécution.

`containerd` est un démon qui gère le cycle de vie de conteneur de télécharger et décompresser l’image de conteneur à l’exécution du conteneur et de surveillance.

Sur Windows, nous avons une approche différente.  Lorsque nous avons commencé à travailler avec Docker pour prendre en charge des conteneurs Windows, nous avons créé directement sur le HCS (Host Compute Service).  [Ce billet de blog](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332) est plein d’informations sur la raison pour laquelle nous avons créé le HCS et pourquoi nous avons effectué cette approche aux conteneurs initialement.

![Architecture de départ de moteur Docker sur Windows](media/hcs.png)

À ce stade, Docker appelle toujours directement dans le HCS. À l’avenir, toutefois, les outils de gestion conteneur étend pour inclure les conteneurs Windows et les fenêtres hôte de conteneur peut appeler containerd et runhcs la manière qu’ils appellent sur containerd et runc sur Linux.

## <a name="runhcs"></a>runhcs

`runhcs` représente une branche de `runc`.  Comme `runc`, `runhcs` est un client de ligne de commande pour exécuter des applications empaquetées en fonction du format ouvert conteneur Initiative (OCI) et est une implémentation conforme de la spécification ouverte Initiative de conteneur.

Différences fonctionnelles entre runc et runhcs sont les suivantes:

* `runhcs` s’exécute sur Windows.  Il communique avec le [HCS](containerd.md#hcs) pour créer et gérer des conteneurs.
* `runhcs` peut s’exécuter une variété de types de conteneurs différents.

  * Windows et Linux [isolation Hyper-V](../manage-containers/hyperv-container.md)
  * Windows traite les conteneurs (image de conteneur doit correspondre à l’hôte de conteneur)

**Utilisation:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` est le nom de l’instance du conteneur que Démarrer. Le nom doit être unique sur votre hôte de conteneur.

Le répertoire d’un ensemble d’applications (à l’aide de `-b bundle`) est facultative.  
À l’instar des runc, les conteneurs sont configurées à l’aide des ensembles d’applications. Un ensemble d’applications d’un conteneur est le répertoire contenant le fichier de spécification du conteneur OCI, «config.json».  La valeur par défaut pour «offre groupée» est le répertoire actif.

Le fichier spécification OCI, «config.json», doit avoir deux champs de s’exécuter correctement:

* Un chemin d’accès à l’espace de travail du conteneur
* Un chemin d’accès au répertoire de couche du conteneur

Commandes de conteneur disponibles dans runhcs sont les suivantes:

* Outils pour créer et exécuter un conteneur
  * **exécutez** crée et s’exécute un conteneur
  * **créer** de créer un conteneur

* Outils pour gérer le processus qui s’exécutent dans un conteneur:
  * **Démarrer** exécute le processus défini par l’utilisateur dans un conteneur créé
  * **Exec** s’exécute un nouveau processus à l’intérieur du conteneur
  * **Suspendre** la mise en pause interrompt l’exécution de tous les processus à l’intérieur du conteneur
  * **reprendre** reprend l’exécution de tous les processus qui ont été précédemment suspendus
  * **PS** ps affiche les processus en cours d’exécution à l’intérieur d’un conteneur

* Outils pour gérer l’état d’un conteneur
  * **état** génère l’état d’un conteneur
  * **Arrêter** envoie le signal spécifié (par défaut: SIGTERM) pour le processus d’initialisation du conteneur
  * **Delete** supprime toutes les ressources détenues par le conteneur souvent utilisé avec des conteneurs détachés

La seule commande qui peut être considérée comme conteneur multiples est la **liste**.  Il répertorie les conteneurs en cours d’exécution ou suspendus démarrés par runhcs avec la racine donnée.

### <a name="hcs"></a>HCS

Nous avons deux wrappers disponibles sur GitHub pour communiquer avec le HCS. Dans la mesure où le HCS est une API C, wrappers facilitent appeler le HCS à partir des langages de niveau supérieur.  

* [hcsshim](https://github.com/microsoft/hcsshim) - HCSShim est écrit dans l’emploi et il est à la base de runhcs.
Obtenir la dernière version à partir de AppVeyor ou créer une vous-même.
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet-computevirtualization est un wrapper pour le HCS en c#.

Si vous souhaitez utiliser le HCS (directement ou via un wrapper), ou si vous voulez rendre un wrapper rouille/Haskell/InsertYourLanguage autour de la HCS, veuillez laisser un commentaire.

Pour une vue plus approfondie le HCS, regardez la [Présentation de DockerCon de John Stark](https://www.youtube.com/watch?v=85nCF5S8Qok).

## <a name="containerdcri"></a>élément de rapport containerd/personnalisé

> [!IMPORTANT]
> Prise en charge de l’élément de rapport personnalisé est uniquement disponible dans Server 2019/Windows 10 1809 et versions ultérieures.  Nous développons également activement containerd pour Windows.
> Développement/test uniquement.

Tandis que les spécifications OCI définit un conteneur unique, [élément de rapport personnalisé](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (interface de runtime de conteneur) décrit les conteneurs en tant que workload(s) dans un bac à sable partagé environnement appelé un pod.  PODS peuvent contenir un ou plusieurs charges de travail de conteneur.  PODS permettent d’orchestrateurs comme Kubernetes et Service Fabric maillage gérer des charges de travail groupées qui doivent se trouver sur le même hôte avec certaines ressources partagées, telles que la mémoire et vNETs.

containerd/élément de rapport personnalisé permet la matrice de compatibilité suivante pour pods:

| Système d’exploitation hôte | Conteneur du système d’exploitation | Isolation | Prise en charge de POD? |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | Oui, prend en charge true multiconteneurs pods. |
|  | Windows Server 2019/1809 | `process`* ou `hyperv` | Oui, prend en charge la valeur true multiconteneurs pods si chaque conteneur de charge de travail du système d’exploitation correspond à l’utilitaire de système d’exploitation de l’ordinateur virtuel. |
|  | Windows Server 2016,</br>Windows Server 1709,</br>Windows Server 1803 | `hyperv` | Partielle, prend en charge de pod bacs à sable pouvant prendre en charge un conteneur isolé du processus unique par l’utilitaire VM si le système d’exploitation de conteneur correspond à l’utilitaire de système d’exploitation de l’ordinateur virtuel. |

Hôtes \*Windows 10 prennent uniquement en charge l’isolation Hyper-V

Liens vers la spécification de l’élément de rapport personnalisé:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) - Pod spécifications
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) - spécifications de la charge de travail

![Containerd en fonction des environnements de conteneur](media/containerd-platform.png)

Tandis que runHCS et containerd peuvent gérer sur n’importe quel système Windows Server 2016 ou version ultérieure, prise en charge des Pods (groupes de conteneurs) requise changements importants sur les outils de conteneur dans Windows.  Prise en charge de l’élément de rapport personnalisé est disponible sur Windows Server 2019/Windows 10 1809 et versions ultérieures.

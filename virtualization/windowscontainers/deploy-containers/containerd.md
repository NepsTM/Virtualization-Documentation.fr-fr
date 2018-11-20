---
title: Création de la pile de conteneur
description: En savoir plus sur le nouveau conteneur blocs de construction disponibles dans Windows.
keywords: LCOW, des conteneurs linux, docker, conteneurs, containerd, élément de rapport personnalisé, runhcs, runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 8a68bf9e5e78add65aedb51fff8521ee258e353e
ms.sourcegitcommit: 9a61fc06c25d17ddb61124a21a3ca821828b833d
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2018
ms.locfileid: "7460494"
---
# <a name="container-platform-tools-on-windows"></a>Outils de plateforme de conteneur sur Windows

La plateforme de conteneur Windows est développé!  Docker a été la première partie du voyage conteneur, maintenant que nous créons des autres outils de plateforme de conteneur.

1. [élément de rapport containerd/personnalisé](https://github.com/containerd/cri) - nouveau dans Windows Server 2019 et Windows 10 1809.
1. [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) - un équivalent d’hôte de conteneur Windows pour runc.
1. [hcs](https://docs.microsoft.com/virtualization/api/) - le Service de calcul hôte + un shim pratique pour rendre plus facile à utiliser.

    * [hcsshim](https://github.com/microsoft/hcsshim)
    * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

Cet article parler de la plateforme de conteneur Windows et Linux, ainsi que chaque outil de plate-forme de conteneur.

## <a name="windows-and-linux-container-platform"></a>Plateforme de conteneur Windows et Linux

Dans les environnements de Linux, les outils de gestion de conteneur comme Docker sont appuient sur un ensemble plus précis des outils de conteneur - [runc](https://github.com/opencontainers/runc) et [containerd](https://containerd.io/).

![Architecture de docker sur Linux](media/docker-on-linux.png)

`runc` est un outil de ligne de commande Linux pour la création et de conteneurs en fonction de la [spécification du runtime OCI conteneur](https://github.com/opencontainers/runtime-spec)en cours d’exécution.

`containerd` est un démon qui gère le cycle de vie de conteneur de télécharger et décompresser l’image de conteneur par le biais de l’exécution du conteneur et de surveillance.

Sur Windows, nous avons effectué une approche différente.  Lorsque nous avons commencé à travailler avec Docker pour prendre en charge des conteneurs Windows, nous avons créé directement sur le HCS (Host Compute Service).  [Ce billet de blog](https://blogs.technet.microsoft.com/virtualization/2017/01/27/introducing-the-host-compute-service-hcs/) est pleine d’informations sur la raison pour laquelle nous avons créé le HCS et pourquoi nous avons effectué cette approche aux conteneurs initialement.

![Architecture de départ de moteur Docker sur Windows](media/hcs.png)

À ce stade, Docker appelle toujours directement dans le HCS. À l’avenir, toutefois, les outils de gestion conteneur étend pour inclure les conteneurs Windows et les fenêtres hôte de conteneur peut appeler containerd et runhcs la manière qu’ils appellent sur containerd et runc sur Linux.

## <a name="runhcs"></a>runhcs

RunHCS est une branche de runc.  Comme runc, runhcs est un client de ligne de commande pour exécuter des applications empaquetées en fonction du format ouvert conteneur Initiative (OCI) et une implémentation conforme de la spécification ouverte Initiative de conteneur.  

Différences fonctionnelles entre runc et runhcs sont les suivantes:

* runhcs s’exécute sur Windows
* runhcs peut s’exécuter à la fois Windows et Linux [conteneurs Hyper-V](../manage-containers/hyperv-container.md) en plus des conteneurs de processus de Windows.

**Utilisation:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` est le nom de l’instance de conteneur que Démarrer. Le nom doit être unique sur votre hôte de conteneur.

Le répertoire de l’ensemble d’applications (à l’aide de `-b bundle`) est facultative.  
À l’instar de runc, les conteneurs sont configurées à l’aide des ensembles d’applications. Un ensemble d’applications d’un conteneur est le répertoire contenant le fichier de spécification du conteneur OCI, «config.json».  La valeur par défaut pour «offre groupée» est le répertoire actif.

Le fichier de spécifications OCI, le «config.json», doit avoir deux champs de s’exécuter correctement:

1. Un chemin d’accès à l’espace de travail du conteneur
1. Un chemin d’accès au répertoire de couche du conteneur

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

La seule commande peut être considérée comme conteneur multiples est la **liste**.  Il répertorie les conteneurs en cours d’exécution (ou mis en pause) démarrés par runhcs avec la racine donnée.

## <a name="containerdcri"></a>élément de rapport containerd/personnalisé

> ! Prise en charge de l’élément de rapport Remarque personnalisé est uniquement disponible dans Server 2019/Windows 10 1809 et versions ultérieures.

Alors que les spécifications OCI définit un conteneur unique, [élément de rapport personnalisé](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (interface de runtime de conteneur) décrit les conteneurs en tant que workload(s) dans un bac à sable partagé environnement appelé un pod.  PODS peuvent contenir un ou plusieurs charges de travail de conteneur.  PODS permettent orchestrateurs comme Kubernetes et le maillage de Service Fabric gérer des charges de travail groupées qui doivent se trouver sur le même hôte avec certaines ressources partagées, telles que la mémoire et vNETs.

Liens vers la spécification de l’élément de rapport personnalisé:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) - Pod spécification
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) - spécifications de la charge de travail

![Containerd en fonction des environnements de conteneur](media/containerd-platform.png)

Tandis que runHCS et containerd peuvent gérer sur n’importe quel système Windows Server 2016 ou version ultérieure, prenant en charge les Pods (groupes de conteneurs) requise Nouveautés pour les outils de conteneur dans Windows.  Prise en charge de l’élément de rapport personnalisé est disponible sur Windows Server 2019 et Windows 10 1809 et versions ultérieures.

## <a name="hcs"></a>HCS

Nous avons deux wrappers disponibles sur GitHub d’interface avec le HCS. Dans la mesure où le HCS est une API C, wrappers facilitent appeler le HCS à partir des langages de niveau supérieur.  

### <a name="hcsshim"></a>HCSShim

HCSShim est écrit en Go et il est à la base de runhcs.
Obtenir la dernière version à partir de AppVeyor ou créer une vous-même.

Essayez-la dans [GitHub](https://github.com/microsoft/hcsshim).

### <a name="dotnet-computevirtualization"></a>dotnet-computevirtualization

> ! Remarque Il s’agit d’une implémentation de référence - l’utiliser pour le développement/test uniquement.

dotnet-computevirtualization est un wrapper pour le HCS en c#.

Essayez-la sur [GitHub](https://github.com/microsoft/dotnet-computevirtualization).

Si vous souhaitez utiliser le HCS (directement ou via un wrapper), ou si vous voulez rendre un wrapper rouille/Haskell/InsertYourLanguage autour de la HCS, veuillez laisser un commentaire.

Pour une vue plus approfondie le HCS, regardez la [Présentation de DockerCon de John Stark](https://www.youtube.com/watch?v=85nCF5S8Qok).
---
title: Compilation de fichiers binaires Kubernetes
author: daschott
ms.author: daschott
ms.date: 08/13/2020
ms.topic: how-to
description: Compilation et compilation croisée des fichiers binaires Kubernetes à partir de la source.
keywords: kubernetes, 1,12, Linux, compiler
ms.openlocfilehash: 21ccee749956abc2ffb02a7b2ec1394c1195fe09
ms.sourcegitcommit: aa139e6e77a27b8afef903fee5c7ef338e1c79d4
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/15/2020
ms.locfileid: "88251592"
---
# <a name="compiling-kubernetes-binaries"></a>Compilation de fichiers binaires Kubernetes

La compilation de Kubernetes requiert un environnement Go de travail. Cette page passe en revue plusieurs façons de compiler les fichiers binaires Linux et d’effectuer une compilation croisée des binaires Windows.
> [!NOTE]
> Cette page est entièrement volontaire et incluse uniquement pour les développeurs intéressés Kubernetes qui veulent expérimenter la dernière & code source le plus récent.

> [!tip]
> Pour recevoir des notifications sur les dernières évolutions auxquelles vous pouvez vous abonner [@kubernetes-announce](https://groups.google.com/forum/#!forum/kubernetes-announce) .

## <a name="installing-go"></a>Installation de Go

Par souci de simplicité, cela implique d’installer Go dans un emplacement temporaire et personnalisé :

```bash
cd ~
wget https://redirector.gvt1.com/edgedl/go/go1.11.1.linux-amd64.tar.gz -O go1.11.1.tar.gz
tar -vxzf go1.11.1.tar.gz
mkdir gopath
export GOROOT="$HOME/go"
export GOPATH="$HOME/gopath"
export PATH="$GOROOT/bin:$PATH"
```

> [!NOTE]
> Cela définit les variables d’environnement pour votre session. Ajoutez `export` à votre `~/.profile`pour un réglage permanent.

Exécutez `go env`pour vérifier que les chemins ont été correctement définis. Il existe plusieurs options pour la création des fichiers binaires Kubernetes :

  - Générez-les [localement ](#building-locally).
  - Générez les fichiers binaires à l’aide de [Vagrant ](#build-with-vagrant).
  - Exploitez les [scripts de compilation conteneurisés standard](https://github.com/kubernetes/kubernetes/tree/master/build#key-scripts) du projet Kubernetes. Pour ce faire, suivez les étapes de [création en local](#building-locally) jusqu’aux étapes `make`, puis suivez les instructions liées.

Pour copier les binaires Windows sur leurs nœuds respectifs, utilisez un outil graphique tel que [WinSCP](https://winscp.net/eng/download.php) ou un outil de ligne de commande comme [pscp](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) pour les transférer vers le répertoire `C:\k`.

## <a name="building-locally"></a>Créer localement

> [!TIP]
> Si vous rencontrez des erreurs « autorisation refusée », vous pouvez les éviter en générant Linux en `kubelet` premier, conformément à la note indiquée dans [`acs-engine`](https://github.com/Azure/acs-engine/blob/master/scripts/build-windows-k8s.sh#L176) :
>
> _En raison de ce qui semble être un bogue dans le système de génération Windows Kubernetes, il faut d’abord créer un fichier binaire Linux à générer `_output/bin/deepcopy-gen` . Si vous générez sur Windows avec cette opération, vous obtiendrez un vide `deepcopy-gen` ._

En premier lieu, récupérez l’espace de stockage Kubernetes :

```bash
KUBEREPO="k8s.io/kubernetes"
go get -d $KUBEREPO
# Note: the above command may spit out a message about
#       "no Go files in...", but it can be safely ignored!
cd $GOPATH/src/$KUBEREPO
```

Maintenant, vérifiez la branche à partir de laquelle effectuer la compilation et construire le fichier binaire Linux `kubelet`. Cela est nécessaire pour éviter les erreurs de génération Windows indiquées ci-avant. Ici, nous allons utiliser `v1.12.2`. Après le `git checkout`, vous pourrez appliquer les réservations permanentes en attente, les correctifs, ou apporter d’autres modifications aux fichiers binaires personnalisés.

```bash
git checkout tags/v1.12.2
make clean && make WHAT=cmd/kubelet
```

Pour finir, créez les fichiers binaires client Windows nécessaires (la dernière étape peut varier, en fonction d’où les fichiers binaires Windows doivent être récupérés ultérieurement) :

```bash
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubectl
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kube-proxy
cp _output/local/bin/windows/amd64/kube*.exe ~/kube-win/
```

Les étapes de création des fichiers binaires Linux sont identiques ; laissez le préfixe  `KUBE_BUILD_PLATFORMS=windows/amd64` pour les commandes. Le répertoire de sortie sera en l’occurrence `_output/.../linux/amd64`.

## <a name="build-with-vagrant"></a>Création avec Vagrant

Il existe un programme d’installation Vagrant [ici ](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux/vagrant). Utilisez-le pour préparer un ordinateur virtuel Vagrant, puis exécutez ces commandes qu’il contient :

```bash
DIST_DIR="${HOME}/kube/"
SRC_DIR="${HOME}/src/k8s-main/"
mkdir ${DIST_DIR}
mkdir -p "${SRC_DIR}"

git clone https://github.com/kubernetes/kubernetes.git ${SRC_DIR}

cd ${SRC_DIR}
git checkout tags/v1.12.2
KUBE_BUILD_PLATFORMS=linux/amd64   build/run.sh make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kube-proxy
cp _output/dockerized/bin/windows/amd64/kube*.exe ${DIST_DIR}

ls ${DIST_DIR}
```


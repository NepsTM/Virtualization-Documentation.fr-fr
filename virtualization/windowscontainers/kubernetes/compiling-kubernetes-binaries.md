---
title: Compilation de fichiers binaires Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Compilation et compilation croisée des fichiers binaires Kubernetes à partir de la source.
keywords: kubernetes, 1.12, linux, compiler
ms.openlocfilehash: 40bf7e65a8910cdab095abb269aa0a92508189cd
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178872"
---
# <a name="compiling-kubernetes-binaries"></a>Compilation de fichiers binaires Kubernetes #
La compilation de Kubernetes requiert un environnement Go de travail. Cette page passe en revue plusieurs façons de compiler les fichiers binaires Linux et d’effectuer une compilation croisée des binaires Windows.
> [!NOTE] 
> Cette page est complètement volontaires et uniquement inclus pour les développeurs de Kubernetes intéressés qui souhaitent faire des essais avec le code source d’avant-garde.

> [!tip]
> Pour recevoir des notifications sur les dernières améliorations, vous pouvez vous abonner à [@kubernetes-announce](https://groups.google.com/forum/#!forum/kubernetes-announce).

## <a name="installing-go"></a>Installation de Go ##
Par souci de simplicité, cela implique d’installer Go dans un emplacement temporaire et personnalisé:

```bash
cd ~
wget https://redirector.gvt1.com/edgedl/go/go1.11.1.linux-amd64.tar.gz -O go1.11.1.tar.gz
tar -vxzf go1.11.1.tar.gz
mkdir gopath
export GOROOT="$HOME/go"
export GOPATH="$HOME/gopath"
export PATH="$GOROOT/bin:$PATH"
```

> [!Note]  
> Cela définit les variables d’environnement pour votre session. Ajoutez `export` à votre `~/.profile`pour un réglage permanent.

Exécutez `go env`pour vérifier que les chemins ont été correctement définis. Il existe plusieurs options pour la création des fichiers binaires Kubernetes:

  - Générez-les [localement ](#build-locally).
  - Générez les fichiers binaires à l’aide de [Vagrant ](#build-with-vagrant).
  - Exploitez les [scripts de compilation conteneurisés standard](https://github.com/kubernetes/kubernetes/tree/master/build#key-scripts) du projet Kubernetes. Pour ce faire, suivez les étapes de [création en local](#build-locally) jusqu’aux étapes `make`, puis suivez les instructions liées.

Pour copier les binaires Windows sur leurs nœuds respectifs, utilisez un outil graphique tel que [WinSCP](https://winscp.net/eng/download.php) ou un outil de ligne de commande comme [pscp](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) pour les transférer vers le répertoire `C:\k`.


## <a name="building-locally"></a>Construction en local ##
> [!Tip]  
> Si vous rencontrez des erreurs «autorisation refusée», elles peuvent être évitées en créant tout d’abord le Linux `kubelet`, selon la note dans [`acs-engine`](https://github.com/Azure/acs-engine/blob/master/scripts/build-windows-k8s.sh#L176):
>  
> _En raison de ce qui semble être un bogue dans le système de génération Kubernetes Windows, il faut d’abord créer un fichier binaire Linux pour générer `_output/bin/deepcopy-gen`. La création sur Windows sans effectuer cette opération génère un `deepcopy-gen` vide._

En premier lieu, récupérez l’espace de stockage Kubernetes:

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

Pour finir, créez les fichiers binaires client Windows nécessaires (la dernière étape peut varier, en fonction d’où les fichiers binaires Windows doivent être récupérés ultérieurement):

```bash
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubectl
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kube-proxy
cp _output/local/bin/windows/amd64/kube*.exe ~/kube-win/
```

Les étapes de création des fichiers binaires Linux sont identiques; laissez le préfixe  `KUBE_BUILD_PLATFORMS=windows/amd64` pour les commandes. Le répertoire de sortie sera en l’occurrence `_output/.../linux/amd64`.


## <a name="build-with-vagrant"></a>Création avec Vagrant ##
Il existe un programme d’installation Vagrant [ici ](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux/vagrant). Utilisez-le pour préparer un ordinateur virtuel Vagrant, puis exécutez ces commandes qu’il contient:

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


---
title: "Nœud maître (Master) Kubernetes à partir de zéro"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "Création d’un cluster maître Kubernetes à partir de zéro."
keywords: kubernetes, 1,9, master, linux
ms.openlocfilehash: 3ea338f7af3dd921731fce0ec5a8b2cf8c4fef0c
ms.sourcegitcommit: f542e8c95b5bb31b05b7c88f598f00f76779b519
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/01/2018
---
# <a name="kubernetes-master--from-scratch"></a>Nœud maître Kubernetes à partir de zéro #
Cette page détaille le déploiement manuel d’un nœud maître Kubernetes du début à la fin.

Une machine Linux de type Ubuntu mise à jour récemment est requise pour poursuivre la procédure. Windows n’est pas du tout utilisé dans ce cas de figure. La compilation croisée des fichiers binaires s’effectue sous Linux.


> [!Warning]  
> En raison de la volatilité de Kubernetes de version en version, ce guide est susceptible d’émettre des hypothèses qui ne seront pas vraies à l’avenir.


## <a name="preparing-the-master"></a>Préparation du Master ##
Tout d’abord, installez tous les composants prérequis:

```bash
sudo apt-get install curl git build-essential docker.io conntrack python2.7
```

Si vous vous trouvez derrière un proxy, définissez des variables d’environnement pour la session active:
```bash
HTTP_PROXY=http://proxy.example.com:80/
HTTPS_PROXY=http://proxy.example.com:443/
http_proxy=http://proxy.example.com:80/
https_proxy=http://proxy.example.com:443/
```
Autrement, si vous souhaitez rendre ce paramètre permanent, ajoutez les variables à /etc/environment (il est nécessaire de se déconnecter puis de se reconnecter pour appliquer les modifications).

Il existe une collection de scripts dans [ce référentiel](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux) qui vous aidera au cours du processus d’installation. Consultez-les pour `~/kube/`. L’intégralité de ce répertoire sera monté pour la plupart des conteneurs Docker au cours des prochaines étapes, veillez-donc à conserver sa structure comme indiqué dans le guide.

```bash
mkdir ~/kube
mkdir ~/kube/bin
git clone https://github.com/Microsoft/SDN /tmp/k8s 
cd /tmp/k8s/Kubernetes/linux
chmod -R +x *.sh
chmod +x manifest/generate.py
mv * ~/kube/
```


### <a name="installing-the-linux-binaries"></a>Installation des fichiers binaires Linux ###

> [!Note]  
> Pour inclure des correctifs ou utiliser un code Kubernetes expérimental plutôt que de télécharger les fichiers binaires prédéfinis, veuillez consulter [cette page](./compiling-kubernetes-binaries.md).

Télécharger et installer les fichiers binaires Linux officiels depuis le [répertoire Kubernetes principal](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1) et les installer comme suit:

```bash
wget -O kubernetes.tar.gz https://github.com/kubernetes/kubernetes/releases/download/v1.9.1/kubernetes.tar.gz
tar -vxzf kubernetes.tar.gz 
cd kubernetes/cluster 
# follow the prompts from this command, the defaults are generally fine:
./get-kube-binaries.sh
cd ../server
tar -vxzf kubernetes-server-linux-amd64.tar.gz 
cd kubernetes/server/bin
cp hyperkube kubectl ~/kube/bin/
```

Ajouter les fichiers binaires à `$PATH`, de sorte à pouvoir les exécuter depuis n’importe quel emplacement. Notez que cette action définit le chemin d’accès uniquement pour la session. Ajoutez cette ligne `~/.profile` pour que cette action devienne permanente.

```bash
$ PATH="$HOME/kube/bin:$PATH"
```

### <a name="install-cni-plugins"></a>Installer les plug-ins CNI ###
Les plug-ins CNI de base sont requis pour une mise en réseau Kubernetes. Ils peuvent être téléchargés [ici](https://github.com/containernetworking/plugins/releases) et doit être extraits vers `/opt/cni/bin/`:

```bash
DOWNLOAD_DIR="${HOME}/kube/cni-plugins"
CNI_BIN="/opt/cni/bin/"
mkdir ${DOWNLOAD_DIR}
cd $DOWNLOAD_DIR
curl -L $(curl -s https://api.github.com/repos/containernetworking/plugins/releases/latest | grep browser_download_url | grep 'amd64.*tgz' | head -n 1 | cut -d '"' -f 4) -o cni-plugins-amd64.tgz
tar -xvzf cni-plugins-amd64.tgz
sudo mkdir -p ${CNI_BIN}
sudo cp -r !(*.tgz) ${CNI_BIN}
ls ${CNI_BIN}
```


### <a name="certificates"></a>Certificats ###
Tout d’abord, obtenir l’adresse IP locale, soit via `ifconfig`, soit:

```bash
$ ip addr show dev eth0
```

si le nom de l’interface est connu. Il y sera fait référence de nombreuses fois au long de ce processus. Le définir en tant que variable d’environnement facilite les choses. L’extrait de code suivant le définit temporairement; si la session se termine ou si le shell se ferme, il doit être défini à nouveau.

```bash
$ MASTER_IP=10.123.45.67   # example! replace
```

Préparer les certificats qui serviront de nœuds pour communiquer dans le cluster:

```bash
cd ~/kube/certs
chmod u+x generate-certs.sh
./generate-certs.sh $MASTER_IP
```

### <a name="prepare-manifests--addons"></a>Préparer les manifestes et les extensions ###
Générer un ensemble de fichiers YAML qui spécifient les pods système de Kubernetes en transmettant l’adresse IP Master et l’adressage CIDR *complet* du cluster au script Python dans le dossier `manifest`:

```bash
cd ~/kube/manifest
./generate.py $MASTER_IP --cluster-cidr 192.168.0.0/16
```

Déplacer le script Python afin que Kubernetes ne le confonde pas avec un manifeste; des problèmes pourront survenir plus tard si cette action n’est pas effectuée.

> [!Important]  
> Si la version actuelle de Kubernetes est différente de celle mentionnée dans ce guide, utilisez les différents indicateurs de version présents dans le script (tel que `--api-version`) pour [personnaliser l’image](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL/hyperkube-amd64) qui déploient les pods. Tous les manifestes n’utilisent pas la même image et peuvent posséder des schémas de version différents (notamment, `etcd` et le Gestionnaire de module complémentaire).


#### <a name="manifest-customization"></a>Personnalisation du manifeste ####
À ce stade, des modifications spécifiques à la configuration peuvent être souhaitables. Par exemple, il est peut-être nécessaire d’assigner manuellement les sous-réseaux aux nœuds, plutôt que de laisser Kubernetes les gérer automatiquement. Cette configuration spécifique dispose d’une option dans le script (voir `--help` pour plus d’explications concernant le paramètre `--im-sure`):

```bash
./generate.py $MASTER_IP --im-sure
```

Les autres options de configuration personnalisée nécessitent une modification manuelle des manifestes générés.


### <a name="configure--run-kubernetes"></a>Configurer et exécuter Kubernetes ###
Configurer Kubernetes pour utiliser les certificats générés. Cette action permet de créer une configuration à l’emplacement `~/.kube/config`:

```bash
cd ~/kube
./configure-kubectl.sh $MASTER_IP
```

À présent, copier le fichier à l’emplacement où les pods le chercheront plus tard:

```bash
mkdir ~/kube/kubelet
sudo cp ~/.kube/config ~/kube/kubelet/
```

Le «client» Kubernetes, `kubelet`, est prêt à démarrer. Les scripts suivants fonctionnent indéfiniment; ouvrez une autre session terminal après chacun d’’entre eux pour qu’ils continuent de fonctionner:

```bash
cd ~/kube
sudo ./start-kubelet.sh
```

Exécutez le script Kubeproxy, en passant le CIDR partiel du cluster:

```bash
cd ~/kube
sudo ./start-kubeproxy.sh 192.168
```


> [!Important]  
> Ce sera le /16 CIDR *complet* prévu qui gérera les nœuds, *même s’il existe un trafic autre que Kubernetes sur ce CIDR.* Kubeproxy s’applique *uniquement* au trafic Kubernetes vers le sous-réseau de *service*, afin qu’il n’interfère pas avec le trafic des autres hôtes.

> [!Note]  
> Ces scripts peuvent être mis en arrière-plan. Ce guide couvre uniquement leur exécution manuelle, étant donné qu’il s’agit de la manière la plus pratique pour déceler les erreurs lors de l’installation.


## <a name="verifying-the-master"></a>Vérification du Master ##
Après quelques minutes, le système doit être dans l’état suivant:

  - Sous `docker ps`, il y aura ~ 23nœuds de travail et conteneurs de pod.
  - Appeler `kubectl cluster-info` affichera les informations concernant le serveur d’API Master Kubernetes, en plus du DNS et des extensions Heapster.
  - `ifconfig` Affichera une nouvelle interface `cbr0` avec le CIDR du cluster choisi.


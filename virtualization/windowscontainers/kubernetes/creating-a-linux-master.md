---
title: Nœud maître (Master) Kubernetes à partir de zéro
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Création d’un Kubernetes cluster maître.
keywords: kubernetes, 1,13, linux maître,
ms.openlocfilehash: 8a3fb073616d115ab84e6cc36f0fb6cedbcf1f7d
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120427"
---
# <a name="creating-a-kubernetes-master"></a>Création d’un nœud maître Kubernetes #
> [!NOTE]
> Ce guide a été validé sur v1.13 Kubernetes. En raison de la volatilité de Kubernetes à partir de la version vers la version, cette section peut émettre des hypothèses qui ne possèdent pas la valeur true pour toutes les versions futures. La documentation officielle de l’initialisation de masques de Kubernetes à l’aide de kubeadm trouverez [ici](https://kubernetes.io/docs/setup/independent/install-kubeadm/). Il suffit d’activer de [section de planification de systèmes d’exploitation mixtes](#enable-mixed-os-scheduling) .

> [!NOTE]  
> Une machine Linux récemment mis à jour est nécessaire pour suivre le long; Kubernetes maître des ressources telles que les [kube-dns](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), [kube-planificateur](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)et [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) n'ont pas été transférées vers Windows encore. 

> [!tip]
> Les instructions de Linux sont spécifiquement vers **16.04 Ubuntu**. Autres versions de Linux certifiées pour s’exécuter Kubernetes offrent également des commandes équivalentes que vous pouvez remplacer. Ils seront également interagir avec succès avec Windows.


## <a name="initialization-using-kubeadm"></a>Initialisation à l’aide de kubeadm ##
Sauf spécification explicite dans le cas contraire, exécutez les commandes ci-dessous en tant que **racine**.

Tout d’abord, obtenez dans un interpréteur de commandes avec élévation de privilèges racine:

```bash
sudo –s
```

Assurez-vous que votre ordinateur est à jour:

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>Installer Docker ###
Pour être en mesure d’utiliser les conteneurs, vous avez besoin d’un moteur de conteneur, par exemple, Docker. Pour obtenir la version la plus récente, vous pouvez utiliser [ces instructions](https://docs.docker.com/install/linux/docker-ce/ubuntu/) pour l’installation de Docker. Vous pouvez vérifier que docker est correctement installé en exécutant un `hello-world` conteneur:

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>Installer kubeadm ###
Télécharger `kubeadm` fichiers binaires pour votre distribution Linux et initialiser votre cluster.

> [!Important]  
> En fonction de votre distribution Linux, vous devrez peut-être remplacer `kubernetes-xenial` sous le [nom de code](https://wiki.ubuntu.com/Releases)de correct.

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>Préparer le nœud maître ###
Kubernetes sur Linux requiert échange d’espace pour être désactivé:

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>Initialiser master ###
Notez votre sous-réseau de cluster (par exemple, 10.244.0.0/16) et le sous-réseau de service (par exemple, 10.96.0.0/12) et initialiser votre maître à l’aide de kubeadm:

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

Ce processus peut prendre plusieurs minutes. Une fois terminé, vous devriez voir un écran comme cette confirmation que votre maître a été initialisé:

![texte](media/kubeadm-init.png)

> [!tip]
> Vous devez prendre note de cette commande de jointure kubeadm. Devraient expirer le jeton kubeadm, vous pouvez utiliser `kubeadm token create --print-join-command` pour créer un nouveau jeton.

> [!tip]
> Si vous disposez d’une version de Kubernetes souhaitée que vous souhaitez utiliser, vous pouvez passer le `--kubernetes-version` indicateur pour kubeadm.

Nous n’avons pas encore terminé. Pour utiliser `kubectl` en tant qu’un utilisateur normal, exécutez le suivant __**dans un interpréteur de commandes utilisateur unelevated, non racine**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Maintenant, vous pouvez utiliser kubectl pour modifier ou afficher des informations sur votre cluster.

### <a name="enable-mixed-os-scheduling"></a>Activer la planification des systèmes d’exploitation mixtes ###
Par défaut, certaines ressources Kubernetes sont écrites de façon qu’ils sont planifiés sur tous les nœuds. Toutefois, dans un environnement multi-OS nous ne voulons pas les ressources de Linux à interférer ou être planifiée double sur les nœuds de Windows et vice versa. Pour cette raison, nous devons appliquer des étiquettes [NodeSelector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) . 

À cet égard, nous allons pour corriger les linux kube-proxy [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) à cible Linux uniquement.

Tout d’abord, nous allons créer un répertoire pour stocker les fichiers manifeste .yaml:
```bash
mkdir -p kube/yaml && cd kube/yaml
```

Vérifiez que la stratégie de mise à jour de `kube-proxy` DaemonSet est défini sur [RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/):

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

Ensuite, le DaemonSet des correctifs en téléchargeant [ce nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) et l’appliquer pour cibler uniquement Linux:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

Une fois qu’effectué avec succès, vous devez voir «Sélecteurs de nœuds» de `kube-proxy` et n’importe quel autre DaemonSets défini sur `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![texte](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>Collecter des informations de cluster ###
Pour joindre avec succès des nœuds futures à du master, vous devez suivre des informations suivantes:
  1. `kubeadm join` commande de sortie ([ici](#initialize-master))
    * Exemple: `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. Définis au cours de sous-réseau de cluster `kubeadm init` ([ici](#initialize-master))
    * Exemple: `10.244.0.0/16`
  3. Sous-réseau de service défini au cours de `kubeadm init` ([ici](#initialize-master))
    * Exemple: `10.96.0.0/12`
    * Vous pouvez également trouver à l’aide de `kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. Adresse IP de service Kube-dns 
    * Exemple: `10.96.0.10`
    * Se trouvent dans le champ «Cluster IP» à l’aide `kubectl get svc/kube-dns -n kube-system`
  5. Kubernetes `config` fichier généré après `kubeadm init` ([ici](#initialize-master)). Si vous avez suivi les instructions, cela peut trouver dans les chemins d’accès suivants:
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>Vérification du master ##
Après quelques minutes, le système doit être dans l’état suivant:

  - Sous `kubectl get pods -n kube-system`, il y aura pods pour les [composants de maître Kubernetes](https://kubernetes.io/docs/concepts/overview/components/#master-components) dans `Running` état.
  - L’appel `kubectl cluster-info` affichera les informations concernant le serveur d’API master Kubernetes en plus des modules complémentaires DNS.
  
> [!tip]
> Dans la mesure où kubeadm ne pas configuré mise en réseau, DNS pods peuvent encore se trouver dans `ContainerCreating` ou `Pending` état. Ils basculeront sur `Running` état après avoir [choisi une solution de réseau](./network-topologies.md).

## <a name="next-steps"></a>Étapes suivantes ## 
Dans cette section, nous décrit comment configurer un nœud maître Kubernetes à l’aide de kubeadm. Vous êtes maintenant prêt à l’étape 3:

> [!div class="nextstepaction"]
> [Choix d’une solution de réseau](./network-topologies.md)
---
title: Nœud maître (Master) Kubernetes à partir de zéro
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Création d’un Kubernetes cluster maître.
keywords: kubernetes, 1.12, linux maître,
ms.openlocfilehash: 2bbcf2d382f20d140c73d9b34cf0f13a74debdfa
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178852"
---
# <a name="creating-a-kubernetes-master"></a>Création d’un nœud maître Kubernetes #
> [!NOTE]
> Ce guide a été validé sur v1.12 de Kubernetes. En raison de la volatilité de Kubernetes à partir de la version vers la version, cette section peut-être émettre des hypothèses qui ne possèdent pas la valeur trues pour toutes les versions futures. Vous pouvez trouver la documentation officielle de l’initialisation de Kubernetes les formes de base à l’aide de kubeadm [ici](https://kubernetes.io/docs/setup/independent/install-kubeadm/). Il suffit d’activer de [section de planification de systèmes d’exploitation mixtes](#enable-mixed-os-scheduling) .

> [!NOTE]  
> Une machine Linux récemment mis à jour est nécessaire pour suivre l’exemple; Kubernetes maître des ressources comme [kube-dns](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), [Planificateur de kube](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)et [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) n'ont pas été adaptés à Windows encore. 

> [!tip]
> Les instructions de Linux sont spécifiquement vers **Ubuntu 16.04**. Autres versions de Linux certifiées pour s’exécuter Kubernetes offrent également des commandes équivalentes que vous pouvez remplacer. Ils seront également interagir avec succès avec Windows.


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
Pour être en mesure d’utiliser des conteneurs, vous avez besoin d’un moteur de conteneur, par exemple, Docker. Pour obtenir la version la plus récente, vous pouvez utiliser [ces instructions](https://docs.docker.com/install/linux/docker-ce/ubuntu/) pour l’installation de Docker. Vous pouvez vérifier que docker est correctement installé en exécutant un `hello-world` conteneur:

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>Installer kubeadm ###
Télécharger `kubeadm` fichiers binaires de votre distribution Linux et initialiser votre cluster.

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
Kubernetes sur Linux nécessite l’échange d’espace pour être désactivé:

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>Initialiser master ###
Notez vers le bas de votre sous-réseau de cluster (par exemple, 10.244.0.0/16) et le sous-réseau de service (par exemple, 10.96.0.0/12) et initialiser votre maître à l’aide de kubeadm:

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

Ce processus peut prendre plusieurs minutes. Une fois terminé, vous devriez voir un écran comme cette confirmation que votre maître a été initialisé:

![texte](media/kubeadm-init.png)

> [!tip]
> Prenez note de la sortie de commande de jointure kubeadm dans l’image ci-dessus *maintenant* avant qu’il ait été perdu.

> [!tip]
> Si vous disposez d’une version de Kubernetes souhaitée que vous souhaitez utiliser, vous pouvez passer le `--kubernetes-version` indicateur pour kubeadm.

Nous n’avons pas encore terminé. Pour utiliser `kubectl` en tant qu’utilisateur standard, exécutez le suivant __**dans un interpréteur de commandes utilisateur unelevated, non racine**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Maintenant, vous pouvez utiliser kubectl pour modifier ou afficher des informations sur votre cluster.

### <a name="enable-mixed-os-scheduling"></a>Activer la planification des systèmes d’exploitation mixtes ###
Par défaut, certaines ressources Kubernetes sont écrites de manière à ce qu’ils sont planifiés sur tous les nœuds. Toutefois, dans un environnement multi-OS nous ne voulons pas les ressources de Linux à interférer ou être planifiée double sur les nœuds de Windows et vice versa. Pour cette raison, nous devons appliquer des étiquettes de [NodeSelector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) . 

À cet égard, nous allons le linux des correctifs kube-proxy [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) à cible Linux.

Vérifiez que la stratégie de mise à jour de `kube-proxy` DaemonSet est défini sur [RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/):

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

Ensuite, le DaemonSet des correctifs en téléchargeant [ce nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) et l’appliquer pour cibler uniquement Linux:

```bash
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

Une fois que vous le succès, vous devez voir «Sélecteurs de nœuds» de `kube-proxy` et n’importe quel autre DaemonSets la valeur `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![texte](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>Collecter des informations sur le cluster ###
Pour joindre correctement futures nœuds à la forme de base, vous devez écrire vers le bas les informations suivantes:
  1. `kubeadm join` commande de sortie ([ici](#initialize-master))
    * Exemple: `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. Définis au cours de sous-réseau de cluster `kubeadm init` ([ici](#initialize-master))
    * Exemple: `10.244.0.0/16`
  3. Sous-réseau de service défini au cours de `kubeadm init` ([ici](#initialize-master))
    * Exemple: `10.96.0.0/12`
    * Vous pouvez également trouver à l’aide de `kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. Adresse IP de service Kube-dns 
    * Exemple: `10.96.0.10`
    * Vous trouverez dans le champ «Cluster IP» à l’aide `kubectl get svc/kube-dns -n kube-system`
  5. Kubernetes `config` fichier généré après `kubeadm init` ([ici](#initialize-master)). Si vous avez suivi les instructions, cela peut trouver dans les chemins d’accès suivants:
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>Vérification du master ##
Après quelques minutes, le système doit être dans l’état suivant:

  - Sous `kubectl get pods -n kube-system`, il y aura pods pour tous les [composants de maître Kubernetes](https://kubernetes.io/docs/concepts/overview/components/#master-components) dans `Running` état.
  - L’appel `kubectl cluster-info` affichera les informations concernant le serveur d’API master Kubernetes en plus des modules complémentaires DNS.

## <a name="next-steps"></a>Étapes suivantes ## 
Dans cette section, nous décrit comment configurer un nœud maître Kubernetes à l’aide de kubeadm. Vous êtes maintenant prêt à l’étape 3:

> [!div class="nextstepaction"]
> [Choix d’une solution de réseau](./network-topologies.md)
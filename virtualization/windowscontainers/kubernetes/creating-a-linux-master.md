---
title: Nœud maître (Master) Kubernetes à partir de zéro
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: how-to
description: Création d’un maître de cluster Kubernetes.
keywords: kubernetes, 1,14, Master, Linux
ms.openlocfilehash: 383163f29ab439ddd817640fca7203269810dd51
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985083"
---
# <a name="creating-a-kubernetes-master"></a>Création d’un maître Kubernetes #
> [!NOTE]
> Ce guide a été validé sur Kubernetes v 1.14. En raison de la volatilité de Kubernetes de la version à la version, cette section peut faire des hypothèses qui ne sont pas vraies pour toutes les futures versions. La documentation officielle sur l’initialisation des maîtres Kubernetes à l’aide de kubeadm est disponible [ici](https://kubernetes.io/docs/setup/independent/install-kubeadm/). Il vous suffit d’activer la [section de planification du système d’exploitation mixte](#enable-mixed-os-scheduling) .

> [!NOTE]
> Un ordinateur Linux récemment mis à jour est requis pour suivre la procédure. Les ressources maîtres Kubernetes comme [Kube-DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), [Kube-Scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)et [Kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) n’ont pas encore été portées vers Windows.

> [!tip]
> Les instructions Linux sont adaptées à **Ubuntu 16,04**. D’autres distributions Linux certifiées pour exécuter Kubernetes doivent également proposer des commandes équivalentes que vous pouvez remplacer. Ils interagissent également avec Windows.


## <a name="initialization-using-kubeadm"></a>Initialisation à l’aide de kubeadm ##
À moins d’être explicitement spécifié dans le cas contraire, exécutez les commandes ci-dessous en tant que **root**.

Tout d’abord, accédez à un interpréteur de commandes racine avec élévation de privilèges :

```bash
sudo –s
```

Assurez-vous que votre machine est à jour :

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>Installation de Docker ###
Pour pouvoir utiliser des conteneurs, vous avez besoin d’un moteur de conteneur, tel que l’ancrage. Pour obtenir la version la plus récente, vous pouvez utiliser [ces instructions](https://docs.docker.com/install/linux/docker-ce/ubuntu/) pour l’installation de l’amarrage. Vous pouvez vérifier que l’arrimeur est correctement installé en exécutant un `hello-world` conteneur :

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>Installer kubeadm ###
Téléchargez les `kubeadm` fichiers binaires pour votre distribution Linux et initialisez votre cluster.

> [!Important]
> En fonction de votre distribution Linux, vous devrez peut-être remplacer `kubernetes-xenial` par le [nom de nom](https://wiki.ubuntu.com/Releases)correct.

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl
```

### <a name="prepare-the-master-node"></a>Préparer le nœud principal ###
Kubernetes sur Linux nécessite que l’espace d’échange soit désactivé :

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

### <a name="initialize-master"></a>Initialiser le maître ###
Notez votre sous-réseau de cluster (par exemple, 10.244.0.0/16) et le sous-réseau de service (par exemple, 10.96.0.0/12) et initialisez votre maître à l’aide de kubeadm :

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

Cela peut prendre quelques minutes. Une fois l’opération terminée, vous devriez voir un écran de ce type confirmant que votre maître a été initialisé :

![text](media/kubeadm-init.png)

> [!tip]
> Vous devez prendre note de cette commande de jointure kubeadm. Faut le jeton kubeadm expire, vous pouvez utiliser `kubeadm token create --print-join-command` pour créer un nouveau jeton.

> [!tip]
> Si vous souhaitez utiliser une version Kubernetes souhaitée, vous pouvez passer l' `--kubernetes-version` indicateur à kubeadm.

Nous n’avons pas encore terminé. Pour utiliser `kubectl` comme utilisateur standard, exécutez la commande suivante __ **dans un interpréteur de commandes utilisateur non racine non élevé**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Vous pouvez maintenant utiliser kubectl pour modifier ou afficher des informations sur votre cluster.

### <a name="enable-mixed-os-scheduling"></a>Activer la planification du système d’exploitation mixte ###
Par défaut, certaines ressources Kubernetes sont écrites de manière à ce qu’elles soient planifiées sur tous les nœuds. Toutefois, dans un environnement à plusieurs systèmes d’exploitation, nous ne voulons pas que les ressources Linux interfèrent ou soient double-planifiées sur les nœuds Windows, et vice versa. C’est pourquoi nous devons appliquer des étiquettes [NodeSelector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) .

À cet égard, nous allons corriger le [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) Linux Kube-proxy pour cibler Linux uniquement.

Tout d’abord, nous allons créer un répertoire pour stocker les fichiers manifeste. YAML :
```bash
mkdir -p kube/yaml && cd kube/yaml
```

Vérifiez que la stratégie de mise à jour de `kube-proxy` DaemonSet est définie sur [RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/):

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

Ensuite, corrigez le DaemonSet en téléchargeant [ce nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) et appliquez-le uniquement à Linux cible :

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

Une fois l’opération réussie, vous devez voir « sélecteurs de nœuds » de `kube-proxy` et tout autre les daemonsets défini sur`beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![text](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>Collecter les informations de cluster ###
Pour joindre avec succès les futurs nœuds au maître, vous devez garder une trace des informations suivantes :
  1. `kubeadm join`commande à partir de la sortie ([ici](#initialize-master))
    * Exemple : `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. Sous-réseau de clusters défini pendant `kubeadm init` ([ici](#initialize-master))
    * Exemple : `10.244.0.0/16`
  3. Sous-réseau de service défini pendant `kubeadm init` ([ici](#initialize-master))
    * Exemple : `10.96.0.0/12`
    * Peut également être trouvé à l’aide de`kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. KUBE-IP du service DNS
    * Exemple : `10.96.0.10`
    * Se trouve dans le champ « IP de cluster » à l’aide de`kubectl get svc/kube-dns -n kube-system`
  5. `config`Fichier Kubernetes généré après `kubeadm init` ([ici](#initialize-master)). Si vous avez suivi les instructions, cela se trouve dans les chemins d’accès suivants :
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>Vérification du maître ##
Après quelques minutes, le système doit être dans l’état suivant :

  - Sous `kubectl get pods -n kube-system` , il y aura des gousses pour les [composants Kubernetes Master](https://kubernetes.io/docs/concepts/overview/components/#master-components) dans `Running` État.
  - `kubectl cluster-info`L’appel de affiche des informations sur le serveur de l’API maître Kubernetes en plus des modules complémentaires DNS.

> [!tip]
> Comme kubeadm n’installe pas la mise en réseau, les gousses DNS peuvent toujours être dans ou dans l' `ContainerCreating` `Pending` État. Ils basculent vers l' `Running` État après avoir [choisi une solution réseau](./network-topologies.md).

## <a name="next-steps"></a>Étapes suivantes ##
Dans cette section, nous avons abordé la configuration d’un maître Kubernetes à l’aide de kubeadm. Vous êtes maintenant prêt pour l’étape 3 :

> [!div class="nextstepaction"]
> [Choix d’une solution réseau](./network-topologies.md)
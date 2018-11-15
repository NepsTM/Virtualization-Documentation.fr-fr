---
title: Jonction de nœuds de Linux
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Jonction d’un nœud de Linux à un cluster Kubernetes avec v1.12.
keywords: kubernetes, 1.12, windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 97c12d70db9679dbb85877f0985c6053f95fa500
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2018
ms.locfileid: "6947978"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Jonction de nœuds Linux à un Cluster

Une fois que vous avez [d’installation d’un nœud maître Kubernetes](creating-a-linux-master.md) et [sélectionné votre solution de réseau auquel vous souhaitez](network-topologies.md), vous êtes prêt à joindre des nœuds Linux à votre cluster. Cela requiert une [préparation sur le nœud de Linux](joining-linux-workers.md#preparing-a-linux-node) avant de joindre.
> [!tip]
> Les instructions de Linux sont spécifiquement vers **Ubuntu 16.04**. Autres versions de Linux certifiées pour s’exécuter Kubernetes offrent également des commandes équivalentes que vous pouvez remplacer. Ils seront également interagir avec succès avec Windows.

## <a name="preparing-a-linux-node"></a>Préparation d’un nœud de Linux

> [!NOTE]
> Sauf spécification explicite dans le cas contraire, exécuter toutes les commandes dans un **interpréteur de commandes avec élévation de privilèges, utilisateur racine**.

Tout d’abord, obtenez dans un shell racine:

```bash
sudo –s
```

Assurez-vous que votre ordinateur est à jour:

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>Installer Docker

Pour être en mesure d’utiliser des conteneurs, vous avez besoin d’un moteur de conteneur, par exemple, Docker. Pour obtenir la version la plus récente, vous pouvez utiliser [ces instructions](https://docs.docker.com/install/linux/docker-ce/ubuntu/) pour l’installation de Docker. Vous pouvez vérifier que docker est correctement installé en exécutant `hello-world` image:

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>Installer kubeadm

Télécharger `kubeadm` fichiers binaires de votre distribution Linux et initialiser votre cluster.

> [!Important]  
> En fonction de votre distribution Linux, vous devrez peut-être remplacer `kubernetes-xenial` sous le [nom de code](https://wiki.ubuntu.com/Releases)de correct.

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>Désactiver l’échange

Kubernetes sur Linux nécessite l’échange d’espace pour être désactivé:

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(Flannel uniquement) Activer le trafic de IPv4 reliée à iptables

Si vous avez choisi Flannel en tant qu’il est recommandé d’activer votre solution d’accès réseau reliées par un pont de trafic IPv4 pour les chaînes iptables. Vous devez ont [déjà cette opération effectuée pour du master](network-topologies.md#flannel-in-host-gateway-mode) et devez maintenant répéter dans le nœud de Linux qui a l’intention de joindre. Elle peut être effectuée à l’aide de la commande suivante:

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>Copier le certificat de Kubernetes

**En tant que régulière, utilisateur (non racine)**, procédez comme 3 suit.

1. Créer des Kubernetes pour Linux répertoire:

```bash
mkdir -p $HOME/.kube
```

1. Copiez le fichier de certificat de Kubernetes (`$HOME/.kube/config`) [à partir du master](./creating-a-linux-master.md#collect-cluster-information) et enregistrer en tant que `$HOME/.kube/config` sur le travail.

1. Définir la propriété de fichier du fichier de configuration copiée comme suit:

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>Jonction de nœud

Enfin, pour rejoindre le cluster, exécutez la `kubeadm join` [nous notée précédemment](./creating-a-linux-master.md#initialize-master) **en tant que racine**de commande:

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

En cas de succès, vous devriez voir une sortie similaire à ceci:

![texte](./media/node-join.png)

## <a name="next-steps"></a>Étapes suivantes

Dans cette section, nous avons abordé comment joindre des travailleurs de Linux à notre cluster Kubernetes. Vous êtes maintenant prêt à l’étape 6:
> [!div class="nextstepaction"]
> [Déploiement des ressources de Kubernetes](./deploying-resources.md)
---
title: Jonction d’un cluster de nœuds Linux
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: how-to
description: Joindre un nœud Linux à un cluster Kubernetes avec v 1.14.
keywords: kubernetes, 1,14, Windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: ababeda847badc2058739c8cd2de36d7f46581d8
ms.sourcegitcommit: bb18e6568393da748a6d511d41c3acbe38c62668
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/12/2020
ms.locfileid: "88161888"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Jonction de nœuds Linux à un cluster

Une fois que vous avez [configuré un nœud principal Kubernetes](creating-a-linux-master.md) et que vous avez [sélectionné la solution réseau](network-topologies.md)de votre choix, vous êtes prêt à joindre les nœuds Linux à votre cluster. Cela nécessite une [préparation sur le nœud Linux avant d'](joining-linux-workers.md#preparing-a-linux-node) être joint.

> [!TIP]
> Les instructions Linux sont adaptées à **Ubuntu 16,04**. D’autres distributions Linux certifiées pour exécuter Kubernetes doivent également proposer des commandes équivalentes que vous pouvez remplacer. Ils interagissent également avec Windows.

## <a name="preparing-a-linux-node"></a>Préparation d’un nœud Linux

> [!NOTE]
> Sauf spécification explicite, exécutez les commandes dans un interpréteur de commandes d' **utilisateur racine élevé**.

Tout d’abord, accédez à un interpréteur de commandes racine :

```bash
sudo –s
```

Assurez-vous que votre machine est à jour :

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>Installation de Docker

Pour pouvoir utiliser des conteneurs, vous avez besoin d’un moteur de conteneur, tel que l’ancrage. Pour obtenir la version la plus récente, vous pouvez utiliser [ces instructions](https://docs.docker.com/install/linux/docker-ce/ubuntu/) pour l’installation de l’amarrage. Vous pouvez vérifier que l’Assistant d’ancrage est correctement installé en exécutant l' `hello-world` image :

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>Installer kubeadm

Téléchargez les `kubeadm` fichiers binaires pour votre distribution Linux et initialisez votre cluster.

> [!IMPORTANT]
> En fonction de votre distribution Linux, vous devrez peut-être remplacer `kubernetes-xenial` par le [nom de nom](https://wiki.ubuntu.com/Releases)correct.

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl
```

## <a name="disable-swap"></a>Désactiver l’échange

Kubernetes sur Linux nécessite que l’espace d’échange soit désactivé :

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(Flannel uniquement) Activer le trafic IPv4 de pont vers iptables

Si vous avez choisi Flannel comme solution de mise en réseau, il est recommandé d’activer le trafic IPv4 de pont vers les chaînes iptables. Vous avez [déjà effectué cette opération pour le maître](network-topologies.md#flannel-in-host-gateway-mode) et vous devez maintenant la répéter pour le nœud Linux qui a l’intention de rejoindre. Vous pouvez le faire à l’aide de la commande suivante :

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>Copier le certificat Kubernetes

**Comme utilisateur standard (non racine)**, effectuez les trois étapes suivantes.

1. Créer un répertoire Kubernetes pour Linux :

```bash
mkdir -p $HOME/.kube
```

2. Copiez le fichier de certificat Kubernetes ( `$HOME/.kube/config` ) [à partir de Master](./creating-a-linux-master.md#collect-cluster-information) et enregistrez-le `$HOME/.kube/config` sur le Worker.

> [!TIP]
> Vous pouvez utiliser des outils SCP tels que [WinSCP](https://winscp.net/eng/download.php) pour transférer le fichier de configuration entre les nœuds.

3. Définissez la propriété du fichier de configuration copié comme suit :

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>Jonction du nœud

Enfin, pour rejoindre le cluster, exécutez la `kubeadm join` commande [indiquée précédemment](./creating-a-linux-master.md#initialize-master) **comme racine**:

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

En cas de réussite, vous devriez obtenir une sortie similaire à ce qui suit :

![Une capture d’écran de la jointure de nœuds complète la sortie dans bash.](./media/node-join.png)

## <a name="next-steps"></a>Étapes suivantes

Dans cette section, nous avons abordé la manière de joindre des employés Linux à notre cluster Kubernetes. Vous êtes maintenant prêt pour l’étape 6 :
> [!div class="nextstepaction"]
> [Déploiement des ressources Kubernetes](./deploying-resources.md)
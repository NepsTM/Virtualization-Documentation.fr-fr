---
title: Topologies de réseau
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: how-to
ms.prod: containers
description: Topologies de réseau prises en charge sur Windows et Linux.
keywords: kubernetes, 1,14, Windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c322edb6a5ead34d7988f83d8cb8fba7c99cec0d
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192536"
---
# <a name="network-solutions"></a>Network Solutions #

Une fois que vous avez [configuré un nœud maître Kubernetes](./creating-a-linux-master.md) , vous êtes prêt à choisir une solution de mise en réseau. Il existe plusieurs façons de rendre le [sous-réseau de cluster](./getting-started-kubernetes-windows.md#cluster-subnet-def) virtuel pouvant être routé entre les nœuds. Choisissez l’une des options suivantes pour Kubernetes sur Windows aujourd’hui :

1. Utilisez un plug-in CNI tel que [Flannel](#flannel-in-vxlan-mode) pour configurer un réseau de superposition pour vous.
2. Utilisez un plug-in CNI tel que [Flannel](#flannel-in-host-gateway-mode) pour programmer des itinéraires pour vous (utilise le mode de mise en réseau l2bridge).
3. Configurez un [commutateur intelligent Top-of-rack (TDR)](#configuring-a-tor-switch) pour router le sous-réseau.

> [!tip]
> Il existe une quatrième solution de mise en réseau sur Windows qui s’appuie sur Open vSwitch (OvS) et Open Virtual Network (OVN). La documentation de ce document est hors de portée pour ce document, mais vous pouvez lire [ces instructions](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) pour le configurer.

## <a name="flannel-in-vxlan-mode"></a>Flannel en mode vxlan

Flannel en mode vxlan peut être utilisé pour configurer un réseau de recouvrement virtuel configurable qui utilise le tunneling VXLAN pour acheminer les paquets entre les nœuds.

### <a name="prepare-kubernetes-master-for-flannel"></a>Préparer Kubernetes maître pour Flannel
Une préparation mineure est recommandée sur le [maître Kubernetes](./creating-a-linux-master.md) dans notre cluster. Il est recommandé d’activer le trafic IPv4 de pont vers les chaînes iptables lors de l’utilisation de Flannel. Pour ce faire, vous pouvez utiliser la commande suivante :

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Télécharger & configurer Flannel ###
Téléchargez le manifeste Flannel le plus récent :

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Vous devez modifier deux sections pour activer le backend de mise en réseau vxlan :

1. Dans la `net-conf.json` section de votre `kube-flannel.yml` , double-Vérifiez :
 * Le sous-réseau de cluster (par exemple, « 10.244.0.0/16 ») est défini comme souhaité.
 * VNI 4096 est défini dans le serveur principal
 * Le port 4789 est défini dans le serveur principal
2. Dans la `cni-conf.json` section de votre `kube-flannel.yml` , remplacez le nom du réseau par `"vxlan0"` .

Après avoir appliqué les étapes ci-dessus, votre `net-conf.json` doit se présenter comme suit :
```json
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan",
        "VNI" : 4096,
        "Port": 4789
      }
    }
```

> [!NOTE]
> VNI doit avoir la valeur 4096 et le port 4789 pour Flannel sur Linux pour interagir avec Flannel sur Windows. La prise en charge d’autres VNIs sera bientôt disponible. Pour obtenir une explication de ces champs, consultez [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) .

Votre `cni-conf.json` doit se présenter comme suit :
```json
cni-conf.json: |
    {
      "name": "vxlan0",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
```
> [!tip]
> Pour plus d’informations sur les options ci-dessus, consultez les documents de plug-in CNI [Flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference), [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)et [Bridge](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) officiels pour Linux.

### <a name="launch-flannel--validate"></a>Lancer Flannel & valider ###
Lancez Flannel à l’aide de :

```bash
kubectl apply -f kube-flannel.yml
```

Ensuite, étant donné que les Pod Flannel sont basés sur Linux, appliquez le correctif Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) à `kube-flannel-ds` DaemonSet pour cibler uniquement Linux (nous lancerons le processus de l’agent hôte « flanneld » Flannel sur Windows ultérieurement lors de la jointure) :

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]
> Si des nœuds ne sont pas basés sur x86-64, remplacez `-amd64` -les par votre architecture de processeur.

Après quelques minutes, vous devriez voir tous les Pod comme étant en cours d’exécution si le réseau Pod Flannel a été déployé.

```bash
kubectl get pods --all-namespaces
```

![texte](media/kube-master.png)

Le NodeSelector doit également être appliqué au DaemonSet Flannel `beta.kubernetes.io/os=linux` .

```bash
kubectl get ds -n kube-system
```

![texte](media/kube-daemonset.png)

> [!tip]
> Pour les autres Flannel-DS-* les daemonsets, ceux-ci peuvent être ignorés ou supprimés, car ils ne sont pas planifiés si aucun nœud ne correspond à cette architecture de processeur.

> [!tip]
> Confondre? Voici un exemple complet de [Kube-Flannel. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) pour Flannel v 0.11.0 avec ces étapes préappliquées pour le sous-réseau de cluster par défaut `10.244.0.0/16` .

Une fois l’opération réussie, passez aux [étapes suivantes](#next-steps).

## <a name="flannel-in-host-gateway-mode"></a>Flannel en mode de passerelle hôte

À côté de [Flannel vxlan](#flannel-in-vxlan-mode), une autre option pour la mise en réseau Flannel est le *mode de passerelle d’hôte* (Host-GW), qui implique la programmation d’itinéraires statiques sur chaque nœud pour les sous-réseaux Pod du nœud à l’aide de l’adresse d’hôte du nœud cible comme tronçon suivant.

### <a name="prepare-kubernetes-master-for-flannel"></a>Préparer Kubernetes maître pour Flannel

Une préparation mineure est recommandée sur le [maître Kubernetes](./creating-a-linux-master.md) dans notre cluster. Il est recommandé d’activer le trafic IPv4 de pont vers les chaînes iptables lors de l’utilisation de Flannel. Pour ce faire, vous pouvez utiliser la commande suivante :

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Télécharger & configurer Flannel ###
Téléchargez le manifeste Flannel le plus récent :

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Il existe un fichier que vous devez modifier pour activer la mise en réseau de l’hôte-GW sur Windows/Linux.

Dans la `net-conf.json` section de votre Kube-Flannel. yml, vérifiez les éléments suivants :
1. Le type de serveur principal réseau utilisé est défini sur `host-gw` au lieu de `vxlan` .
2. Le sous-réseau de cluster (par exemple, « 10.244.0.0/16 ») est défini comme souhaité.

Après avoir appliqué les 2 étapes, votre `net-conf.json` doit se présenter comme suit :
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Lancer Flannel & valider ###
Lancez Flannel à l’aide de :

```bash
kubectl apply -f kube-flannel.yml
```

Ensuite, étant donné que les Pod Flannel sont basés sur Linux, appliquez notre correctif Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) à `kube-flannel-ds` DaemonSet pour cibler uniquement Linux (nous lancerons le processus de l’agent hôte « flanneld » Flannel sur Windows ultérieurement lors de la jointure) :

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]
> Si des nœuds ne sont pas basés sur x86-64, remplacez `-amd64` -les par l’architecture de processeur souhaitée.

Après quelques minutes, vous devriez voir tous les Pod comme étant en cours d’exécution si le réseau Pod Flannel a été déployé.

```bash
kubectl get pods --all-namespaces
```

![texte](media/kube-master.png)

Le NodeSelector doit également être appliqué au DaemonSet Flannel.

```bash
kubectl get ds -n kube-system
```

![texte](media/kube-daemonset.png)

> [!tip]
> Pour les autres Flannel-DS-* les daemonsets, ceux-ci peuvent être ignorés ou supprimés, car ils ne sont pas planifiés si aucun nœud ne correspond à cette architecture de processeur.

> [!tip]
> Confondre? Voici un exemple complet de [Kube-Flannel. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) pour Flannel v 0.11.0 avec ces deux étapes préappliquées pour le sous-réseau de cluster par défaut `10.244.0.0/16` .

Une fois l’opération réussie, passez aux [étapes suivantes](#next-steps).

## <a name="configuring-a-tor-switch"></a>Configuration d’un commutateur TDR ##
> [!NOTE]
> Vous pouvez ignorer cette section si vous avez choisi [Flannel comme solution de mise en réseau](#flannel-in-host-gateway-mode).
La configuration du commutateur TDR se produit en dehors de vos nœuds réels. Pour plus d’informations, consultez la [documentation officielle sur Kubernetes](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Étapes suivantes ##
Dans cette section, nous avons abordé la sélection et la configuration d’une solution de mise en réseau. Vous êtes maintenant prêt pour l’étape 4 :

> [!div class="nextstepaction"]
> [Rejoindre les Workers Windows](./joining-windows-workers.md)
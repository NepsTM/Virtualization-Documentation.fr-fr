---
title: Topologies réseau
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Topologies réseau prises en charge sur Windows et Linux.
keywords: kubernetes, 1,14, Windows, mise en route
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6b0e13258b749ad3dfd5c8349200ca8a54908952
ms.sourcegitcommit: 42cb47ba4f3e22163869d094bd0c9cff415a43b0
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/02/2019
ms.locfileid: "9884980"
---
# <a name="network-solutions"></a>Network Solutions #

Dès lors que vous avez [configuré un nœud maître Kubernetes](./creating-a-linux-master.md) , vous êtes prêt à choisir une solution réseau. Il existe plusieurs façons de rendre le [sous-réseau de cluster](./getting-started-kubernetes-windows.md#cluster-subnet-def) virtuel routable entre les nœuds. Sélectionnez l’une des options suivantes pour Kubernetes sur Windows aujourd’hui:

1. Utilisez un plug-in CNI tel que [Flannel](#flannel-in-vxlan-mode) pour configurer un réseau de superposition pour vous.
2. Utilisez un plug-in CNI tel que [Flannel](#flannel-in-host-gateway-mode) pour programmer des itinéraires pour vous (utilise le mode réseau d’l2bridge).
3. Configurez un [commutateur intelligent de haut du rack (TDR)](#configuring-a-tor-switch) pour acheminer le sous-réseau.

> [!tip]  
> Il existe une solution réseau de quatrième sur Windows qui utilise l’Open vSwitch (OvS) et le réseau virtuel (OVN). Pour ce faire, vous pouvez le faire hors de l’objectif de ce document, mais vous pouvez lire [ces instructions](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) pour le configurer.

## <a name="flannel-in-vxlan-mode"></a>Flannel en mode vxlan

Flannel en mode vxlan peut être utilisé pour configurer un réseau de superposition virtuel configurable qui utilise le tunneling VXLAN pour acheminer les paquets entre les nœuds.

### <a name="prepare-kubernetes-master-for-flannel"></a>Préparer le maître Kubernetes pour Flannel
Une préparation mineure est recommandée sur le [maître Kubernetes](./creating-a-linux-master.md) dans notre cluster. Il est recommandé d’activer le trafic IPv4 avec passerelle vers des chaînes iptables lors de l’utilisation de Flannel. Pour cela, vous pouvez utiliser la commande suivante:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Télécharger & configurer Flannel ###
Téléchargez la version la plus récente du manifeste Flannel:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Il existe deux sections à modifier pour activer le serveur principal du réseau vxlan:

1. Dans la `net-conf.json` section de votre `kube-flannel.yml`, vérifiez les éléments suivants:
 * Le sous-réseau de cluster (par exemple, «10.244.0.0/16») est défini comme vous le souhaitez.
 * VNI 4096 est défini dans le système principal
 * Le port 4789 est défini dans le système principal
2. Dans la `cni-conf.json` section de votre `kube-flannel.yml`, changez le nom du réseau `"vxlan0"`en.

Après avoir appliqué les étapes ci- `net-conf.json` dessus, vous devez vous répartir comme suit:
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
> VNI doit être défini sur 4096 et port 4789 pour Flannel sur Linux pour interagir avec Flannel sur Windows. La prise en charge des autres VNIs est disponible prochainement. Pour obtenir une explication de ces champs, voir [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) .

Vous `cni-conf.json` devez voir ce qui suit:
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
> Pour plus d’informations sur les options ci-dessus, consultez les documents de plugin officiels CNI [Flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference), [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)et [Bridge](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) pour Linux.

### <a name="launch-flannel--validate"></a>Lancer Flannel & valider ###
Lancez Flannel à l’aide de:

```bash
kubectl apply -f kube-flannel.yml
```

Ensuite, dans la mesure où les gousses Flannel sont basées sur Linux [](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) , appliquez le `kube-flannel-ds` correctif Linux NodeSelector à DaemonSet pour cibler uniquement Linux (nous lançons le processus d’agent d’hébergement «Flanneld» Flannel sur Windows ultérieurement lors de la participation):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Si des nœuds ne sont pas basés sur x86 `-amd64` -64, remplacez-le par votre architecture de processeur.

Après quelques minutes, vous devriez voir toutes les Pod comme exécutées si le réseau Flannel Pod a été déployé.

```bash
kubectl get pods --all-namespaces
```

![texte](media/kube-master.png)

Le NodeSelector `beta.kubernetes.io/os=linux` Flannel DaemonSet doit également être appliqué.

```bash
kubectl get ds -n kube-system
```

![texte](media/kube-daemonset.png)

> [!tip]  
> Pour le Flannel-DS-* restants, vous pouvez ignorer ou supprimer les éléments dans la mesure où ils ne seront pas prévus s’il n’y a aucun nœud correspondant à cette architecture de processeur.

> [!tip]  
> Confondus? Voici un exemple complet [Kube-Flannel. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) pour Flannel v 0.11.0 avec ces étapes préappliquées pour le sous-réseau `10.244.0.0/16`de clusters par défaut.

Lorsque vous avez terminé, passez aux [étapes suivantes](#next-steps).

## <a name="flannel-in-host-gateway-mode"></a>Flannel en mode de passerelle hôte

Parallèlement à [Flannel vxlan](#flannel-in-vxlan-mode), une autre option de mise en réseau Flannel est le *mode hôte-passerelle* (Host-GW), qui implique la programmation d’itinéraires statiques sur chaque nœud pour les sous-réseaux Pod du nœud cible à l’aide de l’adresse hôte du nœud cible comme tronçon suivant.

### <a name="prepare-kubernetes-master-for-flannel"></a>Préparer le maître Kubernetes pour Flannel

Une préparation mineure est recommandée sur le [maître Kubernetes](./creating-a-linux-master.md) dans notre cluster. Il est recommandé d’activer le trafic IPv4 avec passerelle vers des chaînes iptables lors de l’utilisation de Flannel. Pour cela, vous pouvez utiliser la commande suivante:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Télécharger & configurer Flannel ###
Téléchargez la version la plus récente du manifeste Flannel:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Il y a un fichier que vous devez modifier pour activer la mise en réseau hôte-GW sur Windows/Linux.

Dans la `net-conf.json` section de votre Kube-Flannel. yml, vérifiez les éléments suivants:
1. Le type du serveur principal du réseau utilisé est défini `host-gw` au lieu `vxlan`de.
2. Le sous-réseau de cluster (par exemple, «10.244.0.0/16») est défini comme vous le souhaitez.

Après avoir appliqué les deux étapes, `net-conf.json` vous devez vous répartir comme suit:
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
Lancez Flannel à l’aide de:

```bash
kubectl apply -f kube-flannel.yml
```

Ensuite, dans la mesure où les gousses Flannel sont basées sur Linux [](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) , appliquez notre `kube-flannel-ds` correctif Linux NodeSelector à DaemonSet pour cibler uniquement Linux (nous lançons le processus d’agent d’hébergement «Flanneld» Flannel sur Windows ultérieurement lors de la participation):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Si des nœuds ne sont pas basés sur x86 `-amd64` -64, remplacez-le par l’architecture de processeur souhaitée.

Après quelques minutes, vous devriez voir toutes les Pod comme exécutées si le réseau Flannel Pod a été déployé.

```bash
kubectl get pods --all-namespaces
```

![texte](media/kube-master.png)

Le NodeSelector Flannel DaemonSet doit également être appliqué.

```bash
kubectl get ds -n kube-system
```

![texte](media/kube-daemonset.png)

> [!tip]  
> Pour le Flannel-DS-* restants, vous pouvez ignorer ou supprimer les éléments dans la mesure où ils ne seront pas prévus s’il n’y a aucun nœud correspondant à cette architecture de processeur.

> [!tip]  
> Confondus? Voici un exemple complet [Kube-Flannel. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) pour Flannel v 0.11.0 en appliquant les deux étapes prédéfinies pour le sous `10.244.0.0/16`-réseau de clusters par défaut.

Lorsque vous avez terminé, passez aux [étapes suivantes](#next-steps).

## <a name="configuring-a-tor-switch"></a>Configuration d’un commutateur TDR ##
> [!NOTE]
> Vous pouvez ignorer cette section si vous avez choisi [Flannel comme solution réseau](#flannel-in-host-gateway-mode).
La configuration du commutateur TDR est en dehors de vos nœuds réels. Pour plus d’informations, consultez la rubrique [documents Kubernetes officiels](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Étapes suivantes ## 
Dans cette section, nous avons expliqué comment sélectionner et configurer une solution réseau. Vous êtes maintenant prêt à passer à l’étape 4:

> [!div class="nextstepaction"]
> [Rejoindre des travailleurs Windows](./joining-windows-workers.md)
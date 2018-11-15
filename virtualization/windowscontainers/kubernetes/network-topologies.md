---
title: Topologies de réseau
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Topologies de réseau prises en charge sur Windows et Linux.
keywords: kubernetes, 1.12, windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: bcbd7b530b58b663305ea5d8b84a75eaf971f997
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2018
ms.locfileid: "6948058"
---
# <a name="network-solutions"></a>Solutions de réseau #

Une fois que vous avez [d’installation d’un nœud maître Kubernetes](./creating-a-linux-master.md) vous êtes prêt à choisir une solution de mise en réseau. Il existe plusieurs façons de rendre le [sous-réseau de cluster](./getting-started-kubernetes-windows.md#cluster-subnet-def) de virtuel routable entre les nœuds. Choisir l’une des options suivantes pour Kubernetes sur Windows aujourd'hui:

1. Utilisez un CNI plug-in tiers tels que [Flannel](network-topologies.md#flannel-in-host-gateway-mode) les itinéraires le programme d’installation pour vous.
1. Configurer un smart [commutateur top-de-rack (ToR)](network-topologies.md#configuring-a-tor-switch) pour router le sous-réseau.

> [!tip]  
> Il existe une troisième mise en réseau de solution sur Windows qui tire parti de commutateur virtuel ouvert (OvS) et de réseau virtuel ouvert (OVN). Cette documentation est hors de portée de ce document, mais vous pouvez lire [ces instructions](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) pour configurer celui-ci.

## <a name="flannel-in-host-gateway-mode"></a>Flannel en mode hôte-passerelle

Une des options disponibles pour la mise en réseau Flannel est *en mode hôte-passerelle* (host-EG), qui implique la configuration d’itinéraires statiques entre les sous-réseaux sur tous les nœuds.
> [!NOTE]  
> Cela diffère au mode de mise en réseau de *superposition* dans Flannel, qui utilise l’encapsulation VXLAN et en cours de développement maintenant. Regardez cet espace …

### <a name="prepare-kubernetes-master-for-flannel"></a>Préparer le nœud maître Kubernetes Flannel

Certaines tâches de préparation mineure est recommandé sur le [nœud maître Kubernetes](./creating-a-linux-master.md) dans notre cluster. Il est recommandé d’activer reliée le trafic IPv4 pour les chaînes iptables lors de l’utilisation de Flannel. Cela peut être effectuée à l’aide de la commande suivante:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Télécharger et configurer Flannel ###
Téléchargez le manifeste Flannel la plus récent:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Il existe deux choses à faire pour activer la mise en réseau entre les deux Windows/Linux hôte-EG.

Dans le `net-conf.json` section de votre kube-flannel.yml, vérifiez que:
1. Le type de système de réseau principal utilisé est défini sur `host-gw` à la place de `vxlan`.
2. Le sous-réseau de cluster (par exemple, «10.244.0.0/16») est défini selon les besoins.

Après avoir appliqué les 2 étapes, votre `net-conf.json` doit se présenter comme suit:
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Lancer Flannel & Valider ###
Lancer à l’aide de Flannel:

```bash
kubectl apply -f kube-flannel.yml
```

Ensuite, dans la mesure où les pods Flannel sont basés sur Linux, correctif notre Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) à `kube-flannel-ds` DaemonSet pour cibler uniquement Linux (nous lançons Flannel «flanneld» hôte-processus de l’agent sur Windows ultérieurement lorsque vous rejoignez):

```
kubectl patch ds/kube-flannel-ds --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

Après quelques minutes, vous devez voir tous les pods comme en cours d’exécution si le réseau de pod Flannel a été déployé.

```bash
kubectl get pods --all-namespaces
```

![texte](media/kube-master.png)

Le DaemonSet Flannel doit également avoir le NodeSelector appliquée.

```bash
kubectl get ds -n kube-system
```

![texte](media/kube-daemonset.png)
> [!tip]  
> Vous êtes perdu? Voici un v0.9.1 de pour Flannel complète [exemple kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) ces étapes 2 appliqué au préalable pour le sous-réseau de cluster par défaut `10.244.0.0/16`.

## <a name="configuring-a-tor-switch"></a>Pour configurer un commutateur ToR ##
> [!NOTE]
> Vous pouvez ignorer cette section si vous avez choisi [Flannel comme solution de mise en réseau](#flannel-in-host-gateway-mode).
Configuration du commutateur ToR se produit en dehors de vos nœuds réelles. Pour plus d’informations, voir [les documents de Kubernetes officiels](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Étapes suivantes ## 
Dans cette section, nous avons abordé comment choisir une solution de mise en réseau. Vous êtes maintenant prêt à l’étape 4:

> [!div class="nextstepaction"]
> [Jonction de nœuds de travail Windows](./joining-windows-workers.md)
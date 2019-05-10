---
title: Topologies de réseau
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Topologies de réseau prises en charge sur Windows et Linux.
keywords: kubernetes, 1.14, windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6a2b7021efa0d90b69a88e1b498cddeadb3af80e
ms.sourcegitcommit: aaf115a9de929319cc893c29ba39654a96cf07e1
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/10/2019
ms.locfileid: "9622914"
---
# <a name="network-solutions"></a>Solutions de réseau #

Une fois que vous avez [d’installation d’un nœud maître Kubernetes](./creating-a-linux-master.md) , vous êtes prêt à choisir une solution de mise en réseau. Il existe plusieurs façons de rendre le [sous-réseau de cluster](./getting-started-kubernetes-windows.md#cluster-subnet-def) virtuel routable entre les nœuds. Choisir l’une des options suivantes pour Kubernetes sur Windows aujourd'hui:

1. Utilisez un plug-in CNI comme [Flannel](#flannel-in-vxlan-mode) permet de configurer un réseau de superposition pour vous.
2. Utilisez un plug-in CNI tels que [Flannel](#flannel-in-host-gateway-mode) les itinéraires programme pour vous.
3. Configurer un actives de [commutateur top-de-rack (ToR)](#configuring-a-tor-switch) pour router le sous-réseau.

> [!tip]  
> Il existe un quatrième mise en réseau de solution sur Windows qui tire parti de commutateur virtuel ouvert (OvS) et de réseau virtuel ouvrir (OVN). Cette documentation est hors de portée de ce document, mais vous pouvez lire [ces instructions](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) pour configurer celui-ci.

## <a name="flannel-in-vxlan-mode"></a>Flannel en mode vxlan

Flannel en mode vxlan peut servir à la configuration d’un réseau de superposition virtuelle configurable qui utilise VXLAN tunneling pour acheminer les paquets entre les nœuds.

### <a name="prepare-kubernetes-master-for-flannel"></a>Préparer le nœud maître Kubernetes Flannel
Certaines tâches de préparation mineure est recommandé sur le [nœud maître Kubernetes](./creating-a-linux-master.md) dans notre cluster. Il est recommandé d’activer le trafic IPv4 reliée aux chaînes iptables lorsque vous utilisez Flannel. Cela peut être effectuée à l’aide de la commande suivante:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Télécharger & configurer Flannel ###
Téléchargez le manifeste Flannel la plus récent:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Il existe deux sections, vous devez modifier pour activer le système principal de mise en réseau vxlan:

1. Dans le `net-conf.json` section de votre `kube-flannel.yml`, vérifiez:
 * Le sous-réseau de cluster (par exemple, «10.244.0.0/16») est défini comme souhaité.
 * 4096 VNI est définie dans le système principal
 * Port 4789 est défini dans le système principal
2. Dans le `cni-conf.json` section de votre `kube-flannel.yml`, modifiez le nom du réseau `"vxlan0"`.

Après avoir appliqué les étapes ci-dessus, votre `net-conf.json` doit se présenter comme suit:
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
> La VNI doit être définie à 4096 et 4789 pour Flannel sur Linux pour interagir avec Flannel sur Windows. Prise en charge pour les autres VNIs sera bientôt disponible. Pour obtenir une explication de ces champs, reportez-vous à la section [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) .

Votre `cni-conf.json` doit se présenter comme suit:
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
> Pour plus d’informations sur les options ci-dessus, veuillez consulter officiel CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference), [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)et le [pont](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) du plug-in documents pour Linux.

### <a name="launch-flannel--validate"></a>Lancer Flannel & valider ###
Lancer à l’aide de Flannel:

```bash
kubectl apply -f kube-flannel.yml
```

Ensuite, dans la mesure où les pods Flannel sont basés sur Linux, appliquer le correctif Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) `kube-flannel-ds` DaemonSet pour cibler uniquement Linux (nous lançons Flannel «flanneld» hôte-processus de l’agent sur Windows ultérieurement lorsque vous joignez):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Si tous les nœuds ne sont pas x86-64 basé, remplacez `-amd64` ci-dessus avec votre architecture de processeur.

Après quelques minutes, vous devez voir tous les pods comme en cours d’exécution si le réseau de pod Flannel a été déployé.

```bash
kubectl get pods --all-namespaces
```

![texte](media/kube-master.png)

Le DaemonSet Flannel doit également avoir le NodeSelector `beta.kubernetes.io/os=linux` appliquée.

```bash
kubectl get ds -n kube-system
```

![texte](media/kube-daemonset.png)

> [!tip]  
> Pour l’autres flannel - ds-* DaemonSets, ces peuvent être ignorées ou supprimées car ils ne seront pas planifiés si aucun nœud correspondant à cette architecture de processeur.

> [!tip]  
> Vous êtes perdu? Voici un [exemple kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) de complète pour Flannel v0.11.0 ces étapes appliquées au préalable pour le sous-réseau de cluster par défaut `10.244.0.0/16`.

Une fois que vous le succès, continuer aux [étapes suivantes](#next-steps).

## <a name="flannel-in-host-gateway-mode"></a>Flannel en mode hôte-passerelle

En même temps que [Flannel vxlan](#flannel-in-vxlan-mode), une autre option de mise en réseau Flannel est *en mode hôte-passerelle* (host-EG), qui implique la programmation d’itinéraires statiques sur chaque nœud à des sous-réseaux l’autre nœud à l’aide d’adresse de l’hôte du nœud cible comme un saut suivant.

### <a name="prepare-kubernetes-master-for-flannel"></a>Préparer le nœud maître Kubernetes Flannel

Certaines tâches de préparation mineure est recommandé sur le [nœud maître Kubernetes](./creating-a-linux-master.md) dans notre cluster. Il est recommandé d’activer le trafic IPv4 reliée aux chaînes iptables lorsque vous utilisez Flannel. Cela peut être effectuée à l’aide de la commande suivante:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Télécharger & configurer Flannel ###
Téléchargez le manifeste Flannel la plus récent:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Il existe un seul fichier, que vous devez modifier afin d’activer la mise en réseau entre les deux Windows/Linux hôte-EG.

Dans le `net-conf.json` section de votre kube-flannel.yml, vérifiez que:
1. Le type de réseau principal utilisé est défini sur `host-gw` à la place de `vxlan`.
2. Le sous-réseau de cluster (par exemple, «10.244.0.0/16») est défini comme souhaité.

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

### <a name="launch-flannel--validate"></a>Lancer Flannel & valider ###
Lancer à l’aide de Flannel:

```bash
kubectl apply -f kube-flannel.yml
```

Ensuite, dans la mesure où les pods Flannel sont basés sur Linux, correctif notre Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) à `kube-flannel-ds` DaemonSet pour cibler uniquement Linux (nous lançons Flannel «flanneld» hôte-processus de l’agent sur Windows ultérieurement lorsque vous joignez):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Si tous les nœuds ne sont pas x86-64 basé, remplacez `-amd64` ci-dessus avec l’architecture du processeur de votre choix.

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
> Pour l’autres flannel - ds-* DaemonSets, ces peuvent être ignorées ou supprimées car ils ne seront pas planifiés si aucun nœud correspondant à cette architecture de processeur.

> [!tip]  
> Vous êtes perdu? Voici un v0.11.0 de pour Flannel complète [exemple kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) ces étapes 2 appliquée au préalable pour le sous-réseau de cluster par défaut `10.244.0.0/16`.

Une fois que vous le succès, continuer aux [étapes suivantes](#next-steps).

## <a name="configuring-a-tor-switch"></a>Pour configurer un commutateur ToR ##
> [!NOTE]
> Vous pouvez ignorer cette section si vous avez choisi [Flannel comme solution de mise en réseau](#flannel-in-host-gateway-mode).
Configuration du commutateur ToR se produit en dehors de vos nœuds réelles. Pour plus d’informations, voir [officiel Kubernetes documents](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Étapes suivantes ## 
Dans cette section, nous avons abordé comment sélectionner et configurer une solution de mise en réseau. Vous êtes maintenant prêt à l’étape 4:

> [!div class="nextstepaction"]
> [Jonction de nœuds de travail Windows](./joining-windows-workers.md)
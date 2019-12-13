---
title: Exécution de Kubernetes en tant que service Windows
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: get-started-article
ms.prod: containers
description: Comment exécuter des composants Kubernetes en tant que services Windows.
keywords: kubernetes, 1,14, Windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: cd5026a244b57b5c70d4abfe076839130315a4f5
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909799"
---
# <a name="kubernetes-components-as-windows-services"></a>Composants Kubernetes en tant que services Windows 

Certains utilisateurs peuvent souhaiter configurer des processus tels que flanneld. exe, kubelet. exe, KUBE-proxy. exe ou d’autres pour qu’ils s’exécutent en tant que services Windows. Cela offre des avantages supplémentaires en matière de tolérance aux pannes, tels que les processus qui redémarrent automatiquement en cas de panne d’un processus ou d’un nœud inattendu.


## <a name="prerequisites"></a>Conditions préalables
1. Vous avez téléchargé [NSSM. exe](https://nssm.cc/download) dans le répertoire `c:\k`
2. Vous avez rejoint le nœud sur votre cluster et exécutez le script [install. ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1) ou [Start. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) sur votre nœud précédemment

## <a name="registering-windows-services"></a>Inscription des services Windows
Vous pouvez exécuter [un exemple de script](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1) qui utilise NSSM. exe qui inscrit `kubelet`, `kube-proxy`et `flanneld.exe` pour s’exécuter en tant que services Windows en arrière-plan :

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# <a name="managementiptabmanagementip"></a>[ManagementIP](#tab/ManagementIP)
Adresse IP affectée au nœud Windows. Vous pouvez utiliser `ipconfig` pour le trouver.

|  |  | 
|---------|---------|
|Paramètre     | `-ManagementIP`        |
|Valeur par défaut    | n.A.        |


# <a name="networkmodetabnetworkmode"></a>[NetworkMode](#tab/NetworkMode)
Le mode réseau `l2bridge` (hôte Flannel-GW) ou `overlay` (Flannel vxlan) choisi comme [solution réseau](./network-topologies.md).

> [!Important] 
> le mode de mise en réseau `overlay` (Flannel vxlan) requiert des binaires Kubernetes v 1.14 ou version ultérieure.

|  |  | 
|---------|---------|
|Paramètre     | `-NetworkMode`        |
|Valeur par défaut    | `l2bridge`        |


# <a name="clustercidrtabclustercidr"></a>[ClusterCIDR](#tab/ClusterCIDR)
[Plage du sous-réseau du cluster](./getting-started-kubernetes-windows.md#cluster-subnet-def).

|  |  | 
|---------|---------|
|Paramètre     | `-ClusterCIDR`        |
|Valeur par défaut    | `10.244.0.0/16`        |


# <a name="kubednsserviceiptabkubednsserviceip"></a>[KubeDnsServiceIP](#tab/KubeDnsServiceIP)
L' [adresse IP du service DNS Kubernetes](./getting-started-kubernetes-windows.md#kube-dns-def).

|  |  | 
|---------|---------|
|Paramètre     | `-KubeDnsServiceIP`        |
|Valeur par défaut    | `10.96.0.10`        |


# <a name="logdirtablogdir"></a>[LogDir](#tab/LogDir)
Répertoire dans lequel les journaux kubelet et Kube-proxy sont redirigés dans leurs fichiers de sortie respectifs.

|  |  | 
|---------|---------|
|Paramètre     | `-LogDir`        |
|Valeur par défaut    | `C:\k`        |

---


> [!TIP] 
> En cas de problème, consultez la [section résolution des problèmes](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services)

## <a name="manual-approach"></a>Approche manuelle
Si le [script référencé ci-dessus](#registering-windows-services) ne fonctionne pas pour vous, cette section fournit des *exemples de commandes* qui peuvent être utilisés pour inscrire ces services manuellement pas à pas.

> [!TIP] 
> Consultez [Kubelet et Kube-proxy peuvent maintenant s’exécuter en tant que services Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services) pour plus d’informations sur la façon de configurer des `kubelet` et des `kube-proxy` pour qu’ils s’exécutent en tant que services Windows natifs via `sc`.

### <a name="register-flanneldexe"></a>Inscrire flanneld. exe
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>Inscrire kubelet. exe
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>Inscrire Kube-proxy. exe (l2bridge/Host-GW)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>Inscrire Kube-proxy. exe (Overlay/vxlan)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```
---
title: Kubernetes en cours d’exécution en tant qu’un Service Windows
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: get-started-article
ms.prod: containers
description: Procédure pour exécuter des composants de Kubernetes en tant que services Windows.
keywords: kubernetes, 1.13, windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: 6c68edda6e2017640b0a490c3c30f063c81698b3
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578640"
---
# <a name="kubernetes-components-as-windows-services"></a>Composants de Kubernetes en tant que Services Windows 

Certains utilisateurs peuvent souhaiter configurer les processus tels que flanneld.exe, kubelet.exe, kube-proxy.exe ou d’autres à s’exécuter en tant que services Windows. Cela offre des avantages de tolérance de pannes supplémentaires telles que les processus de redémarrage automatique lors de blocage d’un processus ou nœud inattendu.


## <a name="prerequisites"></a>Prérequis
1. Vous avez téléchargé [nssm.exe](https://nssm.cc/download) dans les `c:\k` directory
2. Vous avez joint le nœud à votre cluster et exécutez le script [install.ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1) ou [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) sur votre nœud précédemment

## <a name="registering-windows-services"></a>Inscription des services Windows
Vous pouvez exécuter [un exemple de script](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1) qui utilise nssm.exe qui inscrivent `kubelet`, `kube-proxy`, et `flanneld.exe` à s’exécuter en tant que services Windows en arrière-plan:

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
L’adresse IP attribuée au nœud Windows. Vous pouvez utiliser `ipconfig` de trouver cet élément.

|  |  | 
|---------|---------|
|Paramètre     | `-ManagementIP`        |
|Valeur par défaut    | chargée        |


# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
Le mode réseau `l2bridge` (flannel hôte-EG) ou `overlay` (flannel vxlan) choisi en tant que [solution de réseau](./network-topologies.md).

> [!Important] 
> `overlay` mode de mise en réseau (flannel vxlan) nécessite des fichiers binaires de Kubernetes v1.14 ou une version ultérieure.

|  |  | 
|---------|---------|
|Paramètre     | `-NetworkMode`        |
|Valeur par défaut    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
La [plage de sous-réseau de cluster](./getting-started-kubernetes-windows.md#cluster-subnet-def).

|  |  | 
|---------|---------|
|Paramètre     | `-ClusterCIDR`        |
|Valeur par défaut    | `10.244.0.0/16`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
L' [adresse IP de service DNS Kubernetes](./getting-started-kubernetes-windows.md#kube-dns-def).

|  |  | 
|---------|---------|
|Paramètre     | `-KubeDnsServiceIP`        |
|Valeur par défaut    | `10.96.0.10`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
Le répertoire dans lequel les journaux kubelet et kube-proxy sont redirigés dans leurs fichiers de sortie respectifs.

|  |  | 
|---------|---------|
|Paramètre     | `-LogDir`        |
|Valeur par défaut    | `C:\k`        |

---


> [!TIP] 
> Doit quelque chose s’est mal passé, veuillez consulter la [section de résolution des problèmes](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services)

## <a name="manual-approach"></a>Approche manuelle
Doit l' [au-dessus de script référencé](#registering-windows-services) ne fonctionne pas pour vous, cette section fournit quelques *exemples de commandes* qui peut être utilisée pour inscrire ces services manuellement pas à pas.

> [!TIP] 
> Pour plus d’informations sur la configuration, consultez [Kubelet et le proxy kube peuvent désormais s’exécuter en tant que services Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services) `kubelet` et `kube-proxy` à s’exécuter en tant que services Windows natifs via `sc`.

### <a name="register-flanneldexe"></a>Inscrire flanneld.exe
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>Inscrire kubelet.exe
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>Inscrire kube-proxy.exe (l2bridge / hôte-EG)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>Inscrire kube-proxy.exe (superposition / vxlan)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```
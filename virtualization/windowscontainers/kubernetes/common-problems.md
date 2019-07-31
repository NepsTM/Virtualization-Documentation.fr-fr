---
title: Résolution des problèmes Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Solutions aux problèmes courants lors du déploiement de Kubernetes et de la jonction de nœuds Windows.
keywords: kubernetes, 1,14, Linux, compile
ms.openlocfilehash: bdf1fd78bbbebcad3562872d9e71c961be6c64eb
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883002"
---
# <a name="troubleshooting-kubernetes"></a>Résolution des problèmes Kubernetes #
Cette page décrit plusieurs problèmes courants lors du déploiement, de la mise en réseau et de la configuration de Kubernetes.

> [!tip]
> Proposez une entrée de FAQ en émettant une réservation permanente auprès de [notre référentiel de documentation](https://github.com/MicrosoftDocs/Virtualization-Documentation/).

Cette page est divisée en catégories suivantes:
1. [Questions générales](#general-questions)
2. [Erreurs réseau courantes](#common-networking-errors)
3. [Erreurs Windows courantes](#common-windows-errors)
4. [Erreurs courantes du maître d’Kubernetes](#common-kubernetes-master-errors)

## <a name="general-questions"></a>Questions générales ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>Comment puis-je savoir que le menu Démarrer. ps1 sur Windows s’est terminé correctement? ###
Vous devez voir kubelet, KUBE-proxy et (si vous avez choisi Flannel en tant que solution de réseau) flanneld les processus d’agent d’hébergement qui s’exécutent sur votre nœud, ainsi que les journaux qui s’affichent dans des fenêtres distinctes de PoSh. De plus, votre nœud Windows doit apparaître en tant que «prêt» dans votre cluster Kubernetes.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>Puis-je configurer pour qu’il s’exécute en arrière-plan au lieu de Windows PoSh? ###
À partir de la version 1,11 de Kubernetes, kubelet & Kube-proxy peut être exécuté en tant que [services Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)natifs. Vous pouvez également utiliser d’autres gestionnaires de services tels que [NSSM. exe](https://nssm.cc/) pour toujours exécuter ces processus (flanneld, kubelet & Kube-proxy) en arrière-plan. Pour plus d’informations, consultez la section [services Windows sur Kubernetes](./kube-windows-services.md) .

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>J’ai des difficultés à exécuter les processus Kubernetes en tant que services Windows ###
Pour la résolution des problèmes initiale, vous pouvez utiliser les indicateurs suivants dans [NSSM. exe](https://nssm.cc/) pour rediriger stdout et stderr vers un fichier de sortie:
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
Pour en savoir plus, consultez la rubrique documents d' [utilisation officiels NSSM](https://nssm.cc/usage) .

## <a name="common-networking-errors"></a>Erreurs réseau courantes ##

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>Je reçois des messages d’erreur tels que «hnsCall a échoué dans Win32: la mauvaise disquette est dans le lecteur.» ###
Cette erreur peut se produire lorsque vous effectuez des modifications personnalisées d’objets HNS ou de nouvelles mises à jour de Windows qui introduisent des modifications apportées à HNS sans déchirer les anciens objets SNPD. Il indique qu’un objet SNPD précédemment créé avant la mise à jour n’est pas compatible avec la version HNS actuellement installée.

Sur Windows Server 2019 (et les options suivantes), les utilisateurs peuvent supprimer des objets SNPD en supprimant le fichier HNS. Data 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

Les utilisateurs doivent être en mesure de supprimer directement les points de terminaison ou réseaux SNPD incompatibles:
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Les utilisateurs de Windows Server version 1903 peuvent accéder à l’emplacement de Registre suivant et supprimer les cartes réseau qui commencent par le nom du `vxlan0` réseau `cbr0`(par exemple, ou):
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```


### <a name="my-windows-pods-cannot-ping-external-resources"></a>Les ressources externes ne peuvent pas être testées dans mes modules Windows ###
Les règles de trafic sortant ne sont pas programmées pour le protocole ICMP pour les modules Windows. Néanmoins, TCP/UDP est pris en charge. Lorsque vous tentez de montrer la connectivité aux ressources hors du cluster, `ping <IP>` utilisez les `curl <IP>` commandes correspondantes.

Si vous rencontrez toujours des problèmes, vous devez probablement être attentif à votre configuration réseau dans [CNI. conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) . Vous pouvez toujours modifier ce fichier statique, la configuration est appliquée aux ressources Kubernetes nouvellement créées.

Pourquoi?
L’une des exigences réseau Kubernetes (voir [modèle Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) consiste à faire en sorte que la communication de cluster se produise sans NAT en interne. Pour respecter cette exigence, nous avons une [](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) «consul» pour toutes les communications pour lesquelles il n’est pas nécessaire de procéder à la traduction d’adresses réseau sortante. Toutefois, cela signifie également que vous devez exclure l’adresse IP externe que vous essayez d’interroger à partir de la demande. Le trafic provenant de vos boîtiers Windows est le SNAT’ed correctement pour recevoir une réponse du monde extérieur. De cette façon, votre compte- `cni.conf` rendu doit ressortir comme suit:
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>Mon nœud Windows ne peut pas accéder à un service de coexclusion ###
L’accès à l’élément de passe local à partir du nœud proprement dit échoue. Cette limitation est connue. L’accès n’est pas pris en charge par les autres nœuds ou clients externes.

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>Après quelques temps, les points de terminaison vNICs et HNS des conteneurs sont supprimés ###
Ce problème peut se produire lorsque le `hostname-override` paramètre n’est pas transmis à [Kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/). Pour le résoudre, les utilisateurs doivent transférer le nom d’hôte à Kube-proxy comme suit:
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>En mode Flannel (vxlan), mes gousses rencontrent des problèmes de connectivité après avoir rejoint le nœud ###
Chaque fois qu’un nœud supprimé a été rejoint pour le cluster, flannelD tente d’affecter un nouveau sous-réseau Pod au nœud. Les utilisateurs doivent supprimer les anciens fichiers de configuration de sous-réseau Pod dans les chemins suivants:
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Après le lancement de Start. ps1, Flanneld est bloqué en «attente de création du réseau». ###
De nombreux rapports concernant ce problème sont examinés; Il est très probable qu’il s’agit d’un problème de synchronisation pour le moment où l’adresse IP de gestion du réseau Flannel est définie. Pour contourner ce problème, il suffit de relancer l’application Start. ps1 ou de la relancer manuellement comme suit:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

Il y a également un [PR](https://github.com/coreos/flannel/pull/1042) qui résout ce problème actuellement.


### <a name="on-flannel-host-gw-my-windows-pods-do-not-have-network-connectivity"></a>Sur Flannel (Host-GW), mes boîtiers Windows ne disposent pas de connectivité réseau ###
Si vous voulez utiliser l2bridge pour la mise en réseau (c’est-à-dire, la [passerelle hôte-Flannel](./network-topologies.md#flannel-in-host-gateway-mode)), vous devez vous assurer que l’usurpation d’adresses Mac est activée pour les VM d’hébergement de conteneurs Windows (invités). Pour ce faire, vous devez exécuter l’outil suivant en tant qu’administrateur sur l’ordinateur hébergeant les ordinateurs virtuels (par exemple, fourni pour Hyper-V):

```powershell
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> [!TIP]
> Si vous utilisez un produit basée sur VMware pour répondre à vos besoins en matière de virtualisation, recherchez le [mode de promiscuité](https://kb.vmware.com/s/article/1004099) pour la configuration requise pour l’usurpation Mac.

>[!TIP]
> Si vous déployez Kubernetes sur des ordinateurs virtuels Azure ou IaaS d’autres fournisseurs de Cloud Computing, vous pouvez également utiliser la [mise en réseau](./network-topologies.md#flannel-in-vxlan-mode) de superposition à la place.

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Mes modules Windows ne peuvent pas être lancés en raison d’un/Run/Flannel/subnet.env manquant ###
Cela indique que Flannel n’a pas été lancé correctement. Vous pouvez essayer de redémarrer flanneld. exe, ou copier les fichiers manuellement de `/run/flannel/subnet.env` sur le maître `C:\run\flannel\subnet.env` Kubernetes sur le nœud Windows Worker et de modifier la `FLANNEL_SUBNET` ligne sur le sous-réseau affecté. Par exemple, si le noeud de sous-réseau 10.244.4.1/24 a été attribué:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
Il est plus sûr de laisser flanneld. exe générer ce fichier pour vous.

### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>La connectivité Pod entre les hôtes est interrompue sur le cluster Kubernetes exécuté sur vSphere 
Dans la mesure où vSphere et Flannel réserve le port 4789 (port VXLAN par défaut) pour la mise en réseau de superposition, les paquets peuvent se terminer par une interception. Si vSphere est utilisé pour la mise en réseau de superposition, il doit être configuré pour utiliser un autre port afin de libérer 4789.  


### <a name="my-endpointsips-are-leaking"></a>Mes points de terminaison/IPs sont en fuite ###
Il existe 2 problèmes actuellement connus qui peuvent provoquer la fuite des points de terminaison. 
1.  Le premier [problème connu](https://github.com/kubernetes/kubernetes/issues/68511) est lié à la version 1,11 de Kubernetes. Évitez d’utiliser la version Kubernetes 1.11.0-1.11.2.
2. Le deuxième [problème connu](https://github.com/docker/libnetwork/issues/1950) qui peut provoquer la fuite de points de terminaison est un problème d’accès concurrentiel au stockage des points de terminaison. Pour recevoir le correctif, vous devez utiliser la version d’amarrage du 18,09 ou une version ultérieure.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>Mes gousses ne peuvent pas se lancer en raison de l’échec d’allocation de «réseau: échec d’allocation pour la plage» ###
Cela indique que l’espace d’adresse IP sur votre nœud est épuisé. Pour nettoyer les [points de terminaison perdus](#my-endpointsips-are-leaking), migrez les ressources sur les nœuds concernés & exécutez les commandes suivantes:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Mon nœud Windows ne peut pas accéder à mes services à l’aide de l’adresse IP de service ###
Il s’agit d’une limitation connue de la pile de mise en réseau actuelle sur Windows. Les *gousses* Windows **sont** néanmoins en mesure d’accéder à l’adresse IP du service.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>Aucun adaptateur réseau n'est détecté lors du démarrage de Kubelet ###
La pile de mise en réseau Windows nécessite un adaptateur virtuel afin que la mise en réseau Kubernetes fonctionne. Si les commandes suivantes ne retournent aucun résultat (dans un shell administrateur), la création d'un réseau virtuel &mdash; prérequis nécessaire au fonctionnement de Kubelet &mdash; a échoué:

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

En règle générale, il est judicieux de modifier le paramètre [InterfaceName](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) du script start. ps1, dans les cas où l’adaptateur réseau de l’hôte n’est pas «Ethernet». Dans le cas contraire, consultez la `start-kubelet.ps1` sortie du script pour voir s’il y a des erreurs lors de la création du réseau virtuel. 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Les pods arrêtent de résoudre les requêtes DNS correctement après être restés actifs un certain temps ###
Il existe un problème de mise en cache DNS connu dans la pile réseau de Windows Server, version 1803 (ou une version antérieure) qui peut entraîner l’échec des requêtes DNS. Pour contourner ce problème, vous pouvez définir les valeurs maximales de cache TTL sur zéro à l’aide des clés de Registre suivantes:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>J’ai toujours des problèmes. Que dois-je faire? ### 
Il peut exister des restrictions supplémentaires sur votre réseau ou sur des ordinateurs hôtes empêchant certains types de communication entre les nœuds. Vérifiez que:
  - vous avez correctement configuré votre [topologie réseau](./network-topologies.md) sélectionnée
  - le trafic qui semble émaner des pods est autorisé
  - le trafic HTTP est autorisé, si vous déployez des services web
  - Les paquets provenant de différents protocoles (IE ICMP et TCP/UDP) ne sont pas déplacés


## <a name="common-windows-errors"></a>Erreurs Windows courantes ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Mes pods Kubernetes sont bloqués à «CréationD'UnConteneur» ###
Ce problème peut avoir de nombreuses causes, mais résulte souvent de la mauvaise configuration de l’image pause. Il s’agit d’un symptôme général du problème suivant.


### <a name="when-deploying-docker-containers-keep-restarting"></a>Lors du déploiement, des conteneurs Docker n'arrêtent pas de redémarrer ###
Vérifiez que votre image pause est compatible avec la version de votre système d’exploitation. Les [instructions](./deploying-resources.md) partent du principe que le système d’exploitation et les conteneurs sont la version 1803. Si vous disposez d’une version ultérieure de Windows, par exemple une build Insider, vous devrez ajuster les images en conséquence. Veuillez vous référer au [référentiel Docker](https://hub.docker.com/u/microsoft/) de Microsoft pour les images. Malgré tout, l’image pause Dockerfile et l’exemple de service s'attendront à ce que la balise `:latest` soit attribuée à l’image.


## <a name="common-kubernetes-master-errors"></a>Erreurs courantes du maître d’Kubernetes ##
Le débogage du master Kubernetes s’articule autour de troiscatégories principales (par ordre de probabilité):

  - Quelque chose ne va pas avec les conteneurs système Kubernetes.
  - Quelque chose ne va pas la manière dont `kubelet` s’exécute.
  - Quelque chose ne va pas au niveau du système.

Exécutez `kubectl get pods -n kube-system` pour voir les pods créés par Kubernetes. Cela peut donner des indications concernant les pods défaillants ou ne démarrant pas correctement. Ensuite, exécutez `docker ps -a` pour voir tous les conteneurs qui sauvegardent ces pods. Enfin, pour afficher les résultats bruts des processus, exécutez `docker logs [ID]` sur les conteneurs qui sont suspectés de provoquer le problème.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>Impossible de se connecter au serveur API à l’adresse `https://[address]:[port]`. ###
Le plus souvent, cette erreur indique des problèmes de certificat. Assurez-vous que vous avez généré correctement le fichier de configuration, que les adresses IP qu’il contient correspondent à celles de votre hôte et que vous l’avez copié dans le répertoire qui est monté par le serveur API.

Si nous vous conseillons d’en suivre les [instructions](./creating-a-linux-master.md), vous pouvez trouver:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 dans le cas contraire, consultez le fichier manifeste du serveur d’API pour vérifier les points de montage.
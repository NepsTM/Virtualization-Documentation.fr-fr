---
title: Résolution des problèmes Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Solutions aux problèmes courants lors du déploiement de Kubernetes et de la jonction de nœuds Windows.
keywords: kubernetes, 1.12, linux, compiler
ms.openlocfilehash: a5e9369b000aa83aa7ec6ec9bb147f0fd844c820
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178912"
---
# <a name="troubleshooting-kubernetes"></a>Résolution des problèmes Kubernetes #
Cette page décrit plusieurs problèmes courants lors du déploiement, de la mise en réseau et de la configuration de Kubernetes.

> [!tip]
> Proposez une entrée de FAQ en émettant une réservation permanente auprès de [notre référentiel de documentation](https://github.com/MicrosoftDocs/Virtualization-Documentation/).

Cette page est divisée en catégories suivantes:
1. [Questions d’ordre générales](#general-questions)
2. [Erreurs réseau courantes](#common-networking-errors)
3. [Erreurs courantes de Windows](#common-windows-errors)
4. [Erreurs courantes de maître Kubernetes](#common-kubernetes-master-errors)

## <a name="general-questions"></a>Questions d’ordre générales ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>Comment savoir start.ps1 sur Windows s’est terminé correctement? ###
Vous devez voir kubelet, kube-proxy, et (si vous choisissez Flannel comme solution de mise en réseau) PoSh windows séparent les processus de l’agent hôte flanneld en cours d’exécution sur le nœud de votre, avec en cours d’exécution qui est affichés dans les journaux. En outre, le nœud de votre Windows doit être répertorié comme «Prêt» dans votre cluster Kubernetes.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>Puis-je configurer pour exécuter tout cela en arrière-plan au lieu de PoSh windows? ###
À partir de Kubernetes version 1.11, kubelet & kube-proxy peuvent être exécutés en tant que native [Services Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services). Vous pouvez également toujours utiliser les gestionnaires de service autre comme [nssm.exe](https://nssm.cc/) à toujours s’exécuter ces processus (flanneld, kubelet et kube-proxy) en arrière-plan pour vous.


## <a name="common-networking-errors"></a>Erreurs réseau courantes ##

### <a name="my-windows-pods-do-not-have-network-connectivity"></a>Mes pods Windows ne disposent pas de connectivité réseau ###
Si vous utilisez toutes les machines virtuelles, assurez-vous que l’usurpation d’identité MAC est activée sur tous les les cartes réseau de machine virtuelle. Consultez la [protection usurpation](./getting-started-kubernetes-windows.md#disable-anti-spoofing-protection) pour plus d’informations.


### <a name="my-windows-pods-cannot-ping-external-resources"></a>Mes pods Windows ne peut pas ping des ressources externes ###
Pods Windows ne sont pas les règles de trafic sortant programmés pour le protocole ICMP aujourd'hui. Toutefois, TCP/UDP est pris en charge. Lorsque vous essayez d’illustrer la connectivité aux ressources en dehors du cluster, veuillez remplacer `ping <IP>` avec correspondant `curl <IP>` commandes.

Si vous êtes toujours confronté à des problèmes, très probablement la configuration de votre réseau dans [cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) mérite certains davantage l’attention. Vous pouvez modifier ce fichier statique, la configuration s’appliqueront à toutes les ressources Kubernetes nouvellement créés.

Pourquoi?
L’une des exigences de mise en réseau Kubernetes (voir le [modèle de Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) est pour la communication de cluster se produire sans NAT en interne. Pour honorer cette exigence, nous avons un [ExceptionList](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) pour toutes les communications où nous ne veulent pas NAT sortant se produire. Toutefois, cela signifie également que vous devez exclure l’adresse IP externe que vous essayez de demander à partir de la ExceptionList. Uniquement le trafic provenant de votre pods Windows sera alors SNAT'ed correctement pour recevoir une réponse depuis l’extérieur. À cet égard, votre ExceptionList dans `cni.conf` doit se présenter comme suit:
```
                "ExceptionList": [
                    "10.244.0.0/16",  # Cluster subnet
                    "10.96.0.0/12",   # Service subnet
                    "10.127.130.0/24" # Management (host) subnet
                ]
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Après le lancement de start.ps1, Flanneld est bloqué dans «En attente pour le réseau doit être créé» ###
Il existe plusieurs rapports de ce problème, qui sont en cours d’étude; Il est très probablement un problème de minutage pour lorsque l’adresse IP de gestion du réseau flannel est définie. Une solution de contournement consiste à simplement relancer start.ps1 ou relancer manuellement comme suit:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

Il existe également une [PR](https://github.com/coreos/flannel/pull/1042) qui résout le problème en cours de validation actuellement.

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Mes pods Windows ne peut pas lancer en raison du manque de /run/flannel/subnet.env ###
Cela indique que Flannel ne démarre pas correctement. Vous pouvez essayer de redémarrer flanneld.exe ou vous pouvez copier les fichiers manuellement de `/run/flannel/subnet.env` sur le nœud maître Kubernetes à `C:\run\flannel\subnet.env` sur le nœud de travail Windows et de modifier le `FLANNEL_SUBNET` ligne à un nombre différent. Par exemple, si vous le souhaitez nœud sous-réseau 10.244.4.1/24:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```

### <a name="my-endpointsips-are-leaking"></a>Mon les points de terminaison/adresses IP sont fuite ###
Il existe 2 problèmes connus qui peuvent provoquer une fuite des points de terminaison. 
1.  Le premier [problème connu](https://github.com/kubernetes/kubernetes/issues/68511) est un problème dans Kubernetes version 1.11. Veuillez Évitez d’utiliser la version 1.11.0 - 1.11.2 de Kubernetes.
2. Le deuxième [problème connu](https://github.com/docker/libnetwork/issues/1950) qui peut provoquer une fuite des points de terminaison est un problème d’accès concurrentiel dans le stockage des points de terminaison. Pour recevoir le correctif, vous devez utiliser Docker EE 18.09 ou une version ultérieure.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>Mes pods ne peut pas lancer dû à «réseau: échec d’allocation pour la plage «erreurs ###
Cela indique que l’espace d’adresse IP sur votre nœud est épuisé. Pour nettoyer les toute [fuite des points de terminaison](#my-endpointsips-are-leaking), veuillez migrer toutes les ressources sur les nœuds concernés et exécutez les commandes suivantes:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Mon nœud Windows ne peut pas accéder à mes services à l’aide de l’adresse IP de service ###
Il s’agit d’une limitation connue de la pile de mise en réseau actuelle sur Windows. Windows *pods* **sont** en mesure d’accéder à l’adresse IP de service toutefois.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>Aucun adaptateur réseau n'est détecté lors du démarrage de Kubelet ###
La pile de mise en réseau Windows nécessite un adaptateur virtuel afin que la mise en réseau Kubernetes fonctionne. Si les commandes suivantes ne retournent aucun résultat (dans un shell administrateur), la création d'un réseau virtuel &mdash; prérequis nécessaire au fonctionnement de Kubelet &mdash; a échoué:

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

consultez la sortie du script `start-kubelet.ps1` pour vérifier si des erreurs surviennent lors de la création du réseau virtuel.

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Les pods arrêtent de résoudre les requêtes DNS correctement après être restés actifs un certain temps ###
Il existe un DNS connus, la mise en cache de problème dans la pile de mise en réseau de Windows Server, version 1803 et qui peut-être parfois entraîner un échec des demandes DNS. Pour contourner ce problème, vous pouvez définir les valeurs de cache de durée de vie maximale à zéro à l’aide de clés de Registre suivantes:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>Je ne vois toujours des problèmes. Que dois-je faire? ### 
Il peut exister des restrictions supplémentaires sur votre réseau ou sur des ordinateurs hôtes empêchant certains types de communication entre les nœuds. Vérifiez que:
  - vous avez correctement configuré votre [topologie de réseau](./network-topologies.md) de choisie
  - le trafic qui semble émaner des pods est autorisé
  - le trafic HTTP est autorisé, si vous déployez des services web
  - Les paquets provenant de différents protocoles (Internet Explorer ICMP ou TCP/UDP) ne sont pas supprimés


## <a name="common-windows-errors"></a>Erreurs courantes de Windows ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Mes pods Kubernetes sont bloqués à «CréationD'UnConteneur» ###
Ce problème peut avoir de nombreuses causes, mais résulte souvent de la mauvaise configuration de l’image pause. Il s’agit d’un symptôme général du problème suivant.


### <a name="when-deploying-docker-containers-keep-restarting"></a>Lors du déploiement, des conteneurs Docker n'arrêtent pas de redémarrer ###
Vérifiez que votre image pause est compatible avec la version de votre système d’exploitation. Les [instructions](./deploying-resources.md) supposent que le système d’exploitation et les conteneurs sont version 1803. Si vous disposez d’une version ultérieure de Windows, par exemple une build Insider, vous devrez ajuster les images en conséquence. Veuillez vous référer au [référentiel Docker](https://hub.docker.com/u/microsoft/) de Microsoft pour les images. Malgré tout, l’image pause Dockerfile et l’exemple de service s'attendront à ce que la balise `:latest` soit attribuée à l’image.


## <a name="common-kubernetes-master-errors"></a>Erreurs courantes de maître Kubernetes ##
Le débogage du master Kubernetes s’articule autour de troiscatégories principales (par ordre de probabilité):

  - Quelque chose ne va pas avec les conteneurs système Kubernetes.
  - Quelque chose ne va pas la manière dont `kubelet` s’exécute.
  - Quelque chose ne va pas au niveau du système.

Exécutez `kubectl get pods -n kube-system` pour voir les pods créés par Kubernetes. Cela peut donner des indications concernant les pods défaillants ou ne démarrant pas correctement. Ensuite, exécutez `docker ps -a` pour voir tous les conteneurs qui sauvegardent ces pods. Enfin, pour afficher les résultats bruts des processus, exécutez `docker logs [ID]` sur les conteneurs qui sont suspectés de provoquer le problème.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>Impossible de se connecter au serveur API à l’adresse `https://[address]:[port]`. ###
Le plus souvent, cette erreur indique des problèmes de certificat. Assurez-vous que vous avez généré correctement le fichier de configuration, que les adresses IP qu’il contient correspondent à celles de votre hôte et que vous l’avez copié dans le répertoire qui est monté par le serveur API.

Si vous continuez [nos instructions](./creating-a-linux-master.md), bonne place pour rechercher qu'il s’agit de:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 dans le cas contraire, reportez-vous au fichier de manifeste du serveur API pour vérifier les points de montage.
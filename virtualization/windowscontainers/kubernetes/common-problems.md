---
title: Résolution des problèmes Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Solutions aux problèmes courants lors du déploiement de Kubernetes et de la jonction de nœuds Windows.
keywords: kubernetes, 1,14, Linux, compiler
ms.openlocfilehash: 471731ec50c7c03816a956bd7aae859ad218be6d
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910449"
---
# <a name="troubleshooting-kubernetes"></a>Résolution des problèmes Kubernetes #
Cette page décrit plusieurs problèmes courants lors du déploiement, de la mise en réseau et de la configuration de Kubernetes.

> [!tip]
> Proposez une entrée de FAQ en émettant une réservation permanente auprès de [notre référentiel de documentation](https://github.com/MicrosoftDocs/Virtualization-Documentation/).

Cette page est sous-divisée en plusieurs catégories :
1. [Questions générales](#general-questions)
2. [Erreurs réseau courantes](#common-networking-errors)
3. [Erreurs Windows courantes](#common-windows-errors)
4. [Erreurs courantes du maître Kubernetes](#common-kubernetes-master-errors)

## <a name="general-questions"></a>Questions générales ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>Comment faire savez que Start. ps1 sur Windows s’est terminé correctement ? ###
Vous devez voir kubelet, KUBE-proxy et (si vous avez choisi Flannel comme solution de mise en réseau) flanneld les processus de l’agent hôte s’exécutant sur votre nœud, avec l’exécution des journaux affichés dans des fenêtres PoSh distinctes. En outre, votre nœud Windows doit être listé comme « prêt » dans votre cluster Kubernetes.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>Puis-je configurer pour exécuter tout cela en arrière-plan au lieu de Windows PoSh ? ###
À compter de Kubernetes version 1,11, kubelet & Kube-proxy peut être exécuté en tant que [services Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)natifs. Vous pouvez également toujours utiliser d’autres gestionnaires de services comme [NSSM. exe](https://nssm.cc/) pour toujours exécuter ces processus (flanneld, kubelet & Kube-proxy) en arrière-plan. Pour obtenir des exemples d’étapes, consultez [services Windows sur Kubernetes](./kube-windows-services.md) .

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>Je rencontre des problèmes lors de l’exécution des processus Kubernetes en tant que services Windows ###
Pour résoudre les problèmes initiaux, vous pouvez utiliser les indicateurs suivants dans [NSSM. exe](https://nssm.cc/) pour rediriger stdout et stderr vers un fichier de sortie :
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
Pour plus d’informations, consultez la documentation relative à [l’utilisation](https://nssm.cc/usage) officielle de NSSM.

## <a name="common-networking-errors"></a>Erreurs réseau courantes ##

### <a name="load-balancers-are-plumbed-inconsistently-across-the-cluster-nodes"></a>Les équilibrages de charge sont montés de manière incohérente entre les nœuds de cluster ###
Sur Windows, KUBE-proxy crée un équilibreur de charge HNS pour chaque service Kubernetes du cluster. Dans la configuration (par défaut) Kube-proxy, les nœuds des clusters contenant plusieurs équilibrages de charge (généralement plus de 100) peuvent manquer de ports TCP éphémères (également appelés plage de ports dynamiques, qui couvre par défaut les ports 49152 à 65535). Cela est dû au grand nombre de ports réservés sur chaque nœud pour chaque équilibrage de charge (non DSR). Ce problème peut se manifester par des erreurs dans Kube-proxy, par exemple :
```
Policy creation failed: hcnCreateLoadBalancer failed in Win32: The specified port already exists.
```

Les utilisateurs peuvent identifier ce problème en exécutant le script [CollectLogs. ps1](https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/debug/collectlogs.ps1) et en consultant les fichiers `*portrange.txt`.

Le `CollectLogs.ps1` imite également la logique d’allocation HNS pour tester la disponibilité de l’allocation du pool de ports dans la plage de ports TCP éphémères, et signale la réussite ou l’échec dans `reservedports.txt`. Le script réserve 10 plages de ports éphémères TCP 64 (pour émuler le comportement de HNS), compte les réussites de réservation & les échecs, puis libère les plages de ports allouées. Un nombre de réussite inférieur à 10 indique que le pool éphémère ne dispose plus de suffisamment d’espace libre. Une synthèse heuristique du nombre de réservations de port de bloc 64 est également générée dans `reservedports.txt`.

Pour résoudre ce problème, quelques étapes sont nécessaires :
1.  Pour une solution permanente, l’équilibrage de charge Kube-proxy doit être défini en [mode DSR](https://techcommunity.microsoft.com/t5/Networking-Blog/Direct-Server-Return-DSR-in-a-nutshell/ba-p/693710). Le mode DSR est entièrement implémenté et disponible sur une version plus récente de [Windows Server Insider 18945](https://blogs.windows.com/windowsexperience/2019/07/30/announcing-windows-server-vnext-insider-preview-build-18945/#o1bs7T2DGPFpf7HM.97) (ou version ultérieure) uniquement.
2. En guise de solution de contournement, les utilisateurs peuvent également augmenter la configuration Windows par défaut des ports éphémères disponibles à l’aide d’une commande telle que `netsh int ipv4 set dynamicportrange TCP <start_port> <port_count>`. *Avertissement :* La substitution de la plage de ports dynamiques par défaut peut avoir des conséquences sur d’autres processus/services sur l’hôte qui reposent sur les ports TCP disponibles à partir de la plage non éphémère. par conséquent, cette plage doit être sélectionnée avec précaution.
3. Il existe une amélioration de l’évolutivité des équilibreurs de charge en mode non DSR utilisant le partage de pool de ports intelligent, qui est planifiée pour être publiée via une mise à jour cumulative au 1er trimestre 2020.

### <a name="hostport-publishing-is-not-working"></a>La publication appelait hostport ne fonctionne pas ###
Il n’est actuellement pas possible de publier des ports à l’aide du champ Kubernetes `containers.ports.hostPort`, car ce champ n’est pas respecté par les plug-ins CNI Windows. Utilisez la publication Deporte pour le moment où les ports doivent être publiés sur le nœud.

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>Je vois des erreurs telles que « échec de hnsCall dans Win32 : la mauvaise disquette est dans le lecteur ». ###
Cette erreur peut se produire lorsque vous apportez des modifications personnalisées aux objets HNS ou que vous installez de nouveaux Windows Update qui introduisent des modifications dans HNS sans détruire les anciens objets HNS. Elle indique qu’un objet HNS précédemment créé avant une mise à jour est incompatible avec la version HNS actuellement installée.

Sur Windows Server 2019 (et versions antérieures), les utilisateurs peuvent supprimer des objets HNS en supprimant le fichier HNS. Data 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

Les utilisateurs doivent être en mesure de supprimer directement les points de terminaison ou réseaux HNS non compatibles :
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Les utilisateurs de Windows Server version 1903 peuvent accéder à l’emplacement de Registre suivant et supprimer toutes les cartes réseau commençant par le nom du réseau (par exemple, `vxlan0` ou `cbr0`) :
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```

### <a name="containers-on-my-flannel-host-gw-deployment-on-azure-cannot-reach-the-internet"></a>Conteneurs sur mon hôte Flannel-le déploiement GW sur Azure ne peut pas accéder à Internet ###
Lors du déploiement de Flannel en mode hôte-GW sur Azure, les paquets doivent traverser le vSwitch de l’hôte physique Azure. Les utilisateurs doivent programmer des [itinéraires définis par l’utilisateur](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined) de type « appliance virtuelle » pour chaque sous-réseau attribué à un nœud. Pour ce faire, vous pouvez utiliser le Portail Azure (voir un exemple [ici](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)) ou via `az` Azure CLI. Voici un exemple de UDR avec le nom « MyRoute » à l’aide des commandes AZ pour un nœud avec IP 10.0.0.4 et le sous-réseau Pod respectif 10.244.0.0/24 :
```
az network route-table create --resource-group <my_resource_group> --name BridgeRoute 
az network route-table route create  --resource-group <my_resource_group> --address-prefix 10.244.0.0/24 --route-table-name BridgeRoute  --name MyRoute --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4 
```

>[!TIP]
> Si vous déployez des Kubernetes sur des machines virtuelles Azure ou IaaS à partir d’autres fournisseurs de Cloud, vous pouvez également utiliser la [mise en réseau de superposition](./network-topologies.md#flannel-in-vxlan-mode) .

### <a name="my-windows-pods-cannot-ping-external-resources"></a>Mes modules Windows ne peuvent pas effectuer de test ping sur les ressources externes ###
Les cadres Windows n’ont pas de règles sortantes programmées pour le protocole ICMP dès aujourd’hui. Toutefois, TCP/UDP est pris en charge. Lorsque vous essayez d’illustrer la connectivité aux ressources en dehors du cluster, remplacez `ping <IP>` par les commandes de `curl <IP>` correspondantes.

Si vous rencontrez toujours des problèmes, il est probable que votre configuration réseau dans [CNI. conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) mérite une attention particulière. Vous pouvez toujours modifier ce fichier statique, la configuration sera appliquée aux ressources Kubernetes nouvellement créées.

Pourquoi ?
L’une des exigences de mise en réseau Kubernetes (voir [modèle Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) concerne la communication en cluster sans NAT en interne. Pour honorer cette exigence, nous avons [une sélection](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) dans le cadre de toutes les communications pour lesquelles nous ne voulons pas que le NAT sortant se produise. Toutefois, cela signifie également que vous devez exclure l’adresse IP externe que vous essayez d’interroger à partir de la nouvelle. Seul le trafic provenant de vos boîtiers Windows sera SNAT’ed correctement pour recevoir une réponse du monde extérieur. À cet égard, votre exceptions dans `cni.conf` doit se présenter comme suit :
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>Mon nœud Windows ne peut pas accéder à un service deexclusion ###
L’accès à l’exclusion locale à partir du nœud lui-même échoue. Cette limitation est connue. L’accès DEPART fonctionne à partir d’autres nœuds ou clients externes.

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>Après un certain temps, les points de terminaison cartes réseau virtuelles et HNS des conteneurs sont en cours de suppression ###
Ce problème peut survenir lorsque le paramètre `hostname-override` n’est pas transmis à [Kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/). Pour le résoudre, les utilisateurs doivent passer le nom d’hôte à Kube-proxy comme suit :
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>En mode Flannel (vxlan), mes Pod rencontrent des problèmes de connectivité après avoir rejoint le nœud ###
Chaque fois qu’un nœud précédemment supprimé est rejoint au cluster, flannelD tente d’affecter un nouveau sous-réseau Pod au nœud. Les utilisateurs doivent supprimer les anciens fichiers de configuration de sous-réseau Pod dans les chemins d’accès suivants :
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Après le lancement de Start. ps1, Flanneld est bloqué dans « en attente de la création du réseau » ###
De nombreux rapports de ce problème sont en cours d’examen. Il s’agit probablement d’un problème de synchronisation lors de la définition de l’adresse IP de gestion du réseau Flannel. Pour contourner ce problème, il suffit de relancer Start. ps1 ou de le relancer manuellement comme suit :
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

Il y a également un [PR](https://github.com/coreos/flannel/pull/1042) qui résout ce problème en cours d’examen.


### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Mes modules Windows ne peuvent pas être lancés en raison d’un/Run/Flannel/subnet.env manquant ###
Cela indique que Flannel n’a pas été lancé correctement. Vous pouvez essayer de redémarrer flanneld. exe, ou vous pouvez copier les fichiers manuellement à partir de `/run/flannel/subnet.env` sur le maître Kubernetes pour `C:\run\flannel\subnet.env` sur le nœud Worker de Windows et modifier la ligne `FLANNEL_SUBNET` sur le sous-réseau qui a été affecté. Par exemple, si le sous-réseau de nœud 10.244.4.1/24 a été affecté :
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
Il est plus sûr de laisser flanneld. exe générer ce fichier pour vous.


### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>La connectivité Pod-Pod entre les hôtes est interrompue sur mon cluster Kubernetes s’exécutant sur vSphere 
Étant donné que vSphere et Flannel réservent le port 4789 (port VXLAN par défaut) pour la mise en réseau de superposition, les paquets peuvent finir par être interceptés. Si vSphere est utilisé pour la mise en réseau de superposition, il doit être configuré pour utiliser un autre port afin de libérer 4789.  


### <a name="my-endpointsips-are-leaking"></a>Mes points de terminaison/adresses IP sont fuites ###
Il existe 2 problèmes actuellement connus qui peuvent entraîner une fuite des points de terminaison. 
1.  Le premier [problème connu](https://github.com/kubernetes/kubernetes/issues/68511) est un problème dans la version 1,11 de Kubernetes. Évitez d’utiliser Kubernetes version 1.11.0-1.11.2.
2. Le deuxième [problème connu](https://github.com/docker/libnetwork/issues/1950) pouvant entraîner une fuite des points de terminaison est un problème d’accès concurrentiel dans le stockage des points de terminaison. Pour recevoir le correctif, vous devez utiliser l’ancrage EE 18,09 ou version ultérieure.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>Impossible de lancer mes pod en raison d’erreurs « réseau : échec d’allocation pour la plage » ###
Cela indique que l’espace d’adressage IP sur votre nœud est utilisé. Pour nettoyer les [points de terminaison perdus](#my-endpointsips-are-leaking), effectuez une migration des ressources sur les nœuds concernés & exécutez les commandes suivantes :
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Mon nœud Windows ne peut pas accéder à mes services à l’aide de l’adresse IP de service ###
Il s’agit d’une limitation connue de la pile de mise en réseau actuelle sur Windows. Toutefois, les *Pod* Windows **sont** en mesure d’accéder à l’adresse IP du service.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>Aucun adaptateur réseau n'est détecté lors du démarrage de Kubelet ###
La pile de mise en réseau Windows nécessite un adaptateur virtuel afin que la mise en réseau Kubernetes fonctionne. Si les commandes suivantes ne retournent aucun résultat (dans un shell administrateur), la création d'un réseau virtuel &mdash; prérequis nécessaire au fonctionnement de Kubelet &mdash; a échoué :

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

Il est souvent utile de modifier le paramètre [NomInterface](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) du script start. ps1, dans les cas où la carte réseau de l’hôte n’est pas « Ethernet ». Dans le cas contraire, consultez la sortie du script `start-kubelet.ps1` pour voir s’il existe des erreurs lors de la création du réseau virtuel. 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Les pods arrêtent de résoudre les requêtes DNS correctement après être restés actifs un certain temps ###
La pile de mise en réseau de Windows Server, version 1803 et inférieure peut parfois provoquer l’échec des requêtes DNS. Pour contourner ce problème, vous pouvez définir les valeurs MAX TTL cache sur zéro à l’aide des clés de Registre suivantes :

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>Je constate toujours des problèmes. Que faire ? ### 
Il peut exister des restrictions supplémentaires sur votre réseau ou sur des ordinateurs hôtes empêchant certains types de communication entre les nœuds. Vérifiez que :
  - vous avez correctement configuré la topologie de votre [réseau](./network-topologies.md) choisie
  - le trafic qui semble émaner des pods est autorisé
  - le trafic HTTP est autorisé, si vous déployez des services web
  - Les paquets de différents protocoles (IE ICMP ou TCP/UDP) ne sont pas supprimés

>[!TIP]
> Pour obtenir des ressources d’auto-assistance supplémentaires, un guide de dépannage Kubernetes pour Windows est également [disponible ici](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648).

## <a name="common-windows-errors"></a>Erreurs Windows courantes ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Mes pods Kubernetes sont bloqués à « CréationD'UnConteneur » ###
Ce problème peut avoir de nombreuses causes, mais résulte souvent de la mauvaise configuration de l’image pause. Il s’agit d’un symptôme général du problème suivant.


### <a name="when-deploying-docker-containers-keep-restarting"></a>Lors du déploiement, des conteneurs Docker n'arrêtent pas de redémarrer ###
Vérifiez que votre image pause est compatible avec la version de votre système d’exploitation. Les [instructions](./deploying-resources.md) partent du principe que le système d’exploitation et les conteneurs sont de la version 1803. Si vous disposez d’une version ultérieure de Windows, par exemple une build Insider, vous devrez ajuster les images en conséquence. Veuillez vous référer au [référentiel Docker](https://hub.docker.com/u/microsoft/) de Microsoft pour les images. Malgré tout, l’image pause Dockerfile et l’exemple de service s'attendront à ce que la balise `:latest` soit attribuée à l’image.


## <a name="common-kubernetes-master-errors"></a>Erreurs courantes du maître Kubernetes ##
Le débogage du master Kubernetes s’articule autour de trois catégories principales (par ordre de probabilité) :

  - Quelque chose ne va pas avec les conteneurs système Kubernetes.
  - Quelque chose ne va pas la manière dont `kubelet` s’exécute.
  - Quelque chose ne va pas au niveau du système.

Exécutez `kubectl get pods -n kube-system` pour voir les pods créés par Kubernetes. Cela peut donner des indications concernant les pods défaillants ou ne démarrant pas correctement. Ensuite, exécutez `docker ps -a` pour voir tous les conteneurs qui sauvegardent ces pods. Enfin, pour afficher les résultats bruts des processus, exécutez `docker logs [ID]` sur les conteneurs qui sont suspectés de provoquer le problème.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>Impossible de se connecter au serveur API à l’adresse `https://[address]:[port]`. ###
Le plus souvent, cette erreur indique des problèmes de certificat. Assurez-vous que vous avez généré correctement le fichier de configuration, que les adresses IP qu’il contient correspondent à celles de votre hôte et que vous l’avez copié dans le répertoire qui est monté par le serveur API.

Si vous suivez [nos instructions](./creating-a-linux-master.md), les bons emplacements à trouver sont les suivants :   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 Sinon, reportez-vous au fichier manifeste du serveur d’API pour vérifier les points de montage.

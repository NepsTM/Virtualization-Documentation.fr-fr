---
title: Résolution des problèmes Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Solutions aux problèmes courants lors du déploiement de Kubernetes et de la jonction de nœuds Windows.
keywords: kubernetes, 1,14, Linux, compile
ms.openlocfilehash: 54396f4b350fa7dfe59e073601f41b0a73f06dca
ms.sourcegitcommit: 76dce6463e820420073dda2dbad822ca4a6241ef
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2019
ms.locfileid: "10307262"
---
# Résolution des problèmes Kubernetes #
Cette page décrit plusieurs problèmes courants lors du déploiement, de la mise en réseau et de la configuration de Kubernetes.

> [!tip]
> Proposez une entrée de FAQ en émettant une réservation permanente auprès de [notre référentiel de documentation](https://github.com/MicrosoftDocs/Virtualization-Documentation/).

Cette page est divisée en catégories suivantes:
1. [Questions générales](#general-questions)
2. [Erreurs réseau courantes](#common-networking-errors)
3. [Erreurs Windows courantes](#common-windows-errors)
4. [Erreurs courantes du maître d’Kubernetes](#common-kubernetes-master-errors)

## Questions générales ##

### Comment puis-je savoir que le menu Démarrer. ps1 sur Windows s’est terminé correctement? ###
Vous devez voir kubelet, KUBE-proxy et (si vous avez choisi Flannel en tant que solution de réseau) flanneld les processus d’agent d’hébergement qui s’exécutent sur votre nœud, ainsi que les journaux qui s’affichent dans des fenêtres distinctes de PoSh. De plus, votre nœud Windows doit apparaître en tant que «prêt» dans votre cluster Kubernetes.

### Puis-je configurer pour qu’il s’exécute en arrière-plan au lieu de Windows PoSh? ###
À partir de la version 1,11 de Kubernetes, kubelet & Kube-proxy peut être exécuté en tant que [services Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)natifs. Vous pouvez également utiliser d’autres gestionnaires de services tels que [NSSM. exe](https://nssm.cc/) pour toujours exécuter ces processus (flanneld, kubelet & Kube-proxy) en arrière-plan. Pour plus d’informations, consultez la section [services Windows sur Kubernetes](./kube-windows-services.md) .

### J’ai des difficultés à exécuter les processus Kubernetes en tant que services Windows ###
Pour la résolution des problèmes initiale, vous pouvez utiliser les indicateurs suivants dans [NSSM. exe](https://nssm.cc/) pour rediriger stdout et stderr vers un fichier de sortie:
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
Pour en savoir plus, consultez la rubrique documents d' [utilisation officiels NSSM](https://nssm.cc/usage) .

## Erreurs réseau courantes ##

### Les équilibreurs de charge sont montés de manière incohérente sur les nœuds de cluster ###
Dans la configuration (par défaut) Kube-proxy, les clusters contenant 100 + équilibreurs de charge peuvent manquer sur les ports TCP éphémères disponibles (alias plage de ports dynamiques, qui couvre généralement les ports 49152 à 65535) en raison du nombre élevé de ports réservés sur chaque nœud pour chaque équilibrage de charge (non-DSR). Cela risque de se manifester par le biais d’erreurs dans Kube-proxy comme suit:
```
Policy creation failed: hcnCreateLoadBalancer failed in Win32: The specified port already exists.
```

Les utilisateurs peuvent identifier ce problème en exécutant le script [CollectLogs. ps1](https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/debug/collectlogs.ps1) et `*portrange.txt` en consultant les fichiers.

La `CollectLogs.ps1` logique de répartition de l’allocation de l’allocation de la part des ressources de l’allocation de port TCP est également mise en rapport `reservedports.txt`avec la réussite et l’échec du test. Le script réserve 10 plages de ports éphémères TCP 64 (pour émuler le comportement de HNS), compte les réussites de réservation & les échecs, puis libère les plages de port affectées. Un nombre de réussite inférieur à 10 indique que le pool éphémère ne dispose pas de l’espace disponible. Un résumé de la quantité d’informations de réservation de port 64 (approximativement disponible) est également généré dans `reservedports.txt`.

Pour résoudre ce problème, vous pouvez prendre les mesures suivantes:
1.  Dans le cas d’une solution permanente, l’équilibrage de charge Kube-proxy doit être défini sur le [mode DSR](https://techcommunity.microsoft.com/t5/Networking-Blog/Direct-Server-Return-DSR-in-a-nutshell/ba-p/693710). Le mode DSR est malheureusement entièrement implémenté sur [Windows Server Insider version 18945](https://blogs.windows.com/windowsexperience/2019/07/30/announcing-windows-server-vnext-insider-preview-build-18945/#o1bs7T2DGPFpf7HM.97) (ou une version ultérieure) uniquement.
2. Pour contourner ce problème, les utilisateurs peuvent également augmenter la configuration Windows par défaut des ports éphémères disponibles `netsh int ipv4 set dynamicportrange TCP <start_port> <port_count>`à l’aide d’une commande telle que. *Avertissement:* Le remplacement de la plage de ports dynamiques par défaut peut avoir des conséquences sur les autres processus/services de l’hôte qui s’appuient sur les ports TCP disponibles à partir de la plage non éphémère, de sorte que cette plage soit soigneusement sélectionnée.
3. Nous travaillons également sur une amélioration de l’évolutivité des équilibreurs de charge de type non-DSR utilisant le partage de la liste de ports intelligents, qui est planifiée pour être publiée via une mise à jour cumulative du 1er trim 2020.

### La publication HostPort ne fonctionne pas ###
Il n’est pas possible actuellement de publier des ports à `containers.ports.hostPort` l’aide du champ Kubernetes, car ce champ n’est pas respecté par les plug-ins Windows CNI. Pour le moment, vous devez utiliser la publication à l’élément à l’aide de l’éditeur.

### Je reçois des messages d’erreur tels que «hnsCall a échoué dans Win32: la mauvaise disquette est dans le lecteur.» ###
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

### Les conteneurs sur mon Flannel Host-le déploiement de GW sur Azure ne peuvent pas accéder à Internet ###
Lorsque vous déployez Flannel en mode Host-GW sur Azure, les paquets doivent passer par le biais du vSwitch d’hôte physique Azure. Les utilisateurs doivent programmer des [itinéraires définis par l’utilisateur](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined) de type «application virtuelle» pour chaque sous-réseau attribué à un nœud. Vous pouvez effectuer cette opération sur le portail Azure (Voir l’exemple [ci](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)-dessous `az` ) ou via le CLI Azure. Voici un exemple de UDR avec le nom «MyRoute» en utilisant les commandes AZ pour un nœud avec IP 10.0.0.4 et le sous-réseau Pod correspondant 10.244.0.0/24:
```
az network route-table create --resource-group <my_resource_group> --name BridgeRoute 
az network route-table route create  --resource-group <my_resource_group> --address-prefix 10.244.0.0/24 --route-table-name BridgeRoute  --name MyRoute --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4 
```

>[!TIP]
> Si vous déployez Kubernetes sur des ordinateurs virtuels Azure ou IaaS d’autres fournisseurs de Cloud Computing, vous pouvez également utiliser la [mise en réseau de superposition](./network-topologies.md#flannel-in-vxlan-mode) à la place.

### Les ressources externes ne peuvent pas être testées dans mes modules Windows ###
Les règles de trafic sortant ne sont pas programmées pour le protocole ICMP pour les modules Windows. Néanmoins, TCP/UDP est pris en charge. Lorsque vous tentez de montrer la connectivité aux ressources hors du cluster, `ping <IP>` utilisez les `curl <IP>` commandes correspondantes.

Si vous rencontrez toujours des problèmes, vous devez probablement être attentif à votre configuration réseau dans [CNI. conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) . Vous pouvez toujours modifier ce fichier statique, la configuration est appliquée aux ressources Kubernetes nouvellement créées.

Pourquoi?
L’une des exigences réseau Kubernetes (voir [modèle Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) consiste à faire en sorte que la communication de cluster se produise sans NAT en interne. Pour respecter cette exigence, nous avons une «consul» pour toutes les communications pour lesquelles il n’est pas nécessaire de procéder à la [traduction d’adresses](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) réseau sortante. Toutefois, cela signifie également que vous devez exclure l’adresse IP externe que vous essayez d’interroger à partir de la demande. Le trafic provenant de vos boîtiers Windows est le SNAT’ed correctement pour recevoir une réponse du monde extérieur. De cette façon, votre compte- `cni.conf` rendu doit ressortir comme suit:
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### Mon nœud Windows ne peut pas accéder à un service de coexclusion ###
L’accès à l’élément de passe local à partir du nœud proprement dit échoue. Cette limitation est connue. L’accès n’est pas pris en charge par les autres nœuds ou clients externes.

### Après quelques temps, les points de terminaison vNICs et HNS des conteneurs sont supprimés ###
Ce problème peut se produire lorsque le `hostname-override` paramètre n’est pas transmis à [Kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/). Pour le résoudre, les utilisateurs doivent transférer le nom d’hôte à Kube-proxy comme suit:
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### En mode Flannel (vxlan), mes gousses rencontrent des problèmes de connectivité après avoir rejoint le nœud ###
Chaque fois qu’un nœud supprimé a été rejoint pour le cluster, flannelD tente d’affecter un nouveau sous-réseau Pod au nœud. Les utilisateurs doivent supprimer les anciens fichiers de configuration de sous-réseau Pod dans les chemins suivants:
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### Après le lancement de Start. ps1, Flanneld est bloqué en «attente de création du réseau». ###
De nombreux rapports concernant ce problème sont examinés; Il est très probable qu’il s’agit d’un problème de synchronisation pour le moment où l’adresse IP de gestion du réseau Flannel est définie. Pour contourner ce problème, il suffit de relancer l’application Start. ps1 ou de la relancer manuellement comme suit:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

Il y a également un [PR](https://github.com/coreos/flannel/pull/1042) qui résout ce problème actuellement.


### Mes modules Windows ne peuvent pas être lancés en raison d’un/Run/Flannel/subnet.env manquant ###
Cela indique que Flannel n’a pas été lancé correctement. Vous pouvez essayer de redémarrer flanneld. exe, ou copier les fichiers manuellement de `/run/flannel/subnet.env` sur le maître `C:\run\flannel\subnet.env` Kubernetes sur le nœud Windows Worker et de modifier la `FLANNEL_SUBNET` ligne sur le sous-réseau affecté. Par exemple, si le noeud de sous-réseau 10.244.4.1/24 a été attribué:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
Il est plus sûr de laisser flanneld. exe générer ce fichier pour vous.


### La connectivité Pod entre les hôtes est interrompue sur le cluster Kubernetes exécuté sur vSphere 
Dans la mesure où vSphere et Flannel réserve le port 4789 (port VXLAN par défaut) pour la mise en réseau de superposition, les paquets peuvent se terminer par une interception. Si vSphere est utilisé pour la mise en réseau de superposition, il doit être configuré pour utiliser un autre port afin de libérer 4789.  


### Mes points de terminaison/IPs sont en fuite ###
Il existe 2 problèmes actuellement connus qui peuvent provoquer la fuite des points de terminaison. 
1.  Le premier [problème connu](https://github.com/kubernetes/kubernetes/issues/68511) est lié à la version 1,11 de Kubernetes. Évitez d’utiliser la version Kubernetes 1.11.0-1.11.2.
2. Le deuxième [problème connu](https://github.com/docker/libnetwork/issues/1950) qui peut provoquer la fuite de points de terminaison est un problème d’accès concurrentiel au stockage des points de terminaison. Pour recevoir le correctif, vous devez utiliser la version d’amarrage du 18,09 ou une version ultérieure.

### Mes gousses ne peuvent pas se lancer en raison de l’échec d’allocation de «réseau: échec d’allocation pour la plage» ###
Cela indique que l’espace d’adresse IP sur votre nœud est épuisé. Pour nettoyer les [points de terminaison perdus](#my-endpointsips-are-leaking), migrez les ressources sur les nœuds concernés & exécutez les commandes suivantes:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### Mon nœud Windows ne peut pas accéder à mes services à l’aide de l’adresse IP de service ###
Il s’agit d’une limitation connue de la pile de mise en réseau actuelle sur Windows. Les *gousses* Windows **sont** néanmoins en mesure d’accéder à l’adresse IP du service.

### Aucun adaptateur réseau n'est détecté lors du démarrage de Kubelet ###
La pile de mise en réseau Windows nécessite un adaptateur virtuel afin que la mise en réseau Kubernetes fonctionne. Si les commandes suivantes ne retournent aucun résultat (dans un shell administrateur), la création d'un réseau virtuel &mdash; prérequis nécessaire au fonctionnement de Kubelet &mdash; a échoué:

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

En règle générale, il est judicieux de modifier le paramètre [InterfaceName](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) du script start. ps1, dans les cas où l’adaptateur réseau de l’hôte n’est pas «Ethernet». Dans le cas contraire, consultez la `start-kubelet.ps1` sortie du script pour voir s’il y a des erreurs lors de la création du réseau virtuel. 

### Les pods arrêtent de résoudre les requêtes DNS correctement après être restés actifs un certain temps ###
Il existe un problème de mise en cache DNS connu dans la pile réseau de Windows Server, version 1803 (ou une version antérieure) qui peut entraîner l’échec des requêtes DNS. Pour contourner ce problème, vous pouvez définir les valeurs maximales de cache TTL sur zéro à l’aide des clés de Registre suivantes:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### J’ai toujours des problèmes. Que dois-je faire? ### 
Il peut exister des restrictions supplémentaires sur votre réseau ou sur des ordinateurs hôtes empêchant certains types de communication entre les nœuds. Vérifiez que:
  - vous avez correctement configuré votre [topologie réseau](./network-topologies.md) sélectionnée
  - le trafic qui semble émaner des pods est autorisé
  - le trafic HTTP est autorisé, si vous déployez des services web
  - Les paquets provenant de différents protocoles (IE ICMP et TCP/UDP) ne sont pas déplacés

>[!TIP]
> Pour accéder à des ressources d’aide automatique supplémentaires, le Guide de résolution des problèmes de Kubernetes pour Windows est également [disponible ici](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648).

## Erreurs Windows courantes ##

### Mes pods Kubernetes sont bloqués à «CréationD'UnConteneur» ###
Ce problème peut avoir de nombreuses causes, mais résulte souvent de la mauvaise configuration de l’image pause. Il s’agit d’un symptôme général du problème suivant.


### Lors du déploiement, des conteneurs Docker n'arrêtent pas de redémarrer ###
Vérifiez que votre image pause est compatible avec la version de votre système d’exploitation. Les [instructions](./deploying-resources.md) partent du principe que le système d’exploitation et les conteneurs sont la version 1803. Si vous disposez d’une version ultérieure de Windows, par exemple une build Insider, vous devrez ajuster les images en conséquence. Veuillez vous référer au [référentiel Docker](https://hub.docker.com/u/microsoft/) de Microsoft pour les images. Malgré tout, l’image pause Dockerfile et l’exemple de service s'attendront à ce que la balise `:latest` soit attribuée à l’image.


## Erreurs courantes du maître d’Kubernetes ##
Le débogage du master Kubernetes s’articule autour de troiscatégories principales (par ordre de probabilité):

  - Quelque chose ne va pas avec les conteneurs système Kubernetes.
  - Quelque chose ne va pas la manière dont `kubelet` s’exécute.
  - Quelque chose ne va pas au niveau du système.

Exécutez `kubectl get pods -n kube-system` pour voir les pods créés par Kubernetes. Cela peut donner des indications concernant les pods défaillants ou ne démarrant pas correctement. Ensuite, exécutez `docker ps -a` pour voir tous les conteneurs qui sauvegardent ces pods. Enfin, pour afficher les résultats bruts des processus, exécutez `docker logs [ID]` sur les conteneurs qui sont suspectés de provoquer le problème.


### Impossible de se connecter au serveur API à l’adresse . `https://[address]:[port]` ###
Le plus souvent, cette erreur indique des problèmes de certificat. Assurez-vous que vous avez généré correctement le fichier de configuration, que les adresses IP qu’il contient correspondent à celles de votre hôte et que vous l’avez copié dans le répertoire qui est monté par le serveur API.

Si nous vous conseillons d’en suivre les [instructions](./creating-a-linux-master.md), vous pouvez trouver:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 dans le cas contraire, consultez le fichier manifeste du serveur d’API pour vérifier les points de montage.

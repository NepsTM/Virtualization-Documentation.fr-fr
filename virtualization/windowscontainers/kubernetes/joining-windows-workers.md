---
title: Jonction au cluster de nœuds Windows
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: how-to
description: Joindre un nœud Windows à un cluster Kubernetes avec v 1.14.
keywords: kubernetes, 1,14, Windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 8954e98eeadca648b3d48599a5174c28101a7ccc
ms.sourcegitcommit: bb18e6568393da748a6d511d41c3acbe38c62668
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/12/2020
ms.locfileid: "88161898"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Jonction de nœuds Windows Server à un cluster

Une fois que vous avez [configuré un nœud principal Kubernetes](./creating-a-linux-master.md) et que vous avez [sélectionné la solution réseau de votre choix](./network-topologies.md), vous êtes prêt à joindre les nœuds Windows Server pour former un cluster. Cela nécessite une [préparation sur les nœuds Windows avant d'](#preparing-a-windows-node) être joint.

## <a name="preparing-a-windows-node"></a>Préparation d’un nœud Windows

> [!NOTE]
> Tous les extraits de code dans les sections Windows doivent être exécutés dans PowerShell avec _élévation_ de privilèges.

### <a name="install-docker-requires-reboot"></a>Installer l’ancrage (nécessite un redémarrage)

Kubernetes utilise l' [arrimeur](https://www.docker.com/) en tant que moteur de conteneur, donc nous devons l’installer. Vous pouvez suivre les [instructions Docs officielles](../manage-docker/configure-docker-daemon.md#install-docker), les [instructions Docker](https://store.docker.com/editions/enterprise/docker-ee-server-windows), ou essayer les étapes suivantes :

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

Si vous vous trouvez derrière un serveur proxy, vous devez définir les variables d’environnement PowerShell suivantes :
```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://proxy.example.com:80/", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://proxy.example.com:443/", [EnvironmentVariableTarget]::Machine)
```

Si, après le redémarrage, vous voyez l’erreur suivante :

![Capture d’écran de l’erreur SVC indiquant « erreur lors de la connexion ».](media/docker-svc-error.png)

Démarrez ensuite le service d’ancrage manuellement :

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>Créer l’image « suspendre » (infrastructure)

> [!IMPORTANT]
> Il est important d’être attentif aux images de conteneur en conflit ; le fait de ne pas avoir la balise attendue peut provoquer une `docker pull` image de conteneur incompatible, provoquant [des problèmes de déploiement](./common-problems.md#when-deploying-docker-containers-keep-restarting) tels que l' `ContainerCreating` État indéfini.

Maintenant que `docker` est installé, vous devez préparer une image « pause » qui est utilisée par Kubernetes pour préparer les pods d’infrastructure. Il existe trois étapes à ce propos :
  1. [extraction de l’image](#pull-the-image)
  2. [balisage](#tag-the-image) en tant que Microsoft/serveur : dernière version
  3. et l' [exécuter](#run-the-container)

#### <a name="pull-the-image"></a>Extraire l’image

Extrayez l’image de votre version de Windows spécifique. Par exemple, si vous exécutez Windows Server 2019 :

```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

#### <a name="tag-the-image"></a>Baliser l’image

Les fichiers dockerfile que vous utiliserez plus loin dans ce guide recherchent la `:latest` balise d’image. Étiquetez l’image de serveur que vous venez de déextraire comme suit :

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>Exécuter le conteneur ####
Vérifiez que le conteneur s’exécute réellement sur votre ordinateur :

```powershell
docker run microsoft/nanoserver:latest
```

Un résultat semblable à celui-ci doit s’afficher :

![text](./media/docker-run-sample.png)

> [!tip]
> Si vous ne pouvez pas exécuter le conteneur, consultez [correspondance de la version de l’hôte du conteneur avec l’image conteneur](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>Préparer Kubernetes pour le répertoire Windows ####
Créez un répertoire « Kubernetes pour Windows » pour stocker les fichiers binaires Kubernetes, ainsi que les scripts de déploiement et les fichiers de configuration.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Copier le certificat Kubernetes ####
Copiez le fichier de certificat Kubernetes ( `$HOME/.kube/config` ) [du maître](./creating-a-linux-master.md#collect-cluster-information) vers ce nouveau `C:\k` répertoire.

> [!tip]
> Vous pouvez utiliser des outils tels que [xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy) ou [WinSCP](https://winscp.net/eng/download.php) pour transférer le fichier de configuration entre les nœuds.

#### <a name="download-kubernetes-binaries"></a>Télécharger les fichiers binaires Kubernetes ####
Pour pouvoir exécuter Kubernetes, vous devez tout d’abord télécharger les `kubectl` `kubelet` `kube-proxy` fichiers binaires, et. Vous pouvez les télécharger à partir des liens figurant dans le `CHANGELOG.md` fichier des [versions les plus récentes](https://github.com/kubernetes/kubernetes/releases/).
 - Par exemple, Voici les [binaires de nœud v 1.14](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries).
 - Utilisez un outil tel que [expand-Archive](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) pour extraire l’archive et placer les fichiers binaires dans `C:\k\` .

#### <a name="optional-setup-kubectl-on-windows"></a>Facultatif Configurer kubectl sur Windows ####
Si vous souhaitez contrôler le cluster à partir de Windows, vous pouvez le faire à l’aide de la `kubectl` commande. Tout d’abord, pour rendre `kubectl` disponible en dehors de l' `C:\k\` annuaire, modifiez la `PATH` variable d’environnement :

```powershell
$env:Path += ";C:\k"
```

Si vous souhaitez que cette modification soit permanente, modifiez la variable dans la cible d’ordinateur :

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

Nous allons ensuite vérifier que le [certificat de cluster](#copy-kubernetes-certificate) est valide. Pour définir l’emplacement où `kubectl` recherche le fichier de configuration, vous pouvez passer le `--kubeconfig` paramètre ou modifier la variable d' `KUBECONFIG` environnement. Par exemple, si la configuration se trouve dans `C:\k\config` :

```powershell
$env:KUBECONFIG="C:\k\config"
```

Pour rendre ce paramètre permanent pour l’étendue de l’utilisateur actuel :

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Enfin, pour vérifier si la configuration a été détectée correctement, vous pouvez utiliser :

```powershell
kubectl config view
```

Si vous recevez une erreur de connexion,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Vous devez vérifier l’emplacement de kubeconfig ou essayer de le copier à nouveau.

Si vous ne voyez aucune erreur, le nœud est maintenant prêt à rejoindre le cluster.

## <a name="joining-the-windows-node"></a>Jointure du nœud Windows ##
Selon la [solution de mise en réseau que vous avez choisie](./network-topologies.md), vous pouvez :
1. [Joindre des nœuds Windows Server à un cluster Flannel (vxlan ou Host-GW)](#joining-a-flannel-cluster)
2. [Joindre des nœuds Windows Server à un cluster avec un commutateur TDR](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Jointure d’un cluster Flannel ###
Il existe une collection de scripts de déploiement Flannel sur [ce référentiel Microsoft](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay) qui vous aide à joindre ce nœud au cluster.

Téléchargez le script de [start.ps1Flannel](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) , dont le contenu doit être extrait `C:\k` :

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

En supposant que vous avez [préparé votre nœud Windows](#preparing-a-windows-node)et `c:\k` que votre répertoire se présente comme indiqué ci-dessous, vous êtes prêt à joindre le nœud.

![text](./media/flannel-directory.png)

#### <a name="join-node"></a>Nœud de jointure ####
Pour simplifier le processus de jointure d’un nœud Windows, il vous suffit d’exécuter un seul script Windows pour lancer `kubelet` , `kube-proxy` , `flanneld` et joindre le nœud.

> [!Note]
> [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) référence [install.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1), qui télécharge des fichiers supplémentaires tels que l' `flanneld` exécutable et le [fichier dockerfile pour le pod d’infrastructure](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *et les installe pour vous*. Pour le mode de mise en réseau par recouvrement, le [pare-feu](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111) est ouvert pour le port UDP local 4789. Il peut y avoir plusieurs fenêtres PowerShell ouvertes/fermées, ainsi que quelques secondes de panne réseau pendant que le nouveau vSwitch externe pour le réseau Pod est créé la première fois.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# <a name="managementip"></a>[ManagementIP](#tab/ManagementIP)
Adresse IP affectée au nœud Windows. Vous pouvez utiliser `ipconfig` pour le Rechercher.

| Paramètre | Valeur par défaut|
|---|---|
| `-ManagementIP` | n.A. **Obligatoire** |

# <a name="networkmode"></a>[NetworkMode](#tab/NetworkMode)
Le mode réseau `l2bridge` (hôte Flannel-GW) ou `overlay` (Flannel vxlan) choisi comme [solution réseau](./network-topologies.md).

> [!Important]
> `overlay`le mode de mise en réseau (Flannel vxlan) requiert des binaires Kubernetes v 1.14 (ou version ultérieure) et [KB4489899](https://support.microsoft.com/help/4489899).

| Paramètre | Valeur par défaut |
|---|---|
| `-NetworkMode` | `12bridge` |


# <a name="clustercidr"></a>[ClusterCIDR](#tab/ClusterCIDR)
[Plage du sous-réseau du cluster](./getting-started-kubernetes-windows.md#cluster-subnet-def).

| Paramètre | Valeur par défaut |
|---|---|
| `-ClusterCIDR` | `10.244.0.0/16` |

# <a name="servicecidr"></a>[ServiceCIDR](#tab/ServiceCIDR)
[Plage de sous-réseau de service](./getting-started-kubernetes-windows.md#service-subnet-def).

| Paramètre | Valeur par défaut |
|---|---|
| `-ServiceCIDR` | `10.96.0.0/12` |

# <a name="kubednsserviceip"></a>[KubeDnsServiceIP](#tab/KubeDnsServiceIP)
L' [adresse IP du service DNS Kubernetes](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster).

| Paramètre | Valeur par défaut |
|---|---|
| `-KubeDnsServiceIP` | `10.96.0.10` |

# <a name="interfacename"></a>[InterfaceName](#tab/InterfaceName)
Nom de l’interface réseau de l’hôte Windows. Vous pouvez utiliser `ipconfig` pour le Rechercher.

| Paramètre | Valeur par défaut |
|---|---|
| `-InterfaceName` | `Ethernet` |

# <a name="logdir"></a>[LogDir](#tab/LogDir)
Répertoire dans lequel les journaux kubelet et Kube-proxy sont redirigés dans leurs fichiers de sortie respectifs.

| Paramètre | Valeur par défaut |
|---|---|
| `-LogDir` | `C:\k` |

---

> [!tip]
> Vous avez déjà noté le sous-réseau de cluster, le sous-réseau de service et l’adresse IP Kube-DNS du maître Linux [précédemment](./creating-a-linux-master.md#collect-cluster-information)

Après l’exécution de ce service, vous devez être en mesure d’effectuer les opérations suivantes :
  * Afficher les nœuds Windows joints à l’aide de`kubectl get nodes`
  * Consultez 3 fenêtres PowerShell ouvertes, une pour `kubelet` , une pour `flanneld` et une autre pour`kube-proxy`
  * Consultez processus Host-agent pour `flanneld` , `kubelet` et `kube-proxy` en cours d’exécution sur le nœud.

En cas de réussite, passez aux [étapes suivantes](#next-steps).

## <a name="joining-a-tor-cluster"></a>Jointure d’un cluster TDR ##
> [!NOTE]
> Vous pouvez ignorer cette section si vous avez [déjà](./network-topologies.md#flannel-in-host-gateway-mode)choisi Flannel comme solution de mise en réseau.

Pour ce faire, vous devez suivre les instructions relatives à la [Configuration des conteneurs Windows Server sur Kubernetes pour la topologie de routage L3 en amont](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology). Cela implique de s’assurer que vous configurez votre routeur en amont de telle sorte que le préfixe CIDR de Pod affecté à un nœud corresponde à son adresse IP de nœud respective.

En supposant que le nouveau nœud est répertorié comme « prêt » par `kubectl get nodes` , kubelet + Kube-proxy est en cours d’exécution et que vous avez configuré votre routeur en amont, vous êtes prêt pour les étapes suivantes.

## <a name="next-steps"></a>Étapes suivantes ##
Dans cette section, nous avons abordé la manière de joindre les Workers Windows à notre cluster Kubernetes. Vous êtes maintenant prêt pour l’étape 5 :

> [!div class="nextstepaction"]
> [Rejoindre des travailleurs Linux](./joining-linux-workers.md)

Si vous n’avez aucun travail Linux, vous pouvez également passer directement à l’étape 6 :

> [!div class="nextstepaction"]
> [Déploiement des ressources Kubernetes](./deploying-resources.md)

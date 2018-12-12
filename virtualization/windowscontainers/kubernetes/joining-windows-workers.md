---
title: Jonction de nœuds Windows
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Jonction d’un nœud Windows à un cluster Kubernetes avec v1.12.
keywords: kubernetes, 1.12, windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 8051270cac6178bad9adf9a8ef9e2324932f7d01
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2018
ms.locfileid: "6179016"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Jonction de nœuds de serveur Windows à un Cluster #
Une fois que vous avez [d’installation d’un nœud maître Kubernetes](./creating-a-linux-master.md) et [sélectionné votre solution de réseau de votre choix](./network-topologies.md), vous êtes prêt à joindre des nœuds de Windows Server pour former un cluster. Cela requiert une [préparation sur les nœuds de Windows](#preparing-a-windows-node) avant de joindre.

## <a name="preparing-a-windows-node"></a>Préparation d’un nœud Windows ##
> [!NOTE]  
> Tous les extraits de code dans les sections Windows doivent être exécutés dans PowerShell avec _élévation_ de privilèges.

### <a name="install-docker-requires-reboot"></a>Installer Docker (nécessite un redémarrage) ###
Kubernetes utilise [Docker](https://www.docker.com/) en tant que son moteur de conteneur, afin que nous devons donc l’installer. Vous pouvez suivre les [instructions Docs officielles](../manage-docker/configure-docker-daemon.md#install-docker), les [instructions Docker](https://store.docker.com/editions/enterprise/docker-ee-server-windows), ou essayer les étapes suivantes:

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

Si vous vous trouvez derrière un serveur proxy, vous devez définir les variables d’environnement PowerShell suivantes:
```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://proxy.example.com:80/", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://proxy.example.com:443/", [EnvironmentVariableTarget]::Machine)
```

Si après le redémarrage, vous voyez l’erreur suivante:

![texte](media/docker-svc-error.png)

Lancer manuellement le service docker:

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>Créer l’image «pause» (infrastructure) ###
> [!Important]
> Il est important d’être prudent d’images de conteneur en conflit; n’ayant ne pas la balise attendue peut provoquer un `docker pull` d’une image de conteneur incompatible, à l’origine [des problèmes de déploiement](./common-problems.md#when-deploying-docker-containers-keep-restarting) comme indéterminée `ContainerCreating` état.

Maintenant que `docker` est installé, vous devez préparer une image «pause» qui est utilisée par Kubernetes pour préparer les pods d’infrastructure. Il existe trois étapes à ceci: 
  1. [comment extraire l’image](#pull-the-image)
  2. [son marquage](#tag-the-image) en tant que microsoft / nanoserver:latest
  3. et [en l’exécutant](#run-the-container)


#### <a name="pull-the-image"></a>Extraire l’image ####     
 Extraire l’image pour votre version de Windows spécifique. Par exemple, si vous exécutez Windows Server 2019:

 ```powershell
docker pull microsoft/nanoserver:1803
 ```

#### <a name="tag-the-image"></a>Balise de l’image ####
Recherchez les fichiers Dockerfile que vous utiliserez plus loin dans ce guide la `:latest` balise d’image. Balise de l’image de nanoserver vous simplement extraits comme suit:

```powershell
docker tag microsoft/nanoserver:1803 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>Exécuter le conteneur ####
Vérifiez que le conteneur s’exécute réellement sur votre ordinateur:

```powershell
docker run microsoft/nanoserver:latest
```

Vous devez voir ce qui suit:

![texte](./media/docker-run-sample.png)

> [!tip]
> Si vous ne pouvez pas exécuter le conteneur, voir: [correspondante version d’hôte de conteneur avec une image de conteneur](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>Préparer le répertoire Kubernetes pour Windows ####
Créez un répertoire de «Kubernetes pour Windows» pour stocker les fichiers binaires Kubernetes, ainsi que les scripts de déploiement et les fichiers de configuration.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Copier le certificat de Kubernetes #### 
Copiez le fichier de certificat de Kubernetes (`$HOME/.kube/config`) [à partir du master](./creating-a-linux-master.md#collect-cluster-information) à ce nouvel `C:\k` répertoire.

#### <a name="download-kubernetes-binaries"></a>Télécharger les fichiers binaires Kubernetes ####
Pour être en mesure d’exécuter Kubernetes, vous devez tout d’abord télécharger le `kubectl`, `kubelet`, et `kube-proxy` fichiers binaires. Vous pouvez les télécharger depuis les liens dans le `CHANGELOG.md` fichier des [versions plus récentes](https://github.com/kubernetes/kubernetes/releases/).
 - Par exemple, voici les [fichiers binaires du nœud de v1.12](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.12.md#node-binaries).
 - Utilisez un outil comme [7-Zip](http://www.7-zip.org/) pour extraire l’archive et placer les fichiers binaires dans `C:\k\`.

#### <a name="optional-setup-kubectl-on-windows"></a>(Facultatif) Kubectl le programme d’installation sur Windows ####
Si vous souhaitez contrôler le cluster à partir de Windows, vous pouvez le faire avec les `kubectl` commande. Tout d’abord, pour rendre `kubectl` disponibles en dehors de la `C:\k\` répertoire, modifier le `PATH` variable d’environnement:

```powershell
$env:Path += ";C:\k"
```

Si vous souhaitez que cette modification soit permanente, modifiez la variable dans la cible d’ordinateur:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

Ensuite, nous allons vérifier que le [certificat de cluster](#copy-kubernetes-certificate) est valide. Afin de définir l’emplacement où `kubectl` recherche le fichier de configuration, vous pouvez transmettre la `--kubeconfig` paramètre ou modifier le `KUBECONFIG` variable d’environnement. Par exemple, si la configuration se trouve dans `C:\k\config`:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Pour rendre ce paramètre permanent pour l’étendue de l’utilisateur actuel:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Enfin, pour vérifier si la configuration a été correctement détectée, vous pouvez utiliser:

```powershell
kubectl config view
```

Si vous recevez une erreur de connexion,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Vérifiez l’emplacement kubeconfig ou si vous essayez de copier de nouveau.

Si vous ne voyez aucune erreur le nœud est maintenant prêt à rejoindre le cluster.

## <a name="joining-the-windows-node"></a>Rejoindre le nœud Windows ##
En fonction de la [solution mise en réseau, que vous avez choisi](./network-topologies.md), vous pouvez:
1. [Joignez les nœuds de Windows Server à un cluster de Flannel](#joining-a-flannel-cluster)
2. [Joignez les nœuds de Windows Server à un cluster avec un commutateur ToR](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Jonction d’un cluster Flannel ###
Il existe une collection de scripts de déploiement Flannel sur [ce référentiel Microsoft](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge) qui vous aideront à joindre ce nœud au cluster.

Vous pouvez télécharger le fichier ZIP directement [ici](https://github.com/Microsoft/SDN/archive/master.zip). La seule chose que vous avez besoin est la `Kubernetes/flannel/l2bridge` répertoire, dont le contenu doit être extraits vers `C:\k\`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mv master/SDN-master/Kubernetes/flannel/l2bridge/* C:/k/
rm -recurse -force master,master.zip
```

En outre, vous devez vous assurer que le sous-réseau de cluster (par exemple, à cocher «10.244.0.0/16») est correct dans:
- [NET-conf.json](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/net-conf.json)


En supposant que vous [préparé votre nœud Windows](#preparing-a-windows-node)et votre `c:\k` répertoire se présente comme suit, vous êtes prêt à rejoindre le nœud.

![texte](./media/flannel-directory.png)

#### <a name="join-node"></a>Joindre nœud #### 
Pour simplifier le processus de jonction d’un nœud Windows, il vous suffit d’exécuter un script Windows unique pour lancer `kubelet`, `kube-proxy`, `flanneld`et joindre le nœud.

> [!Note]
> [Ce script](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1) à télécharger les fichiers supplémentaires telles que la mise à jour `flanneld` exécutable et les [fichiers Dockerfile pour pod d’infrastructure](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *et exécuter celles pour vous*. Il peut exister plusieurs windows powershell d’être ouvert/fermé, ainsi que quelques secondes de panne réseau pendant que le commutateur virtuel externe pour le réseau de pod l2bridge est créé la première fois.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> 
```

> [!tip]
> Déjà noté vers le bas du sous-réseau de cluster, du sous-réseau de service et IP kube-DNS à partir du master Linux [antérieures](./creating-a-linux-master.md#collect-cluster-information)

Après l’exécution de cela, vous devez être en mesure de:
  * Vue joint à l’aide des nœuds de Windows `kubectl get nodes`
  * Voir la rubrique 3 windows powershell, une pour `kubelet`, une pour `flanneld`et l’autre pour `kube-proxy`
  * Voir les processus de l’agent hôte pour `flanneld`, `kubelet`, et `kube-proxy` en cours d’exécution sur le nœud


## <a name="joining-a-tor-cluster"></a>Jonction d’un cluster ToR ##
> [!NOTE]
> Vous pouvez ignorer cette section si vous avez choisi Flannel en tant que votre réseau solution [précédemment](./network-topologies.md#flannel-in-host-gateway-mode).

Pour ce faire, vous devez suivre les instructions pour [la définition des conteneurs Windows Server sur Kubernetes pour la topologie de routage de couche 3 en amont](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology). Cela inclut l’établissement que vous avez configuré votre routeur en amont tels que le pod CIDR préfixe attribuées à un nœud est mappé à son adresse IP nœud respectifs.

En supposant que le nouveau nœud est répertorié comme «Prêt» par `kubectl get nodes`, kubelet + kube-proxy est en cours d’exécution et que vous avez configuré votre routeur ToR en amont, vous êtes prêt pour les étapes suivantes.

## <a name="next-steps"></a>Étapes suivantes ##
Dans cette section, nous avons abordé comment joindre des travailleurs de Windows à notre cluster Kubernetes. Vous êtes maintenant prêt à l’étape 5:

> [!div class="nextstepaction"]
> [Jonction de nœuds de travail Linux](./joining-linux-workers.md)

Par ailleurs, si vous n’avez pas n’importe quel travailleurs Linux n’hésitez pas à passez directement à l’étape 6:

> [!div class="nextstepaction"]
> [Déploiement des ressources de Kubernetes](./deploying-resources.md)
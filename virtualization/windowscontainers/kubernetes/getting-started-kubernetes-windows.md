---
title: Kubernetes sur Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: Jonction d’un nœud Windows à un cluster Kubernetes avec la version bêta1.9.
keywords: Kubernetes, 1.9, Windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c6127fe8ab9de6a56816fb8187d4dec525425510
ms.sourcegitcommit: 7c3af076eb8bad98e1c3de0af63dacd842efcfa3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/07/2018
---
# <a name="kubernetes-on-windows"></a>Kubernetes sur Windows #

Avec la dernière version de Kubernetes1.9 et Windows Server [version1709](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1709#networking), les utilisateurs peuvent tirer parti des fonctionnalités les plus récentes de la mise en réseau Windows:

  - **compartiments de pod partagés**: les pods d’infrastructure et de travail partagent désormais un compartiment réseau (comparable à un espace de noms Linux)
  - **optimisation du point de terminaison**: grâce au partage des compartiments, les services de conteneurs doivent suivre (au moins) deux fois moins de points de terminaison qu’auparavant
  - **optimisation du chemin d’accès aux données**: les améliorations apportées à la plateforme de filtrage virtuel (VFP, Virtual Filtering Platform) et au service HNS (Host Networking Service) permettent un équilibrage de charge basé sur le noyau


Cette page explique comment joindre un tout nouveau nœud Windows à un cluster Linux existant. Pour partir entièrement de zéro, reportez-vous à [cette page](./creating-a-linux-master.md) &mdash; l’une des nombreuses ressources disponibles pour le déploiement d’un cluster Kubernetes &mdash; pour configurer un master à partir de zéro de la même façon que nous l’avons fait.

> [!TIP] 
> Si vous souhaitez déployer un cluster sur Azure, l’outil open source ACS-Engine facilite cette opération. Une [procédure pas à pas](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/windows.md) est disponible.

<a name="definitions"></a> Voici les définitions de quelques termes auxquels il est fait référence dans ce guide:

  - Le **réseau externe** est le réseau sur lequel vos nœuds communiquent.
  - <a name="cluster-subnet-def"></a>Le **sous-réseau de cluster** est un réseau virtuel routable; les nœuds sont attribués à des sous-réseaux plus petits pour permettre l’utilisation de leurs pods.
  - Le **sous-réseau de service** est un sous-réseau non routable et purement virtuel sur 11.0/16, que les pods utilisent pour accéder uniformément aux services sans se préoccuper de la topologie du réseau. Il est converti vers/depuis l’espace d’adressage routable par `kube-proxy` en cours d’exécution sur les nœuds.

## <a name="what-you-will-accomplish"></a>Les tâches que vous allez accomplir ##

À la fin de ce guide, vous aurez:

> [!div class="checklist"]  
> * Configuré un nœud master [Linux](#preparing-the-linux-master).  
> * Joint un [nœud de travail Windows](#preparing-a-windows-node) à un nœud master.  
> * Préparé notre [topologie de réseau](#network-topology).  
> * Déployé un [exemple de service Windows](#running-a-sample-service).  
> * Abordé les [erreurs et problèmes courants](./common-problems.md).  

## <a name="preparing-the-linux-master"></a>Préparation du master Linux ##

Que vous ayez suivi [les instructions](./creating-a-linux-master.md) ou que vous disposiez déjà d’un cluster existant, la seule chose dont le master Linux ait besoin est la configuration de certificat de Kubernetes. Celle-ci peut se trouver dans `/etc/kubernetes/admin.conf`, `~/.kube/config`, ou dans un autre emplacement, en fonction de votre installation.

## <a name="preparing-a-windows-node"></a>Préparation d’un nœud Windows ##

> [!NOTE]  
> Tous les extraits de code dans les sections Windows doivent être exécutés dans PowerShell avec _élévation_ de privilèges.

Kubernetes utilise [Docker](https://www.docker.com/) comme orchestrateur de conteneur. Nous devons donc l’installer. Vous pouvez suivre les [instructions Docs officielles](../manage-docker/configure-docker-daemon.md#install-docker), les [instructions Docker](https://store.docker.com/editions/enterprise/docker-ee-server-windows), ou essayer les étapes suivantes:

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

Il existe une collection de scripts sur [ce référentiel Microsoft](https://github.com/Microsoft/SDN) qui vous aideront à joindre ce nœud au cluster. Vous pouvez télécharger le fichier ZIP directement [ici](https://github.com/Microsoft/SDN/archive/master.zip). La seule chose dont vous ayez besoin est le dossier `Kubernetes/windows`, dont le contenu doit être déplacé sur `C:\k\`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mkdir C:/k/
mv master/SDN-master/Kubernetes/windows/* C:/k/
rm -recurse -force master,master.zip
```

Copiez le fichier de certificat [identifié précédemment](#preparing-the-linux-master) dans ce nouveau répertoire `C:\k`.

## <a name="network-topology"></a>Topologie réseau ##

Il existe plusieurs façons de rendre le [sous-réseau de cluster](#cluster-subnet-def) virtuel routable. Vous pouvez:

  - Configurer le [mode hôte-passerelle](./configuring-host-gateway-mode.md), en définissant des itinéraires statiques de tronçon suivant entre les nœuds pour permettre la communication de pod à pod.
  - Configurer un commutateur top-of-rack (ToR) intelligent pour router le sous-réseau.
  - Utilisez un plug-in de superposition tiers comme [Flannel](https://coreos.com/flannel/docs/latest/kubernetes.html) (la prise en charge de Windows pour Flannel est en version bêta).

### <a name="creating-the-pause-image"></a>Création de l’image «Pause» ###

Maintenant que `docker` est installé, vous devez préparer une image «pause» qui est utilisée par Kubernetes pour préparer les pods d’infrastructure.

```powershell
docker pull microsoft/windowsservercore:1709
docker tag microsoft/windowsservercore:1709 microsoft/windowsservercore:latest
cd C:/k/
docker build -t kubeletwin/pause .
```

> [!NOTE]
> Nous lui attribuons la balise `:latest`, car l’exemple de service que vous déploierez ultérieurement en dépend, même si en réalité, il peut ne pas _s’agir_ de la dernière image WindowsServerCore disponible. Il est important de prendre garde aux images de conteneuren conflit; ne pas attribuer la balise attendue peut provoquer un `docker pull` d’une image de conteneur incompatible, entraînant ainsi des [problèmes de déploiement](./common-problems.md#when-deploying-docker-containers-keep-restarting). 


### <a name="downloading-binaries"></a>Téléchargement des fichiers binaires ###
Pendant le déroulement du `pull`, téléchargez les fichiers binaires côté client suivants à partir de Kubernetes:

  - `kubectl.exe`
  - `kubelet.exe`
  - `kube-proxy.exe`

Vous pouvez les télécharger depuis les liens se trouvant dans le fichier `CHANGELOG.md` de la dernière version1.9. À ce jour, à savoir [1.9.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1), et les fichiers binaires Windows sont [ici](https://storage.googleapis.com/kubernetes-release/release/v1.9.1/kubernetes-node-windows-amd64.tar.gz). Utilisez un outil comme [7-Zip](http://www.7-zip.org/) pour extraire l’archive et placer les fichiers binaires dans `C:\k\`.

Afin de rendre la commande `kubectl` disponible en dehors du répertoire `C:\k\`, modifiez la variable d’environnement `PATH`:

```powershell
$env:Path += ";C:\k"
```

Si vous souhaitez que cette modification soit permanente, modifiez la variable dans la cible d’ordinateur:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

### <a name="joining-the-cluster"></a>Rejoindre le cluster ###
Vérifiez que la configuration du cluster est valide à l’aide de:

```powershell
kubectl version
```

Si vous recevez une erreur de connexion,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

vérifiez si la configuration a été correctement détectée:

```powershell
kubectl config view
```

Pour modifier l’emplacement où `kubectl` recherche le fichier de configuration, vous pouvez transmettre le paramètre `--kubeconfig` ou modifier la variable d’environnement `KUBECONFIG`. Par exemple, si la configuration se trouve dans `C:\k\config`:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Pour rendre ce paramètre permanent pour l’étendue de l’utilisateur actuel:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Le nœud est maintenant prêt à rejoindre le cluster. Exécutez ces scripts (dans cet ordre) dans deux fenêtres PowerShell *avec élévation de privilèges* distinctes. Le paramètre `-ClusterCidr` dans le premier script est le [sous-réseau de cluster](#cluster-subnet-def) configuré; ici, il s’agit de `192.168.0.0/16`.

```powershell
./start-kubelet.ps1 -ClusterCidr 192.168.0.0/16
./start-kubeproxy.ps1
```

Le nœud Windows sera visible à partir du master Linux sous `kubectl get nodes` d’ici une minute!


### <a name="validating-your-network-topology"></a>Validation de la topologie de votre réseau ###

Il existe quelques tests de base permettant de valider une configuration de réseau appropriée:

  - **Connectivité de nœud à nœud**: les pings entre les nœuds master et de travail Windows doivent réussir dans les deux sens.

  - **Connectivité de sous-réseau de pod à nœud**: pings entre l’interface de pod virtuel et les nœuds. Recherchez l’adresse de la passerelle sous `route -n` et `ipconfig` sur Linux et Windows, respectivement, en recherchant l’interface `cbr0`.

Si l’un de ces tests de base ne fonctionne pas, essayez la [page de dépannage](./common-problems.md#common-networking-errors) pour résoudre les problèmes courants.


## <a name="running-a-sample-service"></a>Exécution d’un exemple de service ##

Vous allez déployer un [service Web basé sur PowerShell](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) très simple pour vous assurer que vous avez rejoint le cluster avec succès et que notre réseau est correctement configuré.

Sur le master Linux, téléchargez et exécutez le service:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/WebServer.yaml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Cela crée un déploiement et un service. Regardez ensuite les pods indéfiniment pour effectuer le suivi de leur état; appuyez simplement sur `Ctrl+C` pour quitter la commande `watch` lorsque vous avez terminé l’observation.

Si tout va bien, il est possible de:

  - voir 4conteneurs sous une commande `docker ps` au niveau du nœud Windows
  - voir 2pods sous une commande `kubectl get pods` à partir du master Linux
  - `curl` sur les adresses IP *pod* sur le port80, le master Linux obtient une réponse du serveur Web; cela montre que la communication de nœud à pod sur le réseau est correcte.
  - effectuer un ping *entre pods* (notamment sur les ordinateurs hôtes, si vous disposez de plusieurs nœuds Windows) via `docker exec`; cela montre que la communication de pod à pod est correcte.
  - `curl` l’adresse *IP de service* virtuelle (visible sous `kubectl get services`) à partir du master Linux et à partir de pods individuels.
  - `curl` le *nom du service* avec le [suffixe DNS par défaut](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services) Kubernetes, illustrant la fonctionnalité DNS.

> [!Warning]  
> Les nœuds de Windows ne seront pas en mesure d’accéder à l’adresse IP de service. Il s’agit d’une [limitation de plateforme connue](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) qui sera améliorée dans la prochaine mise à jour de Windows Server.


### <a name="port-mapping"></a>Mappage de ports ### 
Il est également possible d’accéder aux services hébergés dans les pods au travers de leurs nœuds respectifs en mappant un port sur le nœud. Il existe un [autre exemple YAML disponible](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml) avec le mappage du port4444 sur le nœud au port80 sur le nœud qui montre cette fonctionnalité. Pour le déployer, suivez les mêmes étapes que précédemment:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

Il devrait maintenant être possible de `curl` sur l'adresse IP du *nœud* sur le port4444 et recevoir une réponse du serveur web. N’oubliez pas que cela limite la montée en charge à un seul pod par nœud dans la mesure où cela oblige à un mappage de un-à-un.

---
title: Kubernetes sur Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "Jonction d’un nœud Windows à un cluster Kubernetes avec la version bêta1.9."
keywords: Kubernetes, 1.9, Windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: f1b832f8a21c034582e157342acf7826fb7b6ea3
ms.sourcegitcommit: b0e21468f880a902df63ea6bc589dfcff1530d6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/17/2018
---
# <a name="kubernetes-on-windows"></a>Kubernetes sur Windows #
Avec la dernière version de Kubernetes1.9 et Windows Server [version1709](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1709#networking), les utilisateurs peuvent tirer parti des fonctionnalités les plus récentes de la mise en réseau Windows:

  - **compartiments de pod partagés**: les pods d’infrastructure et de travail partagent désormais un compartiment réseau (comparable à un espace de noms Linux)
  - **optimisation du point de terminaison**: grâce au partage des compartiments, les services de conteneurs doivent suivre (au moins) deux fois moins de points de terminaison qu’auparavant
  - **optimisation du chemin d’accès aux données**: les améliorations apportées à la plateforme de filtrage virtuel (VFP, Virtual Filtering Platform) et au service HNS (Host Networking Service) permettent un équilibrage de charge basé sur le noyau


Cette page explique comment joindre un tout nouveau nœud Windows à un cluster Linux existant. Pour partir entièrement de zéro, reportez-vous à [cette page](./creating-a-linux-master.md) &mdash; l’une des nombreuses ressources disponibles pour le déploiement d’un cluster Kubernetes &mdash; pour configurer un master à partir de zéro de la même façon que nous l’avons fait.


<a name="definitions"></a> Voici les définitions de quelques termes auxquels il est fait référence dans ce guide:

  - Le **réseau externe** est le réseau sur lequel vos nœuds communiquent.
  - <a name="cluster-subnet-def"></a>Le **sous-réseau de cluster** est un réseau virtuel routable; les nœuds sont attribués à des sous-réseaux plus petits pour permettre l’utilisation de leurs pods.
  - Le **sous-réseau de service** est un sous-réseau non routable et purement virtuel sur 11.0/16, que les pods utilisent pour accéder uniformément aux services sans se préoccuper de la topologie du réseau. Il est converti vers/depuis l’espace d’adressage routable par `kube-proxy` en cours d’exécution sur les nœuds.


## <a name="what-we-will-accomplish"></a>Les tâches que nous allons accomplir ##
À la fin de ce guide, nous aurons:

> [!div class="checklist"]  
> * Préparé notre [topologie de réseau](#network-topology).  
> * Configuré un nœud master [Linux](#preparing-the-linux-master).  
> * Joint un [nœud de travail Windows](#preparing-a-windows-node) à un nœud master.  
> * Déployé un [exemple de service Windows](#running-a-sample-service).  
> * Abordé les [erreurs et problèmes courants](./common-problems.md).  


## <a name="network-topology"></a>Topologie du réseau ##
Il existe plusieurs façons de rendre le [sous-réseau de cluster](#cluster-subnet-def) virtuel routable. Vous pouvez:

  - Configurer le [mode hôte-passerelle](./configuring-host-gateway-mode.md), en définissant des itinéraires statiques de tronçon suivant entre les nœuds pour permettre la communication de pod à pod.
  - Configurer un commutateur top-of-rack (ToR) intelligent pour router le sous-réseau.
  - Utiliser un plug-in de superposition tiers comme [Flannel](https://coreos.com/flannel/docs/latest/kubernetes.html) (la prise en charge de Windows pour Flannel est en version bêta).


## <a name="preparing-the-linux-master"></a>Préparation du master Linux ##
Que vous ayez suivi [nos instructions](./creating-a-linux-master.md) ou que vous disposiez déjà d’un cluster existant, la seule chose dont le master Linux ait besoin est la configuration de certificat de Kubernetes. Celle-ci peut se trouver dans `/etc/kubernetes/admin.conf`, `~/.kube/config`, ou dans un autre emplacement, en fonction de votre installation.


## <a name="preparing-a-windows-node"></a>Préparation d’un nœud Windows ##
> [!Note]  
> Tous les extraits de code dans les sections Windows doivent être exécutés dans PowerShell avec _élévation_ de privilèges.

Kubernetes utilise [Docker](https://www.docker.com/) comme orchestrateur de conteneur. Nous devons donc l’installer. Vous pouvez suivre les [instructions MSDN officielles](virtualization/windowscontainers/manage-docker/configure-docker-daemon.md#install-docker), les [instructions Docker](https://store.docker.com/editions/enterprise/docker-ee-server-windows), ou essayez les étapes suivantes:

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

Il existe une collection de scripts sur [ce référentiel Microsoft](https://github.com/Microsoft/SDN) qui nous aideront à joindre ce nœud au cluster. Vous pouvez télécharger le fichier ZIP directement [ici](https://github.com/Microsoft/SDN/archive/master.zip). La seule chose dont nous ayons besoin est le dossier `Kubernetes/windows`, dont le contenu doit être déplacé sur `C:\k\`:

```powershell
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mkdir C:/k/
mv master/SDN-master/Kubernetes/windows/* C:/k/
rm -recurse -force master,master.zip
```

Copiez le fichier de certificat [identifié précédemment](#preparing-the-linux-master) dans ce nouveau répertoire `C:\k`.


### <a name="creating-the-pause-image"></a>Création de l’image «Pause» ###
Maintenant que `docker` est installé, nous devons préparer une image «pause» qui est utilisée par Kubernetes pour préparer les pods d’infrastructure.

```powershell
docker pull microsoft/windowsservercore:1709
docker tag microsoft/windowsservercore:1709 microsoft/windowsservercore:latest
cd C:/k/
docker build -t kubeletwin/pause .
```

> [!Note]  
> Nous lui attribuons la balise `:latest`, car l’exemple de service que nous déploierons ultérieurement en dépend, même si en réalité, il peut ne pas _s'agir_ de la dernière image WindowsServerCore disponible. Il est important de prendre garde aux images de conteneuren conflit; ne pas attribuer la balise attendue peut provoquer un `docker pull` d’une image de conteneur incompatible, entraînant ainsi des [problèmes de déploiement](./common-problems.md#when-deploying-docker-containers-keep-restarting). 


### <a name="downloading-binaries"></a>Téléchargement des fichiers binaires ###
Pendant le déroulement du `pull`, téléchargez les fichiers binaires côté client suivants à partir de Kubernetes:

  - `kubectl.exe`
  - `kubelet.exe`
  - `kube-proxy.exe`

Vous pouvez les télécharger depuis les liens se trouvant dans le fichier `CHANGELOG.md` de la dernière version1.9. À ce jour, à savoir [1.9.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1), et les fichiers binaires Windows sont [ici](https://storage.googleapis.com/kubernetes-release/release/v1.9.1/kubernetes-node-windows-amd64.tar.gz). Utilisez un outil comme [7-Zip](http://www.7-zip.org/) pour extraire l’archive et placer les fichiers binaires dans `C:\k\`.


### <a name="joining-the-cluster"></a>Rejoindre le cluster ###
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

Si l’un de ces tests de base ne fonctionne pas, essayez la [page de dépannage](./common-problems.md#network-connectivity) pour résoudre les problèmes courants.


## <a name="running-a-sample-service"></a>Exécution d’un exemple de service ##
Nous allons déployer un [service Web basé sur PowerShell](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) très simple pour nous assurer que nous avons rejoint le cluster avec succès et que notre réseau est correctement configuré.


Sur le master Linux, téléchargez et exécutez le service:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/WebServer.yaml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Cela va créer un déploiement et un service. Regardez ensuite les pods indéfiniment pour effectuer le suivi de leur état; appuyez simplement sur `Ctrl+C` pour quitter la commande `watch` lorsque vous avez terminé l’observation.


Si tout s’est bien passé, vous serez en mesure de valider que les opérations suivantes sont possibles:

  - voir 4conteneurs sous une commande `docker ps` du côté Windows.
  - `curl` sur les adresses IP *pod* sur le port80, le master Linux obtient une réponse du serveur Web; cela montre que la communication de nœud à pod sur le réseau est correcte.
  - `curl` sur l’IP *nœud* sur le port 4444, obtention d’une réponse du serveur Web; cela montre que le mappage de port de hôte à conteneur est correct.
  - effectuer un ping *entre pods* (notamment sur les ordinateurs hôtes, si vous disposez de plusieurs nœuds Windows) via `docker exec`; cela montre que la communication de pod à pod est correcte.
  - `curl` l’adresse *IP de service* virtuelle (visible sous `kubectl get services`) à partir du master Linux et à partir de pods individuels.
  - `curl` le *nom du service* avec le [suffixe DNS par défaut](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services) Kubernetes, illustrant la fonctionnalité DNS.

> [!Warning]  
> Les nœuds de Windows ne seront pas en mesure d’accéder à l’adresse IP de service. Cette [limitation est connue](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip).

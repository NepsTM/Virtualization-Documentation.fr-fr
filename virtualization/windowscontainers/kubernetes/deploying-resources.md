---
title: Jonction de nœuds de Linux
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Déploiement resoureces Kubernetes sur un cluster Kubernetes de systèmes d’exploitation mixtes.
keywords: kubernetes, 1.12, windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 608cda1494d03da59e8a875910c8eedd04ba11dc
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178998"
---
# <a name="deploying-kubernetes-resources"></a>Déploiement des ressources de Kubernetes #
En supposant que vous disposez d’un cluster Kubernetes constituée d’au moins 1 maître et de 1 travail, vous êtes prêt à déployer des ressources de Kubernetes.
> [!TIP] 
> Curieux de savoir quelles ressources Kubernetes sont pris en charge dès aujourd'hui sur Windows? Pour plus d’informations, consultez [officiellement pris en charge des fonctionnalités](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features) et [Kubernetes sur la feuille de route Windows](https://trello.com/b/rjTqrwjl/windows-k8s-roadmap) .


## <a name="running-a-sample-service"></a>Un exemple de service en cours d’exécution ##
Vous allez déployer un [service Web basé sur PowerShell](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) très simple pour vous assurer que vous avez rejoint le cluster avec succès et que notre réseau est correctement configuré.

Avant cela, il est toujours une bonne idée pour vous assurer que tous les nœuds de notre sont sains.
```bash
kubectl get nodes
```

Si tout s’affiche correctement, vous pouvez télécharger et exécuter le service suivant:
> [!Important] 
> Avant `kubectl apply`, assurez-vous de bien sûr à double-check/modifier le `microsoft/windowsservercore` image dans l’exemple de fichier à [une image de conteneur qui est exécutable par vos nœuds](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)!

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Cela crée un déploiement et un service. La dernière commande observer interroge les pods indéfiniment pour effectuer le suivi de leur état; Appuyez simplement sur `Ctrl+C` pour quitter le `watch` commande quand terminé l’observation.

Si tout va bien, il est possible de:

  - voir 2 conteneurs par pod sous `docker ps` commande sur le nœud Windows
  - voir 2pods sous une commande `kubectl get pods` à partir du master Linux
  - `curl` sur les adresses IP *pod* sur le port80, le master Linux obtient une réponse du serveur Web; cela montre que la communication de nœud à pod sur le réseau est correcte.
  - effectuer un ping *entre pods* (notamment sur les ordinateurs hôtes, si vous disposez de plusieurs nœuds Windows) via `docker exec`; cela montre que la communication de pod à pod est correcte.
  - `curl` *adresse IP de service* virtuelle (visible sous `kubectl get services`) à partir du master Linux et à partir de pods individuels; Cela illustre un service correct pour la communication de pod.
  - `curl` le *nom du service* avec le Kubernetes [suffixe DNS par défaut](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services), montrant la découverte de service appropriée.
  - `curl` le *NodePort* à partir du master Linux ou les ordinateurs en dehors du cluster; Cela illustre la connectivité entrante.
  - `curl` IP externes à partir d’à l’intérieur du pod; Cela illustre la connectivité sortante.

> [!Note]  
> Les *hôtes de conteneur* Windows sera **pas** être en mesure d’accéder à l’adresse IP de service à partir des services planifiés sur chacun d’eux. Il s’agit d’une [limitation de plateforme connue](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) qui sera améliorée dans les futures versions de Windows Server. Windows *pods* **sont** en mesure d’accéder à l’adresse IP de service toutefois.

### <a name="port-mapping"></a>Mappage de port ### 
Il est également possible d’accéder aux services hébergés dans les pods au travers de leurs nœuds respectifs en mappant un port sur le nœud. Il existe un [autre exemple YAML disponible](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml) avec le mappage du port4444 sur le nœud au port80 sur le nœud qui montre cette fonctionnalité. Pour le déployer, suivez les mêmes étapes que précédemment:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

Il devrait maintenant être possible de `curl` sur l'adresse IP du *nœud* sur le port4444 et recevoir une réponse du serveur web. N’oubliez pas que cela limite la montée en charge à un seul pod par nœud dans la mesure où cela oblige à un mappage de un-à-un.


## <a name="next-steps"></a>Étapes suivantes ##
Dans cette section, nous avons abordé la planification des ressources de Kubernetes sur les nœuds de Windows. Le guide est terminée. S’il existe des problèmes, veuillez consulter la section Résolution des problèmes:

> [!div class="nextstepaction"]
> [Résolution des problèmes](./common-problems.md)
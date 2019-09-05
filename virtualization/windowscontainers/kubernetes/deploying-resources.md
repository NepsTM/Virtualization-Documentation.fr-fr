---
title: Joindre des nœuds Linux
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Déploiement d’Kubernetes Resoureces sur un cluster Kubernetes de systèmes d’exploitation mixte.
keywords: kubernetes, 1,14, Windows, mise en route
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: e6c569ae8d5bf50e24ea0fc7a6dd04734b60a863
ms.sourcegitcommit: d252f356a3de98f224e1550536810dfc75345303
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/04/2019
ms.locfileid: "10069943"
---
# <a name="deploying-kubernetes-resources"></a>Déploiement de ressources Kubernetes #
En supposant que vous disposez d’un cluster Kubernetes composé d’au moins 1 maître et 1 travailleur, vous pouvez déployer des ressources Kubernetes.
> [!TIP] 
> Vous voulez savoir quelles sont les ressources de Kubernetes prises en charge dans Windows? Pour plus d’informations, reportez-vous à la rubrique [fonctionnalités officiellement prises en charge](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) et [Kubernetes sur le planning Windows](https://github.com/orgs/kubernetes/projects/8) .


## <a name="running-a-sample-service"></a>Exécution d’un exemple de service ##
Vous allez déployer un [service Web basé sur PowerShell](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) très simple pour vous assurer que vous avez rejoint le cluster avec succès et que notre réseau est correctement configuré.

Avant cela, il est toujours judicieux de vérifier que tous les nœuds sont sains.
```bash
kubectl get nodes
```

Si tout fonctionne correctement, vous pouvez télécharger et exécuter le service suivant:
> [!Important] 
> Avant `kubectl apply`tout, assurez-vous de vérifier ou de `microsoft/windowsservercore` modifier l’image dans l’exemple de fichier en [une image de conteneur exécutable par vos nœuds](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)!

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Cela crée un déploiement et un service. La dernière commande espion interroge indéfiniment les gousses pour suivre leur statut; Il suffit `Ctrl+C` d’appuyer sur `watch` pour quitter la commande lorsque vous avez terminé l’observation.

Si tout va bien, il est possible de:

  - Voir 2 conteneurs par Pod sous `docker ps` commande du nœud Windows
  - voir 2pods sous une commande `kubectl get pods` à partir du master Linux
  - `curl` sur les adresses IP *pod* sur le port80, le master Linux obtient une réponse du serveur Web; cela montre que la communication de nœud à pod sur le réseau est correcte.
  - effectuer un ping *entre pods* (notamment sur les ordinateurs hôtes, si vous disposez de plusieurs nœuds Windows) via `docker exec`; cela montre que la communication de pod à pod est correcte.
  - `curl` *adresse IP du service* virtuel (voir `kubectl get services`sous) à partir du maître Linux et de la Pod individuelle; Cela démontre un service approprié aux communications par pod.
  - `curl` *nom du service* avec le [suffixe DNS par défaut](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)Kubernetes, montrant une découverte correcte du service.
  - `curl` le ** dépassement du maître ou des machines Linux hors du cluster; Cela illustre la connectivité entrante.
  - `curl` IPs externes à l’intérieur de la Pod; Cela démontre la connectivité sortante.

> [!Note]  
> Les *hôtes de conteneur* Windows ne seront **pas** en mesure d’accéder à l’adresse IP du service via les services programmés. Il s’agit d’une [limitation de plateforme connue](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) qui sera améliorée dans les futures versions de Windows Server. Les *gousses* Windows **sont** néanmoins en mesure d’accéder à l’adresse IP du service.

## <a name="next-steps"></a>Étapes suivantes ##
Dans cette section, nous avons expliqué comment planifier des ressources Kubernetes sur des nœuds Windows. Cela conclut le guide. En cas de problème, consultez la section résolution des problèmes:

> [!div class="nextstepaction"]
> [Résolution des problèmes](./common-problems.md)

Dans le cas contraire, vous pouvez également être intéressé par l’exécution des composants Kubernetes en tant que services Windows:
> [!div class="nextstepaction"]
> [Services Windows](./kube-windows-services.md)

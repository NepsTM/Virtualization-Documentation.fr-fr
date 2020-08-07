---
title: Jonction de nœuds Linux
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: how-to
description: Déploiement de Kubernetes Resoureces sur un cluster Kubernetes de système d’exploitation mixte.
keywords: kubernetes, 1,14, Windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: a342a03153564b2f76af45a9b792b58ae8afc18c
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985333"
---
# <a name="deploying-kubernetes-resources"></a>Déploiement des ressources Kubernetes #
En supposant que vous avez un cluster Kubernetes constitué d’au moins 1 maître et 1 Worker, vous êtes prêt à déployer des ressources Kubernetes.
> [!TIP]
> Vous êtes curieux de savoir quelles sont les ressources Kubernetes actuellement prises en charge sur Windows ? Pour plus d’informations, consultez [fonctionnalités officiellement prises en charge](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) et Kubernetes sur la feuille de [route Windows](https://github.com/orgs/kubernetes/projects/8) .


## <a name="running-a-sample-service"></a>Exécution d’un exemple de service ##
Vous allez déployer un [service Web basé sur PowerShell](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) très simple pour vous assurer que vous avez rejoint le cluster avec succès et que notre réseau est correctement configuré.

Avant cela, il est toujours judicieux de s’assurer que tous les nœuds sont intègres.
```bash
kubectl get nodes
```

Si tout semble correct, vous pouvez télécharger et exécuter le service suivant :
> [!Important]
> Avant `kubectl apply` , veillez à double-vérifier/modifier l' `microsoft/windowsservercore` image de l’exemple de fichier sur [une image conteneur exécutable par vos nœuds](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions).

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Cela crée un déploiement et un service. La dernière commande Watch interroge indéfiniment les Pod pour suivre leur état. Appuyez simplement sur `Ctrl+C` pour quitter la `watch` commande lorsque vous avez terminé.

Si tout va bien, il est possible de :

  - Voir 2 conteneurs par Pod sous `docker ps` la commande sur le nœud Windows
  - voir 2 pods sous une commande `kubectl get pods` à partir du master Linux
  - `curl`sur les adresses IP *Pod* sur le port 80 de la base de référence Linux, obtient une réponse du serveur Web. Cela démontre la communication entre le nœud et la pod sur le réseau.
  - effectuer un ping *entre pods* (notamment sur les ordinateurs hôtes, si vous disposez de plusieurs nœuds Windows) via `docker exec` ; cela montre que la communication de pod à pod est correcte.
  - `curl`l' *adresse IP du service* virtuel (voir sous `kubectl get services` ) à partir de la base de référence Linux et des gousses individuelles ; cela démontre la communication entre le service et la pod.
  - `curl`*nom du service* avec le [suffixe DNS par défaut](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)Kubernetes, qui illustre la détection de service appropriée.
  - `curl`le *deexclusion* du maître Linux ou des machines situées en dehors du cluster ; Cela démontre la connectivité entrante.
  - `curl`Adresses IP externes à l’intérieur du Pod ; Cela démontre une connectivité sortante.

> [!Note]
> Les *hôtes de conteneur* Windows ne seront **pas** en mesure d’accéder à l’adresse IP du service à partir des services planifiés sur ces derniers. Il s’agit d’une [limitation de plateforme connue](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) qui sera améliorée dans les futures versions de Windows Server. Toutefois, les *Pod* Windows **sont** en mesure d’accéder à l’adresse IP du service.

## <a name="next-steps"></a>Étapes suivantes ##
Dans cette section, nous avons abordé la planification des ressources Kubernetes sur les nœuds Windows. Cela conclut le guide. En cas de problème, consultez la section résolution des problèmes :

> [!div class="nextstepaction"]
> [Dépannage](./common-problems.md)

Dans le cas contraire, vous pouvez également être intéressé par l’exécution des composants Kubernetes en tant que services Windows :
> [!div class="nextstepaction"]
> [services Windows](./kube-windows-services.md)

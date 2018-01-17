---
title: "Résolution des problèmes Kubernetes"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: troubleshooting
ms.prod: containers
description: "Solutions aux problèmes courants lors du déploiement de Kubernetes et de la jonction de nœuds Windows."
keywords: kubernetes, 1.9, linux, compiler
ms.openlocfilehash: 73b44ffd12fba58ac4ef38352c012061a6817945
ms.sourcegitcommit: ad5f6344230c7c4977adf3769fb7b01a5eca7bb9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2017
---
# <a name="troubleshooting-kubernetes"></a>Résolution des problèmes Kubernetes #
Cette page décrit plusieurs problèmes courants lors du déploiement, de la mise en réseau et de la configuration de Kubernetes.

> [!tip]
> Proposez une entrée de FAQ en émettant une réservation permanente auprès de [notre référentiel de documentation ](https://github.com/MicrosoftDocs/Virtualization-Documentation/).


## <a name="common-deployment-errors"></a>Problèmes de déploiement courants ##
Le débogage du master Kubernetes s’articule autour de troiscatégories principales (par ordre de probabilité):

  - Quelque chose ne va pas avec les conteneurs système Kubernetes.
  - Quelque chose ne va pas la manière dont `kubelet` s’exécute.
  - Quelque chose ne va pas au niveau du système.


Exécutez `kubectl get pods -n kube-system` pour voir les pods créés par Kubernetes. Cela peut donner des indications concernant les pods défaillants ou ne démarrant pas correctement. Ensuite, exécutez `docker ps -a` pour voir tous les conteneurs qui sauvegardent ces pods. Enfin, pour afficher les résultats bruts des processus, exécutez `docker logs [ID]` sur les conteneurs qui sont suspectés de provoquer le problème.


### <a name="permission-denied-errors"></a>Erreurs _"Autorisation refusée"_ ###
Assurez-vous que les scripts disposent des autorisations de s’exécuter:

```bash
chmod +x [script name]
```

En outre, certains scripts doivent être exécutés avec des privilèges d’administrateur (comme `kubelet`) et doivent être précédés de `sudo`.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>Impossible de se connecter au serveur API à l’adresse `https://[address]:[port]`. ###
Le plus souvent, cette erreur indique des problèmes de certificat. Assurez-vous que vous avez généré correctement le fichier de configuration, que les adresses IP qu’il contient correspondent à celles de votre hôte et que vous l’avez copié dans le répertoire qui est monté par le serveur API.

Si vous avez suivi [nos instructions](./creating-a-linux-master), il s’agit du répertoire `~/kube/kubelet/`. Sinon, reportez-vous au fichier manifeste du serveur API pour vérifier les points de montage.


## <a name="common-networking-errors"></a>Erreurs réseau courantes ##
Il peut exister des restrictions supplémentaires sur votre réseau ou sur des ordinateurs hôtes empêchant certains types de communication entre les nœuds. Vérifiez que:

  - le trafic qui semble émaner des pods est autorisé
  - le trafic HTTP est autorisé, si vous déployez des services web
  - les paquets ICMP ne sont pas supprimés


<!-- ### My Linux node cannot ping my Windows pods ### -->

## <a name="common-windows-errors"></a>Erreurs Windows courantes ##


### <a name="my-windows-pods-cannot-access-the-linux-master-or-vice-versa"></a>Mes pods Windows ne peuvent pas accéder au master Linux, ou vice versa. ###
Si vous utilisez un ordinateur virtuel Hyper-V, assurez-vous que l’usurpation d’adresses MAC est activée sur le ou les adaptateurs réseau.


### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Mon nœud Windows ne peut pas accéder à mes services à l’aide de l’adresse IP de service. ###
Il s’agit d’une limitation connue de la pile de mise en réseau actuelle sur Windows.

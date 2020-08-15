---
title: Kubernetes sur Windows
author: daschott
ms.author: daschott
ms.date: 08/13/2020
ms.topic: overview
description: Prise en main de Kubernetes sur Windows.
keywords: kubernetes, Windows, prise en main
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 0e5e468d3c0092752e13f0dbbb4d9c87bd0328e9
ms.sourcegitcommit: aa139e6e77a27b8afef903fee5c7ef338e1c79d4
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/15/2020
ms.locfileid: "88251542"
---
# <a name="kubernetes-on-windows"></a>Kubernetes sur Windows

> [!TIP]
> Vous voulez savoir quelles fonctionnalités Kubernetes sont prises en charge sur Windows aujourd’hui ? Pour plus d’informations, consultez [fonctionnalités officiellement prises en charge](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) et la feuille [de route Kubernetes sur Windows](https://github.com/orgs/kubernetes/projects/8) .

Cette page fournit une vue d’ensemble de la prise en main de Kubernetes sur Windows.


### <a name="try-out-kubernetes-on-windows"></a>Essayer Kubernetes sur Windows

Consultez [déploiement de Kubernetes sur Windows](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/adding-windows-nodes/) pour obtenir des instructions sur l’installation manuelle de Kubernetes sur Windows dans l’environnement de votre choix.


### <a name="scheduling-windows-containers"></a>Planification des conteneurs Windows

Consultez la page [planification des conteneurs Windows dans Kubernetes](https://kubernetes.io/docs/setup/production-environment/windows/user-guide-windows-containers/) pour obtenir les meilleures pratiques et recommandations sur la planification des conteneurs Windows dans Kubernetes.


### <a name="deploying-kubernetes-on-windows-in-azure"></a>Déploiement de Kubernetes sur Windows dans Azure

Le guide [des conteneurs Windows sur Azure Kubernetes service](/azure/aks/windows-container-cli) facilite cette tâche. Si vous envisagez de déployer et de gérer vous-même tous les composants Kubernetes, consultez notre [procédure pas](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md) à pas à l’aide de l’outil open source `AKS-Engine` .

### <a name="troubleshooting"></a>Résolution des problèmes
Pour obtenir une liste de solutions de contournement et des solutions aux problèmes connus, consultez [Troubleshooting Kubernetes](./common-problems.md) .
>[!TIP]
> Pour obtenir des ressources d’auto-assistance supplémentaires, un guide de dépannage réseau Kubernetes pour Windows est également [disponible ici](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648).
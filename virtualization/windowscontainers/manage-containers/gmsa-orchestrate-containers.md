---
title: Orchestrer des conteneurs avec un gMSA
description: Comment orchestrer des conteneurs Windows avec un compte de service administré de groupe (gMSA).
keywords: docker, conteneurs, Active Directory, GMSA, orchestration, kubernetes, compte de service administré de groupe, comptes de service administrés de groupe
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 3d102aac45a1becf1879a718bb255d753b215006
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910259"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>Orchestrer des conteneurs avec un gMSA

Dans les environnements de production, vous utiliserez souvent un Orchestrator de conteneur pour déployer et gérer vos applications et services. Chaque orchestrateur possède ses propres paradigmes de gestion et est chargé d’accepter les spécifications d’informations d’identification à fournir à la plateforme de conteneur Windows.

Lorsque vous orchestrez des conteneurs avec des comptes de service administrés de groupe (service administrés), assurez-vous que :

> [!div class="checklist"]
> * Tous les hôtes de conteneur qui peuvent être planifiés pour exécuter des conteneurs avec service administrés sont joints à un domaine
> * Les hôtes de conteneur ont accès pour récupérer les mots de passe de tous les service administrés utilisés par les conteneurs
> * Les fichiers de spécification d’informations d’identification sont créés et téléchargés vers l’orchestrateur ou copiés vers chaque hôte de conteneur, en fonction de la façon dont l’orchestrateur préfère les gérer.
> * Les réseaux de conteneurs permettent aux conteneurs de communiquer avec les contrôleurs de domaine Active Directory pour récupérer les tickets gMSA

## <a name="how-to-use-gmsa-with-service-fabric"></a>Utilisation de gMSA avec Service Fabric

Service Fabric prend en charge l’exécution de conteneurs Windows avec un gMSA quand vous spécifiez l’emplacement des spécifications des informations d’identification dans votre manifeste d’application. Vous devez créer le fichier de spécification des informations d’identification et le placer dans le sous-répertoire **CredentialSpecs** du répertoire des données de l’ancrage sur chaque hôte afin que service Fabric puisse le localiser. Vous pouvez exécuter l’applet de commande **CredentialSpec** , qui fait partie du [module PowerShell CredentialSpec](https://aka.ms/credspec), pour vérifier si les spécifications de vos informations d’identification se trouvent à l’emplacement approprié.

Pour plus d’informations sur la configuration de votre application, consultez [démarrage rapide : déployer des conteneurs Windows pour service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) et configurer [gMSA pour les conteneurs Windows s’exécutant sur service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) .

## <a name="how-to-use-gmsa-with-docker-swarm"></a>Guide pratique pour utiliser gMSA avec Dockr essaim

Pour utiliser un gMSA avec des conteneurs gérés par le Dockr essaim, exécutez la commande [dockr service Create](https://docs.docker.com/engine/reference/commandline/service_create/) avec le paramètre `--credential-spec` :

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Pour plus d’informations sur l’utilisation des spécifications d’informations d’identification avec les services d’ancrage, consultez l' [exemple d’ancrage](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) de l’amarrage.

## <a name="how-to-use-gmsa-with-kubernetes"></a>Utilisation de gMSA avec Kubernetes

La prise en charge de la planification des conteneurs Windows avec service administrés dans Kubernetes est disponible en tant que fonctionnalité alpha dans Kubernetes 1,14. Pour obtenir les informations les plus récentes sur cette fonctionnalité et pour savoir comment la tester dans votre distribution Kubernetes, consultez [configurer gMSA pour](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) les boîtiers et les conteneurs Windows.

## <a name="next-steps"></a>Étapes suivantes

Outre l’orchestration des conteneurs, vous pouvez également utiliser service administrés pour :

- [Configurer des applications](gmsa-configure-app.md)
- [Exécuter des conteneurs](gmsa-run-container.md)

Si vous rencontrez des problèmes lors de l’installation, consultez notre [Guide de résolution](gmsa-troubleshooting.md) des problèmes pour obtenir des solutions possibles.

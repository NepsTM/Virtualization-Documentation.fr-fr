---
title: Orchestrer des conteneurs avec un gMSA
description: Comment orchestrer des conteneurs Windows avec un compte de service géré par groupe (gMSA).
keywords: dockeur, conteneurs, Active Directory, GMSA, orchestration, kubernetes, compte de service géré par groupe, comptes de service géré par groupe
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 3d102aac45a1becf1879a718bb255d753b215006
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209839"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>Orchestrer des conteneurs avec un gMSA

Dans les environnements de production, vous utilisez souvent un conteneur Orchestrator pour déployer et gérer vos applications et services. Chaque Orchestrator dispose de ses propres paradigmes de gestion et est tenu d’accepter les spécifications d’informations d’identification à transmettre à la plateforme de conteneur Windows.

Lorsque vous orchestrez des conteneurs avec des comptes de service géré par groupe (gMSAs), assurez-vous que:

> [!div class="checklist"]
> * Tous les hôtes de conteneur qui peuvent être planifiés pour exécuter des conteneurs avec gMSAs sont joints au domaine
> * Les hôtes de conteneur ont accès pour récupérer les mots de passe de tous les gMSAs utilisés par les conteneurs.
> * Les fichiers de spécification d’informations d’identification sont créés et téléchargés vers l’Orchestrator ou copiés sur chaque hôte de conteneur, en fonction de la façon dont l’Orchestrator préfère les gérer.
> * Les réseaux de conteneurs permettent aux conteneurs de communiquer avec les contrôleurs de domaine Active Directory pour récupérer les tickets gMSA

## <a name="how-to-use-gmsa-with-service-fabric"></a>Utiliser gMSA avec service Fabric

Service Fabric prend en charge l’exécution de conteneurs Windows avec un gMSA lorsque vous spécifiez l’emplacement des spécifications d’informations d’identification dans le manifeste de votre application. Vous devez créer le fichier de spécifications d’information d’identification et le placer dans le sous-répertoire **CredentialSpecs** du répertoire de données de l’ancrage sur chaque hôte de manière à ce que le service Fabric puisse le Rechercher. Vous pouvez exécuter l’applet de connexion **Get-CredentialSpec** , qui fait partie du [module CredentialSpec PowerShell](https://aka.ms/credspec), pour vérifier si les spécifications d’informations d’identification se trouvent à l’emplacement approprié.

Pour plus d’informations sur la configuration de votre application, voir [démarrage rapide: déploiement de conteneurs Windows sur le service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) et configuration [de gMSA pour les conteneurs Windows en cours d’exécution sur le service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) .

## <a name="how-to-use-gmsa-with-docker-swarm"></a>Utiliser gMSA avec l’essaimeur d’amarrage

Pour utiliser une gMSA avec des conteneurs gérés par essaim d’amarrage, exécutez la commande de création du service `--credential-spec` d' [ancrage](https://docs.docker.com/engine/reference/commandline/service_create/) avec le paramètre:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Pour plus d’informations sur l’utilisation des spécifications d’information d’identification avec les services d’amarrage, voir l' [exemple d’ancrage de dock](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) .

## <a name="how-to-use-gmsa-with-kubernetes"></a>Utiliser gMSA avec Kubernetes

La prise en charge de la planification de conteneurs Windows avec gMSAs dans Kubernetes est disponible sous la forme d’une fonctionnalité alpha dans Kubernetes 1,14. Pour plus d’informations sur cette fonctionnalité, voir [configurer gMSA pour les gousses et conteneurs Windows](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) et savoir comment la tester dans votre distribution Kubernetes.

## <a name="next-steps"></a>Étapes suivantes

Outre l’orchestration des conteneurs, vous pouvez également utiliser gMSAs pour:

- [Configurer des applications](gmsa-configure-app.md)
- [Conteneurs d’exécution](gmsa-run-container.md)

Si vous rencontrez des problèmes lors de l’installation, consultez notre [Guide de résolution des problèmes](gmsa-troubleshooting.md) pour consulter les solutions possibles.

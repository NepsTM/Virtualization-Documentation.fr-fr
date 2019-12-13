---
title: Configurer votre application pour utiliser un compte de service administré de groupe
description: Comment configurer des applications pour utiliser des comptes de service administrés de groupe (service administrés) pour les conteneurs Windows.
keywords: ancrage, conteneurs, Active Directory, GMSA, applications, applications, compte de service administré de groupe, comptes de service administrés de groupe, configuration
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 6635381d5f7ddbebf7bdea4624af241b9f6a6864
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909789"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>Configurer votre application pour utiliser un gMSA

Dans la configuration standard, un conteneur ne reçoit qu’un seul compte de service administré de groupe (gMSA) utilisé chaque fois que le compte d’ordinateur conteneur tente de s’authentifier auprès des ressources réseau. Cela signifie que votre application doit s’exécuter en tant que **système local** ou **service réseau** si elle doit utiliser l’identité gMSA.

## <a name="run-an-iis-app-pool-as-network-service"></a>Exécuter un pool d’applications IIS en tant que service réseau

Si vous hébergez un site Web IIS dans votre conteneur, tout ce que vous devez faire pour tirer parti du gMSA est de définir l’identité du pool d’applications sur **service réseau**. Vous pouvez le faire dans votre fichier dockerfile en ajoutant la commande suivante :

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

Si vous avez précédemment utilisé les informations d’identification de l’utilisateur statique pour votre pool d’applications IIS, envisagez le gMSA comme remplacement de ces informations d’identification. Vous pouvez modifier les gMSA entre les environnements de développement, de test et de production, et IIS récupère automatiquement l’identité actuelle sans avoir à modifier l’image de conteneur.

## <a name="run-a-windows-service-as-network-service"></a>Exécuter un service Windows en tant que service réseau

Si votre application en conteneur s’exécute en tant que service Windows, vous pouvez définir le service pour qu’il s’exécute en tant que service **réseau** dans votre fichier dockerfile :

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>Exécuter des applications console arbitraires en tant que service réseau

Pour les applications console génériques qui ne sont pas hébergées dans IIS ou Service Manager, il est souvent plus facile d’exécuter le conteneur en tant que **service réseau** pour que l’application hérite automatiquement du contexte gMSA. Cette fonctionnalité est disponible à partir de la version 1709 de Windows Server.

Ajoutez la ligne suivante à votre fichier dockerfile pour qu’elle s’exécute en tant que service réseau par défaut :

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

Vous pouvez également vous connecter à un conteneur en tant que service réseau sur une base unique avec `docker exec`. Cela s’avère particulièrement utile si vous résolvez des problèmes de connectivité dans un conteneur en cours d’exécution lorsque le conteneur n’est généralement pas exécuté en tant que service réseau.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>Étapes suivantes

Outre la configuration des applications, vous pouvez également utiliser service administrés pour :

- [Exécuter des conteneurs](gmsa-run-container.md)
- [Orchestrer des conteneurs](gmsa-orchestrate-containers.md)

Si vous rencontrez des problèmes lors de l’installation, consultez notre [Guide de résolution](gmsa-troubleshooting.md) des problèmes pour obtenir des solutions possibles.

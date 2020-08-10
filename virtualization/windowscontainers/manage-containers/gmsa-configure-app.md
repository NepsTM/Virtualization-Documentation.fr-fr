---
title: Configurer votre application pour utiliser un compte de service administré de groupe
description: Comment configurer des applications pour utiliser des comptes de service administré de groupe pour des conteneurs Windows.
keywords: docker, conteneurs, active directory, compte de service administré de groupe, applis, applications, comptes de service administrés de groupe, configuration
author: rpsqrd
ms.date: 09/10/2019
ms.topic: how-to
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: fa14f2fa953373f8dfea74f822b025be0910fd54
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985263"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>Configurer votre application pour utiliser un compte de service administré de groupe

Dans la configuration classique, un conteneur ne reçoit qu’un seul compte de service administré de groupe, qui est utilisé chaque fois que le compte d’ordinateur conteneur tente de s’authentifier auprès des ressources réseau. Cela signifie que votre application doit s’exécuter en tant que **Système local** ou **Service réseau** si elle doit utiliser l’identité de compte de service administré de groupe.

## <a name="run-an-iis-app-pool-as-network-service"></a>Exécuter un pool d’applications IIS en tant que service réseau

Si vous hébergez un site web IIS dans votre conteneur, vous n’avez qu’à utiliser le compte de service administré de groupe pour définir l’identité de votre pool d’applications sur **Service réseau**. Vous pouvez le faire dans votre fichier Dockerfile en ajoutant la commande suivante :

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

Si vous avez précédemment utilisé les informations d’identification de l’utilisateur statiques pour votre pool d’applications IIS, envisagez d’utiliser le compte de service administré de groupe à la place de ces informations d’identification. Vous pouvez modifier le compte de service administré de groupe entre les environnements de développement, de test et de production, et IIS récupère automatiquement l’identité actuelle sans avoir à modifier l’image du conteneur.

## <a name="run-a-windows-service-as-network-service"></a>Exécuter un service Windows en tant que Service réseau

Si votre application en conteneur s’exécute en tant que service Windows, vous pouvez définir le service pour qu’il s’exécute en tant que **Service réseau** dans votre fichier Dockerfile :

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>Exécuter des applications de console arbitraires en tant que service réseau

Pour des applications de console génériques qui ne sont pas hébergées dans IIS ou Service Manager, il est souvent plus facile d’exécuter le conteneur en tant que **Service réseau** de sorte que l’application hérite automatiquement du contexte du compte de service administré de groupe. Cette fonctionnalité est disponible à partir de la version 1709 de Windows Server.

Ajoutez la ligne suivante à votre fichier Dockerfile pour qu’elle s’exécute en tant que Service réseau par défaut :

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

Vous pouvez également établir une connexion à un conteneur en tant que Service réseau de façon ponctuelle avec `docker exec`. C’est particulièrement utile si vous résolvez des problèmes de connectivité dans un conteneur en cours d’exécution lorsque celui-ci ne s’exécute généralement pas en tant que Service réseau.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>Étapes suivantes

Outre la configuration des applications, vous pouvez utiliser des comptes de service administré de groupe pour effectuer les opérations suivantes :

- [Exécuter des conteneurs](gmsa-run-container.md)
- [Orchestrer des conteneurs](gmsa-orchestrate-containers.md)

Si vous rencontrez des problèmes lors de la configuration, vous trouverez peut-être des solutions dans notre [Guide de résolution des problèmes](gmsa-troubleshooting.md).

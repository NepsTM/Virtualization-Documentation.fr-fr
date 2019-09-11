---
title: Configurer votre application pour qu’elle utilise un compte de service géré par groupe
description: Comment configurer des applications de manière à utiliser les comptes de service géré de groupe (gMSAs) pour les conteneurs Windows.
keywords: dockeur, conteneurs, Active Directory, GMSA, applications, applications, compte de service géré par groupe, comptes de service géré par groupe, configuration
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 234909d7f0cb0f30ee7fbf4796dd0381bfbff89f
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079724"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>Configurer votre application pour utiliser une gMSA

Dans la configuration par défaut, un conteneur n’est fourni qu’à un seul compte de service géré de groupe (gMSA), qui est utilisé chaque fois que le compte d’ordinateur conteneur tente d’s’authentifier aux ressources réseau. Cela signifie que votre application doit s’exécuter en tant que service **système local** ou **service réseau** si elle doit utiliser l’identité gMSA.

## <a name="run-an-iis-app-pool-as-network-service"></a>Exécuter un pool d’applications IIS en tant que service réseau

Si vous hébergez un site Web IIS dans votre conteneur, il vous suffit de définir l’identité de votre pool d’applications sur **service réseau**pour pouvoir utiliser le gMSA. Pour cela, vous pouvez ajouter la commande suivante dans votre Dockerfile:

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

Si vous utilisiez précédemment des informations d’identification d’utilisateur statique pour votre pool d’applications IIS, considérez le gMSA comme remplacement de ces informations d’identification. Vous pouvez modifier l’gMSA entre environnements de développement, de test et de production, et les services Internet (IIS) sélectionnent automatiquement l’identité actuelle sans avoir à modifier l’image du conteneur.

## <a name="run-a-windows-service-as-network-service"></a>Exécuter un service Windows en tant que service réseau

Si votre application conteneur s’exécute en tant que service Windows, vous pouvez définir le service pour qu’il s’exécute en tant que service **réseau** dans votre Dockerfile:

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>Exécuter des applications consoles arbitraires en tant que service réseau

Pour les applications de console génériques qui ne sont pas hébergées dans les services Internet (IIS) et le Service Manager, il est souvent plus facile d’exécuter le conteneur en tant que **service réseau** , ce qui permet à l’application d’hériter automatiquement du contexte gMSA. Cette fonctionnalité est disponible sous Windows Server version 1709.

Ajoutez la ligne suivante à votre Dockerfile pour qu’elle s’exécute en tant que service réseau par défaut:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

Vous pouvez également vous connecter à un conteneur en tant que service réseau en fonction d’un `docker exec`seul arrêt. Cela est particulièrement utile si vous rencontrez des problèmes de connectivité dans un conteneur en cours d’exécution lorsque le conteneur ne s’exécute pas normalement en tant que service réseau.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>Étapes suivantes

En plus de configurer des applications, vous pouvez également utiliser gMSAs pour:

- [Conteneurs d’exécution](gmsa-run-container.md)
- [Orchestrer des conteneurs](gmsa-orchestrate-containers.md)

Si vous rencontrez des problèmes lors de l’installation, consultez notre [Guide de résolution des problèmes](gmsa-troubleshooting.md) pour consulter les solutions possibles.

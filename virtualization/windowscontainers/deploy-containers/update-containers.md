---
title: Mettre à jour des conteneurs Windows Server
description: Comment Windows peut générer et exécuter des conteneurs dans plusieurs versions
keywords: métadonnées, conteneurs, version
author: heidilohr
ms. author: helohr
manager: lizross
ms.date: 03/10/2020
ms.openlocfilehash: 84413f27bfce66e7d259c05795a280ed34b582ab
ms.sourcegitcommit: 6f216408434a437da87a72d582500a4ca6c2679c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/21/2020
ms.locfileid: "80112692"
---
# <a name="update-windows-server-containers"></a>Mettre à jour des conteneurs Windows Server

Dans le cadre de la maintenance mensuelle de Windows Server, nous publions régulièrement des images mises à jour du conteneur du système d’exploitation de base Windows Server. Ces mises à jour vous permettent d’automatiser la construction d’images de conteneur mises à jour ou de les mettre à jour manuellement en extrayant la dernière version. Les conteneurs Windows Server n’ont pas de pile de maintenance comme Windows Server. Vous ne pouvez pas obtenir de mises à jour à l’intérieur d’un conteneur comme vous le feriez avec Windows Server. Par conséquent, chaque mois, nous reconstruisons les images du conteneur du système d’exploitation de base Windows Server avec les mises à jour, et publions les images mises à jour.

D’autres images de conteneur, telles que .NET ou IIS, seront reconstruites à partir des images du conteneur du système d’exploitation de base mises à jour, et publiées chaque mois.

## <a name="how-to-get-windows-server-container-updates"></a>Comment obtenir des mises à jour du conteneur Windows Server

Nous mettons à jour les images du conteneur du système d’exploitation de base Windows Server en nous alignant sur la cadence de maintenance de Windows. Les images du conteneur mises à jour sont publiées le deuxième mardi de chaque mois. Nous y faisons parfois référence sous le nom de publication « B », assorti d’un numéro de préfixe basé sur le mois de sortie. Par exemple, nous appelons la mise à jour de février « 2B », et celle de mars « 3B ». Cette mise à jour mensuelle est la seule publication standard incluant de nouveaux correctifs de sécurité.

Le serveur hébergeant ces conteneurs, appelé hôte de conteneurs ou simplement « hôte », peut faire l’objet d’une maintenance lors d’événements de mise à jour supplémentaires autres que les publications « B ». Pour en savoir plus à ce sujet, consultez notre billet de blog sur la [cadence de maintenance de mise à jour de Windows](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/windows-10-update-servicing-cadence/ba-p/222376).

Les nouvelles images du conteneur du système d’exploitation de base Windows Server sont publiées peu après 10 heures PST, le deuxième mardi de chaque mois, dans le Registre de conteneurs Microsoft, et les balises recommandées ciblent la version B la plus récente. Voici quelques exemples :

- ltsc2019 [(LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc):  docker pull mcr.microsoft.com/windows/servercore:ltsc2019
- 1909 [(SAC)](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel): docker pull mcr.microsoft.com/windows/servercore:1909

Si vous connaissez mieux le Docker Hub que le Registre de conteneurs Microsoft, [ce billet de blog](https://azure.microsoft.com/blog/microsoft-syndicates-container-catalog/) vous fournit des explications plus détaillées.  

Pour chaque publication, l’image du conteneur concernée est également publiée avec deux balises supplémentaires, respectivement pour le numéro de révision et pour le numéro d’article de la base de connaissances, afin de cibler des révisions d’image du conteneur spécifiques. Par exemple :

- docker pull mcr.microsoft.com/windows/servercore:10.0.17763.1040
- docker pull mcr.microsoft.com/windows/servercore:1809-KB4546852

Ces exemples extraient tous deux l’image du conteneur Server Core de Windows Server 2019 avec la mise à jour de sécurité du 18 février.  

Pour obtenir la liste complète des images du conteneur du système d’exploitation de base Windows Server, avec leurs versions et balises respectives, consultez la page [Images du conteneur du système d’exploitation de base Windows](https://hub.docker.com/_/microsoft-windows-base-os-images) sur le Docker Hub.

Les images de Windows Server ayant fait l’objet de la maintenance mensuelle qui sont publiées par Microsoft sur la Place de marché Azure contiennent également des images préinstallées du conteneur du système d’exploitation de base. Pour plus d’informations, consultez notre [page concernant la tarification de Windows Server sur la Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=PlansAndPrice). Généralement, nous mettons à jour ces images quelque cinq jours ouvrés après la publication « B ».

Pour obtenir la liste complète des images et versions de Windows Server, consultez l’[historique de mise à jour de Windows Server sur la Place de marché Azure](https://support.microsoft.com/help/4497947/windows-server-release-on-azure-marketplace-update-history).

## <a name="host-and-container-version-compatibility"></a>Compatibilité des versions de l’hôte et du conteneur

Il existe deux types de modes d’isolation pour les conteneurs Windows : Isolation des processus et isolation Hyper-V. L’isolation Hyper-V est plus flexible lorsqu’il s’agit de la compatibilité des versions de l’hôte et du conteneur. Pour en savoir plus, consultez [Compatibilité des versions](version-compatibility.md) et [Modes d’isolation](../manage-containers/hyperv-container.md). Sauf indication contraire, cette section se concentre sur les conteneurs isolés par processus.

Lorsque vous mettez à jour l’hôte ou l’image de votre conteneur avec les mises à jour mensuelles, tant que l’hôte et l’image du conteneur sont tous deux pris en charge (Windows Server, version 1809 ou ultérieure), leurs révisions n’ont pas besoin de correspondre pour que le conteneur démarre et s’exécute normalement.

En revanche, vous risquez de rencontrer des problèmes lors de l’utilisation de conteneurs Windows Server avec la version de mise à jour de sécurité du 11 février 2020 (également appelée « 2B ») ou des mises à jour de sécurité mensuelles ultérieures. Pour plus de détails, consultez cet [article du Support Microsoft](https://support.microsoft.com/help/4542617/you-might-encounter-issues-when-using-windows-server-containers-with-t). Ces problèmes résultent d’une modification de sécurité qui nécessite un changement de l’interface entre le mode utilisateur et le mode noyau pour garantir la sécurité de vos applications. Ils se produisent uniquement sur des conteneurs isolés du processus, parce que ceux-ci partagent le mode noyau avec l’hôte du conteneur. Cela signifie que les images du conteneur sans le composant mode utilisateur mis à jour ne sont ni sécurisées, ni compatibles avec la nouvelle interface de noyau sécurisée.

Nous avons donc publié un correctif le 18 février 2020. Cette nouvelle publication a établi une « nouvelle ligne de base ». Cette nouvelle ligne de base suit les règles suivantes :

- Toutes les combinaisons d’hôtes et de conteneurs antérieurs à la publication 2B fonctionnent.
- Toutes les combinaisons d’hôtes et de conteneurs postérieurs à la publication 2B fonctionnent.
- Aucune combinaison d’hôtes et de conteneurs qui ne sont pas soit antérieurs, soit postérieurs à la nouvelle ligne de base ne fonctionne. Par exemple, la combinaison d’un hôte 3B et d’un conteneur 1B ne fonctionne pas.

Nous allons utiliser la publication de la mise à jour de sécurité mensuelle de mars 2020 comme exemple pour vous montrer comment fonctionnent ces nouvelles règles de compatibilité. Dans le tableau suivant, la publication de la mise à jour de sécurité de mars 2020 est appelée « 3B », celle de février 2020 « 2B », et celle de janvier 2020 « 1B ».

| Host | Conteneur | Compatibilité |
|---|---|---|
| 3B | 3B | Oui |
| 3B | 2B | Oui |
| 3B | 1B ou antérieure | Non |
| 2B | 3B | Oui |
| 2B | 2B | Oui |
| 2B | 1B ou antérieure | Non |
| 1B ou antérieure | 3B | Non |
| 1B ou antérieure | 2B | Non |
| 1B ou antérieure | 1B ou antérieure | Oui |

Pour référence, le tableau suivant répertorie les numéros de version des images du conteneur du système d’exploitation de base avec les publications des mises à jour de sécurité mensuelles 1B, 2B et 3B sur les différentes publications majeures du système d’exploitation, de Windows Server 2016 à la dernière publication de Windows Server, à savoir la version 1909.

| Version de Windows Server (étiquette flottante) | Version de mise à jour pour la publication du 14/1/20 (1B)| Version de mise à jour pour la publication du 18/2/20 (2B) | Version de mise à jour pour la publication du 10/3/20 (3B) |
|---|---|---|---|
| Windows Server 2016 (ltsc2016) | 10.0.14393.3443 (KB4534271) | 10.0.14393.3506 (KB4546850) | 10.0.14393.3568 (KB4551573) |
| Windows Server, version 1803 (1803) | 10.0.17134.1246 (KB4534293) | 10.0.17134.1305 (KB4546851)  | Cette version a atteint la fin du support. Pour plus d’informations, consultez [Cycles de vie de la maintenance des images de base](base-image-lifecycle.md).|
| Windows Server, version 1809 (1809)| 10.0.17763.973 (KB4534273) | 10.0.17763.1040 (KB4546852) | 10.0.17763.1098 (KB4538461) |
| Windows Server 2019 (ltsc2019) | 10.0.17763.973 (KB4534273) | 10.0.17763.1040 (KB4546852) | 10.0.17763.1098 (KB4538461) |
| Windows Server, version 1903 (1903) |10.0.18362.592 (KB4528760) | 10.0.18362.658 (KB4546853) | 10.0.18362.719 (KB4540673) |
| Windows Server, version 1909 (1909) | 10.0.18363.592 (KB4528760) | 10.0.18363.658 (KB4546853) | 10.0.18363.719 (KB4540673) |

## <a name="troubleshoot-host-and-container-image-mismatches"></a>Résoudre les discordances d’images d’hôte et de conteneur

Avant de commencer, prenez connaissance des informations sur la [Compatibilité des versions](version-compatibility.md). Ces informations vous aideront à déterminer si votre problème est dû à une discordance de correctifs. Si vous établissez que telle est bien la cause du problème, vous pouvez suivre les instructions de cette section pour le résoudre.

### <a name="query-the-version-of-your-container-host"></a>Interroger la version de l’hôte de votre conteneur

Si vous pouvez accéder à l’hôte de votre conteneur, vous pouvez exécuter la commande `ver` pour obtenir la version de son système d’exploitation. Par exemple, si vous exécutez `ver` sur un système exécutant Windows Server 2019 avec la dernière publication de la mise à jour de sécurité de février 2020, vous verrez ceci :

```batch
Microsoft Windows [Version 10.0.17763.1039]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>ver 

Microsoft Windows [Version 10.0.17763.1039]
```

>[!NOTE]
>Dans cet exemple, le numéro de révision est 1039, non 1040, parce que la publication de la mise à jour de sécurité de février 2020 n’avait pas de publication 2B hors plage pour Windows Server. Il n’existait de publication 2B hors plage que pour les conteneurs, dont le numéro de révision était 1040.

Si vous n’avez pas d’accès direct à l’hôte de votre conteneur, contactez votre administrateur informatique. Si vous opérez sur le cloud, consultez le site web du fournisseur de cloud pour connaître la version du système d’exploitation de l’hôte du conteneur qu’il exécute. Par exemple, si vous utilisez Azure Kubernetes Service (AKS), vous trouverez la version du système d’exploitation de l’hôte dans les [notes de publication d’AKS](https://github.com/Azure/AKS/releases).

### <a name="query-the-version-of-your-container-image"></a>Interroger la version de l’image de votre conteneur

Pour déterminer la version que votre conteneur exécute, procédez comme suit :

1. Exécutez la cmdlet suivante dans PowerShell :

    ```powershell
    docker images
    ```

    La sortie doit ressembler à ceci :

     ```powershell
     REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
     mcr.microsoft.com/windows/servercore   ltsc2019            b456290f487c        4 weeks ago         4.84GB
     mcr.microsoft.com/windows              1809                58229ca44fa7        4 weeks ago         12GB
     mcr.microsoft.com/windows/nanoserver   1809                f519d4f3a868        4 weeks ago         251M

2. Run the `docker inspect` command for the Image ID of the container image that isn't working. This will tell you which version the container image is targeting.

   For example, let's say we `run docker inspect` for an ltsc 2019 container image:

   ```powershell
   docker inspect b456290f487c

       "Architecture": "amd64",

        "Os": "windows",

        "OsVersion": "10.0.17763.1039",

        "Size": 4841309825,

        "VirtualSize": 4841309825,
    ```

    Dans cet exemple, la version du système d’exploitation du conteneur est `10.0.17763.1039`.

    Si vous exécutez déjà un conteneur, vous pouvez également exécuter la commande `ver` dans celui-ci pour obtenir sa version. Par exemple, l’exécution de la commande `ver` dans une image du conteneur Server Core de Windows Server 2019 avec la dernière publication de la mise à jour de sécurité de février 2020 indique ceci :

    ```batch
    Microsoft Windows [Version 10.0.17763.1040]
    (c) 2020 Microsoft Corporation. All rights reserved.

    C:\>ver

    Microsoft Windows [Version 10.0.17763.1040]
    ```

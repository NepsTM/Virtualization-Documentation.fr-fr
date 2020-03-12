---
title: Mettre à jour les conteneurs Windows Server
description: Comment Windows peut générer et exécuter des conteneurs dans plusieurs versions
keywords: métadonnées, conteneurs, version
author: heidilohr
ms. author: helohr
manager: lizross
ms.date: 03/10/2020
ms.openlocfilehash: 12a60398a12437ea733b2da31aae853a7afd2c57
ms.sourcegitcommit: 8eedfdc1fda9d0abb36e28dc2b5fb39891777364
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/10/2020
ms.locfileid: "79037416"
---
# <a name="update-windows-server-containers"></a>Mettre à jour les conteneurs Windows Server

Dans le cadre de la maintenance de Windows Server chaque mois, nous publions régulièrement des images de conteneur de système d’exploitation de base Windows Server mises à jour. Avec ces mises à jour, vous pouvez automatiser la création d’images de conteneur mises à jour ou les mettre à jour manuellement en tirant la dernière version. Les conteneurs Windows Server n’ont pas de pile de maintenance comme Windows Server. Vous ne pouvez pas obtenir les mises à jour au sein d’un conteneur comme vous le feriez avec Windows Server. Par conséquent, chaque mois, nous reconstruisons les images de conteneur du système d’exploitation de base Windows Server avec les mises à jour et publions les images de conteneur mises à jour.

D’autres images de conteneur, telles que .NET ou IIS, sont reconstruites en fonction des images de conteneur de système d’exploitation de base mises à jour et publiées chaque mois.

## <a name="how-to-get-windows-server-container-updates"></a>Obtention des mises à jour des conteneurs Windows Server

Nous mettons à jour les images de conteneur du système d’exploitation de base Windows Server en alignement avec la cadence de maintenance Windows. Les images de conteneur mises à jour sont publiées le deuxième mardi de chaque mois, que nous faisons parfois référence à notre version « B », avec un numéro de préfixe basé sur le mois de sortie. Par exemple, nous appelons la mise à jour de février « 2B » et notre mise à jour de mars « 3B ». Cet événement de mise à jour mensuelle est la seule version standard qui inclut de nouveaux correctifs de sécurité.

Le serveur hébergeant ces conteneurs, appelé hôte de conteneur ou simplement « hôte », peut être desservi pendant des événements de mise à jour supplémentaires autres que les versions « B ». Pour en savoir plus sur la cadence de maintenance de Windows Update, consultez notre billet de blog sur la [cadence de maintenance de Windows Update](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/windows-10-update-servicing-cadence/ba-p/222376) .

Les nouvelles images de conteneur du système d’exploitation de base Windows Server sont publiées peu de temps après 10:00 PST le deuxième mardi de chaque mois de la Container Registry Microsoft, et les balises proposées ciblent la version B la plus récente. Quelques exemples :

- ltsc2019 [(LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc): docker pull MCR.Microsoft.com/Windows/ServerCore :ltsc2019
- 1909 [(sac)](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel): docker pull MCR.Microsoft.com/Windows/ServerCore :1909

Si vous êtes plus familiarisé avec le concentrateur de station d’accueil que la taille de la plateforme, ce billet de [blog](https://azure.microsoft.com/blog/microsoft-syndicates-container-catalog/) vous donnera une explication plus détaillée.  

Pour chaque version, l’image de conteneur correspondante est également publiée avec deux balises supplémentaires pour le numéro de révision et le numéro d’article de la base de connaissances pour cibler des révisions d’image de conteneur spécifiques. Par exemple :

- docker pull mcr.microsoft.com/windows/servercore :10.0.17763.1040
- docker pull mcr.microsoft.com/windows/servercore :1809-KB4546852

Ces exemples extrayent l’image de conteneur Server Core de Windows Server 2019 avec la mise à jour de sécurité de février 18.  

Pour obtenir la liste complète des images de conteneur du système d’exploitation de base Windows Server, des versions et de leurs balises respectives, consultez les [images conteneur du système d’exploitation de base Windows](https://hub.docker.com/_/microsoft-windows-base-os-images) sur le hub d’ancrage.

Les images Windows Server Service mensuelles publiées sur la place de marché Microsoft Azure sont également fournies avec des images de conteneur de système d’exploitation de base préinstallées. Pour plus d’informations, consultez notre page de tarification de la place de [marché Windows Server Azure](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=PlansAndPrice). Nous mettons généralement à jour ces images environ cinq jours ouvrés après la version « B ».

Pour obtenir la liste complète des images et versions de Windows Server, consultez [Windows Server version sur l’historique des mises à jour](https://support.microsoft.com/help/4497947/windows-server-release-on-azure-marketplace-update-history)de la place de marché Azure.

## <a name="host-and-container-version-compatibility"></a>Compatibilité des versions des hôtes et des conteneurs

Il existe deux types de modes d’isolation pour les conteneurs Windows : l’isolation des processus et l’isolation Hyper-V. L’isolation Hyper-V est plus flexible lorsqu’il s’agit de la compatibilité des versions d’hôte et de conteneur. Pour plus d’informations, consultez [compatibilité des versions](version-compatibility.md) et [modes d’isolation](../manage-containers/hyperv-container.md). Cette section se concentre sur les conteneurs isolés du processus, sauf indication contraire.

Lorsque vous mettez à jour votre hôte de conteneur ou votre image de conteneur avec les mises à jour mensuelles, tant que l’image de conteneur et l’ordinateur hôte sont toutes deux prises en charge (Windows Server version 1809 ou ultérieure), les révisions d’image de l’hôte et du conteneur n’ont pas besoin de correspondre pour que le conteneur démarre et Exécutez normalement.

Toutefois, vous pouvez rencontrer des problèmes lors de l’utilisation de conteneurs Windows Server avec la version de mise à jour de sécurité du 11 février 2020 (également appelée « 2B ») ou des mises à jour de sécurité mensuelles ultérieures. Pour plus d’informations, consultez [cet article support Microsoft](https://support.microsoft.com/help/4542617/you-might-encounter-issues-when-using-windows-server-containers-with-t) . Ces problèmes résultent d’une modification de sécurité qui nécessitait une interface entre le mode utilisateur et le mode noyau pour assurer la sécurité de vos applications. Ces problèmes se produisent uniquement sur les conteneurs isolés du processus, car les conteneurs de processus isolés partagent le mode noyau avec l’hôte de conteneur. Cela signifie que les images de conteneur sans le composant de mode utilisateur mis à jour n’étaient pas sécurisées et ne sont pas compatibles avec la nouvelle interface de noyau sécurisé.

Nous avons publié un correctif le 18 février 2020. Cette nouvelle version a établi une « nouvelle ligne de base ». Cette nouvelle ligne de base suit les règles suivantes :

- Toutes les combinaisons d’hôtes et de conteneurs qui sont à la fois antérieures à 2B fonctionnent.
- Toute combinaison d’hôtes et de conteneurs qui sont à la fois postérieurs à 2B fonctionnera.
- Aucune combinaison d’hôtes et de conteneurs sur différents côtés de la nouvelle ligne de base ne fonctionnera. Par exemple, un hôte 3B et un conteneur 1B ne fonctionneront pas.

Nous allons utiliser la version de mise à jour de sécurité mensuelle de mars 2020 comme exemple pour vous montrer comment ces nouvelles règles de compatibilité fonctionnent. Dans le tableau suivant, la version de la mise à jour de sécurité de mars 2020 est appelée « 3B ». la mise à jour du février 2020 est « 2B » et la mise à jour du 1er 2020 janvier est « 1B ».

| Host | Conteneur | Compatibilité |
|---|---|---|
| Dollars | Dollars | Oui |
| Dollars | b | Oui |
| Dollars | 1B ou version antérieure | Non |
| b | Dollars | Oui |
| b | b | Oui |
| b | 1B ou version antérieure | Non |
| 1B ou version antérieure | Dollars | Non |
| 1B ou version antérieure | b | Non |
| 1B ou version antérieure | 1B ou version antérieure | Oui |

Pour référence, le tableau suivant répertorie les numéros de version pour les images de conteneur de système d’exploitation de base avec les mises à jour de sécurité mensuelles 1B, 2B et 3B sur les différentes versions de système d’exploitation majeures de Windows Server 2016 vers la version 1909 la plus récente de Windows Server.

| Version de Windows Server (étiquette flottante) | Version de mise à jour pour la version 1/14/20 (1B)| Version de mise à jour pour la version 2/18/20 (2B) | Mise à jour de la version 3/10/20 (3B) |
|---|---|---|---|
| Windows Server 2016 (ltsc2016) | 10.0.14393.3443 (B4534271) | 10.0.14393.3506 (KB4546850) | Conteneur à publier dans les jours à venir |
| Windows Server, version 1803 (1803) | 10.0.17134.1246 (KB4534293) | 10.0.17134.1305 (KB4546851)  | Cette version a atteint la fin de la prise en charge. Pour plus d’informations, consultez [cycles de vie de maintenance des images de base](base-image-lifecycle.md).|
| Windows Server 2019 (ltsc2019) | 10.0.17763.973 (KB4534273) | 10.0.17763.1040 (KB4546852) | 10.0.17763.1098 (KB4538461) |
| Windows Server, version 1903 (1903) |10.0.18362.592 (KB4528760) | 10.0.18362.658 (KB4546853) | 10.0.18362.719 (KB4540673) |
| Windows Server, version 1909 (1909) | 10.0.18363.592 (KB4528760) | 10.0.18363.658 (KB4546853) | 10.0.18363.719 (KB4540673) |

## <a name="troubleshoot-host-and-container-image-mismatches"></a>Résoudre les incompatibilités d’image d’hôte et de conteneur

Avant de commencer, veillez à vous familiariser avec les informations de [compatibilité des versions](version-compatibility.md). Ces informations vous aideront à déterminer si votre problème est dû à des correctifs incompatibles. Une fois que vous avez établi une erreur de mise en correspondance des correctifs, vous pouvez suivre les instructions de cette section pour résoudre le problème.

### <a name="query-the-version-of-your-container-host"></a>Interroger la version de votre hôte de conteneur

Si vous pouvez accéder à votre hôte de conteneur, vous pouvez exécuter la commande `ver` pour récupérer sa version du système d’exploitation. Par exemple, si vous exécutez `ver` sur un système qui exécute Windows Server 2019 avec la dernière version de la mise à jour de sécurité de février 2020, vous verrez ceci :

```batch
Microsoft Windows [Version 10.0.17763.1039]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>ver 

Microsoft Windows [Version 10.0.17763.1039]
```

>[!NOTE]
>Le numéro de révision de cet exemple s’affiche sous la forme 1039, et non 1040, car la version de la mise à jour de sécurité de février 2020 n’a pas de version 2B hors bande pour Windows Server. Il n’existait qu’une version 2B hors bande pour les conteneurs, qui avait un numéro de révision de 1040.

Si vous n’avez pas d’accès direct à votre hôte de conteneur, contactez votre administrateur informatique. Si vous exécutez sur le Cloud, consultez le site Web du fournisseur de Cloud pour connaître la version du système d’exploitation hôte du conteneur qu’il exécute. Par exemple, si vous utilisez le service Azure Kubernetes (AKS), vous trouverez la version du système d’exploitation hôte dans les [notes de publication de AKS](https://github.com/Azure/AKS/releases).

### <a name="query-the-version-of-your-container-image"></a>Interroger la version de votre image de conteneur

Suivez ces instructions pour déterminer quelle version votre conteneur est en cours d’exécution :

1. Exécutez l’applet de commande suivante dans PowerShell :

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

    Dans cet exemple, la version du système d’exploitation du conteneur apparaît comme `10.0.17763.1039`.

    Si vous exécutez déjà un conteneur, vous pouvez également exécuter la commande `ver` dans le conteneur lui-même pour obtenir la version. Par exemple, l’exécution de `ver` dans une image de conteneur Server Core de Windows Server 2019 avec la dernière version de la mise à jour de sécurité de février 2020 vous indiquera ceci :

    ```batch
    Microsoft Windows [Version 10.0.17763.1040]
    (c) 2020 Microsoft Corporation. All rights reserved.

    C:\>ver

    Microsoft Windows [Version 10.0.17763.1040]
    ```

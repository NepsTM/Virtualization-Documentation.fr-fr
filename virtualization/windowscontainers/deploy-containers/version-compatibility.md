---
title: Compatibilité des versions avec les conteneurs Windows
description: Comment Windows peut générer et exécuter des conteneurs dans plusieurs versions
keywords: métadonnées, conteneurs, version
author: taylorb-microsoft
ms.openlocfilehash: 23258d9181bb3c89cc59de3ba534cc6643c170f4
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681009"
---
# <a name="windows-container-version-compatibility"></a>Compatibilité des versions des conteneurs Windows

Windows Server 2016 et la mise à jour anniversaire de Windows 10 (version 14393) étaient les premières versions de Windows qui pouvaient générer et exécuter des conteneurs Windows Server. Les conteneurs créés à l’aide de ces versions peuvent s’exécuter sur des versions plus récentes, telles que Windows Server version1709, mais vous devez être au fait de certains éléments avant de commencer.

Étant donné que nous avons amélioré les fonctionnalités liées aux conteneurs Windows, nous avons dû apporter des modifications qui affectent la compatibilité. Les conteneurs plus anciens s’exécutent de la même manière sur les hôtes plus récents dotés [d’une isolation Hyper-V](../manage-containers/hyperv-container.md)et utilisent la même version du noyau (plus ancienne). Toutefois, si vous voulez exécuter un conteneur sur la base d’une nouvelle version de Windows, il ne peut être exécuté que sur la build d’hôte la plus récente.

|Version du système d’exploitation du conteneur|Version du système d’exploitation de l‘hôte|Compatibilité|
|---|---|---|
|Windows Server2016<br>Builds: 14393.* |Windows Server2016<br>Builds: 14393.* |Prise `process` en `hyperv` charge ou isolement|
|Windows Server2016<br>Builds: 14393.* |WindowsServer version1709<br>Builds16299.* |Ne prend `hyperv` en charge que l’isolement|
|Windows Server2016<br>Builds: 14393.* |Windows10 Fall Creators Update<br>Builds16299.* |Ne prend `hyperv` en charge que l’isolement|
|Windows Server2016<br>Builds: 14393.* |Windows Server version 1803<br>Builds 17134. * |Ne prend `hyperv` en charge que l’isolement|
|Windows Server2016<br>Builds: 14393.* |Windows10 version1803<br>Builds 17134. * |Ne prend `hyperv` en charge que l’isolement|
|Windows Server2016<br>Builds: 14393.* |Windows Server2019<br>Builds 17763. * |Ne prend `hyperv` en charge que l’isolement|
|Windows Server2016<br>Builds: 14393.* |Windows10, version1809<br>Builds 17763. * |Ne prend `hyperv` en charge que l’isolement|
|WindowsServer, version1709<br>Builds16299.* |Windows Server2016<br>Builds: 14393.* |Non prise en charge|
|WindowsServer, version1709<br>Builds16299.* |WindowsServer, version1709<br>Builds16299.* |Prise `process` en `hyperv` charge ou isolement|
|WindowsServer, version1709<br>Builds16299.* |Windows10 Fall Creators Update<br>Builds16299.* |Ne prend `hyperv` en charge que l’isolement|
|WindowsServer, version1709<br>Builds16299.* |WindowsServer, version1803<br>Builds 17134. * |Ne prend `hyperv` en charge que l’isolement|
|WindowsServer, version1709<br>Builds16299.* |Windows10 version1803<br>Builds 17134. * |Ne prend `hyperv` en charge que l’isolement|
|WindowsServer, version1709<br>Builds16299.* |Windows Server2019<br>Builds 17763. * |Ne prend `hyperv` en charge que l’isolement|
|WindowsServer, version1709<br>Builds16299.* |Windows10, version1809<br>Builds 17763. * |Ne prend `hyperv` en charge que l’isolement|
|WindowsServer, version1803<br>Builds 17134. * |Windows Server2016<br>Builds: 14393.* |Non prise en charge|
|WindowsServer, version1803<br>Builds 17134. * |WindowsServer, version1709<br>Builds16299.* |Non pris en charge|
|WindowsServer, version1803<br>Builds 17134. * |Windows10 Fall Creators Update<br>Builds16299.* |Non pris en charge|
|WindowsServer, version1803<br>Builds 17134. * |WindowsServer, version1803<br>Builds 17134. * |Prise `process` en `hyperv` charge ou isolement|
|WindowsServer, version1803<br>Builds 17134. * |Windows10 version1803<br>Builds 17134. * |Ne prend `hyperv` en charge que l’isolement|
|WindowsServer, version1803<br>Builds 17134. * |Windows Server2019<br>Builds 17763. * |Ne prend `hyperv` en charge que l’isolement|
|WindowsServer, version1803<br>Builds 17134. * |Windows10, version1809<br>Builds 17763. * |Ne prend `hyperv` en charge que l’isolement|
|Windows Server2019<br>Builds 17763. * |Windows Server2016<br>Builds: 14393.* |Non prise en charge|
|Windows Server2019<br>Builds 17763. * |WindowsServer, version1709<br>Builds16299.* |Non pris en charge
|Windows Server2019<br>Builds 17763. * |Windows10 Fall Creators Update<br>Builds16299.* |Non pris en charge|
|Windows Server2019<br>Builds 17763. * |WindowsServer, version1803<br>Builds 17134. * |Non prise en charge|
|Windows Server2019<br>Builds 17763. * |Windows10 version1803<br>Builds 17134. * |Non prise en charge|
|Windows Server2019<br>Builds 17763. * |Windows Server2019<br>Builds 17763. * |Prise `process` en `hyperv` charge ou isolement|
|Windows Server2019<br>Builds 17763. * |Windows10, version1809<br>Builds 17763. * |Ne prend `hyperv` en charge que l’isolement|

## <a name="matching-container-host-version-with-container-image-versions"></a>Version d’hôte de conteneur correspondante avec les versions d’image de conteneur

### <a name="windows-server-containers"></a>Conteneurs WindowsServer

Étant donné que les conteneurs Windows Server et l’hôte sous-jacent partagent un seul noyau, l’image de base du conteneur doit correspondre à celle de l’hôte. Si les versions sont différentes, le conteneur risque de démarrer, mais la fonction complet n’est pas garantie. Le système d’exploitation Windows comporte quatre niveaux de contrôle de version: Major, minor, Build et révision. Par exemple, la version 10.0.14393.103 aurait une version majeure de 10, une version mineure de 0, un numéro de build de 14393 et un numéro de révision d' 103. Le numéro de version n’est modifié que lorsque de nouvelles versions du système d’exploitation sont publiées, comme la version 1709, 1803, la mise à jour de créateurs de créateurs, etc. Le numéro de révision est mis à jour quand des mises à jour Windows sont appliquées.

#### <a name="build-number-new-release-of-windows"></a>Numéro de build (nouvelle version de Windows)

Le démarrage des conteneurs Windows Server est bloqué lorsque le numéro de build entre l’hôte du conteneur et l’image du conteneur est différent. Par exemple, lorsque l’hôte de conteneur est version 10.0.14393. * (Windows Server 2016) et que l’image du conteneur est version 10.0.16299. * (Windows Server version 1709), le conteneur ne démarre pas.  

#### <a name="revision-number-patching"></a>Numéro de révision (correction)

Les conteneurs Windows Server ne sont pas bloqués lorsque les numéros de révision de l’hôte de conteneur et l’image du conteneur sont différents. Par exemple, si l’hôte de conteneur est version 10.0.14393.1914 (Windows Server 2016 avec KB4051033 appliqué) et que l’image du conteneur est version 10.0.14393.1944 (Windows Server 2016 avec KB4053579 appliqué), l’image continuera de s’exécuter, même si leur version les numéros sont différents.

Pour les hôtes ou images Windows Server 2016, la révision de l’image du conteneur doit correspondre à celle de l’hôte afin qu’elle soit prise en charge. Toutefois, pour les hôtes ou les images utilisant Windows Server version 1709 ou ultérieure, cette règle n’est pas applicable, et l’image hôte et du conteneur n’a pas besoin de révisions correspondantes. Nous vous recommandons de mettre à jour vos systèmes avec les derniers correctifs et mises à jour.

#### <a name="practical-application"></a>Application pratique

Exemple 1: l’hôte de conteneur exécute Windows Server 2016 avec KB4041691 appliqué. Tout conteneur Windows Server déployé sur cet hôte doit être basé sur les images de base du conteneur version 10.0.14393.1770. Si vous appliquez KB4053579 au conteneur hôte, vous devez également mettre à jour les images pour vous assurer que le conteneur hôte les prend en charge.

Exemple 2: l’hôte de conteneur exécute Windows Server version 1709 avec KB4043961 appliqué. Tout conteneur Windows Server déployé sur cet hôte doit être basé sur une image de base du conteneur Windows Server version 1709 (10.0.16299), mais il n’est pas nécessaire de faire correspondre l’hôte Ko. Si KB4054517 est appliqué à l’hôte, les images de conteneur seront toujours prises en charge, mais nous vous recommandons de les mettre à jour pour résoudre les problèmes de sécurité potentiels.

#### <a name="querying-version"></a>Interrogation de la version

Méthode 1: nouveauté de la version 1709, la commande de l’invite de commandes et la commande **ver** renvoient désormais les détails de la révision.

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

Méthode 2: interroger la clé de Registre suivante: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion

Exemple :

```batch
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```

```batch
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

Pour vérifier la version utilisée par votre image de base, passez en revue les balises sur le concentrateur ou la table de hachage d’image fournie dans la description de l’image. La page [historique des mises à jour de Windows 10](https://support.microsoft.com/help/12387/windows-10-update-history) répertorie les pages de lancement de chaque Build et révision.

### <a name="hyper-v-isolation-for-containers"></a>Isolement Hyper-V pour les conteneurs

Vous pouvez exécuter des conteneurs Windows avec ou sans l’isolement Hyper-V. L’isolation Hyper-V crée une limite sécurisée autour du conteneur avec un ordinateur virtuel optimisé. À la différence des conteneurs Windows standard qui partagent le noyau entre les conteneurs et l’hôte, chaque conteneur Hyper-V isolé dispose de sa propre instance du noyau Windows. Cela signifie que vous pouvez utiliser des versions de système d’exploitation différentes dans l’hôte de conteneur et l’image (pour plus d’informations, consultez la matrice de compatibilité suivante).  

Pour exécuter un conteneur ayant une isolation Hyper-V, ajoutez simplement la balise `--isolation=hyperv` à votre commande docker run.

## <a name="errors-from-mismatched-versions"></a>Erreurs en cas d’incompatibilité des versions

Si vous essayez d’exécuter une combinaison non prise en charge, vous recevez le message d’erreur suivant:

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

Vous pouvez résoudre ce problème de trois manières:

- Reconstruisez le conteneur en fonction de la `mcr.microsoft.com/windows/nanoserver` version appropriée de ou `mcr.microsoft.com/windows/servercore`
- S’il s’agit de l’hôte plus récent, exécutez l’outil **d’amarrage-isolement = HyperV...**
- Essayez d’exécuter le conteneur sur un autre hôte avec la même version de Windows

## <a name="choose-which-container-os-version-to-use"></a>Choisir la version du système d’exploitation conteneur à utiliser

<<<<<<< HEAD
>[!NOTE]
>À compter du 16 avril 2019, la balise «la plus récente» n’est plus publiée ou mise à jour pour les [images du conteneur du système d’exploitation Windows](https://hub.docker.com/_/microsoft-windows-base-os-images). Déclarez une balise spécifique lorsque vous soulevez ou référencez des images à partir de ces pensions.

<a name="you-must-know-which-version-you-need-to-use-for-your-container-for-example-if-you-want-windows-server-version-1809-as-your-container-os-and-want-to-have-the-latest-patches-for-it-you-should-use-the-tag-1809-when-specifying-which-version-of-the-base-os-container-images-you-want-like-so"></a>Vous devez déterminer la version que vous devez utiliser pour votre conteneur. Par exemple, si vous souhaitez que Windows Server version 1809 comme système d’exploitation de conteneur et que vous vouliez disposer des derniers correctifs, vous `1809` devez utiliser la balise lorsque vous spécifiez la version des images de conteneur du système d’exploitation de base que vous voulez utiliser, comme suit:
=======
Il est important de vous assurer de connaître la version du système d’exploitation de conteneur dont vous avez besoin. Si vous utilisez Windows Server version1709 et souhaitez obtenir les derniers correctifs qui y sont applicables, vous devez utiliser la balise «1709» lorsque vous spécifiez la version des images du conteneur du système d’exploitation de base que vous voulez, comme suit:
>>>>>>> origine/maître

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809
...
```

Toutefois, si vous voulez un correctif de Windows Server version 1809, vous pouvez spécifier le numéro KB dans la balise. Par exemple, pour obtenir une image de conteneur de systèmes d’exploitation de base nano Server à partir de Windows Server version 1809 avec le KB4493509 appliqué, vous pouvez le spécifier comme suit:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809-KB4493509
...
```

Vous pouvez également spécifier les correctifs exacts dont vous avez besoin avec le schéma que nous avons utilisé précédemment en spécifiant la version du système d’exploitation dans la balise:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:10.0.17763.437
...
```

Les images de base du serveur principal en fonction de Windows Server 2019 et de Windows Server 2016 sont des publications [de canal de maintenance à long terme (LTSC)](https://docs.microsoft.com/en-us/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc) . Si vous souhaitez que Windows Server 2019 soit le système d’exploitation du conteneur de l’image principale du serveur et que vous souhaitez obtenir les derniers correctifs, vous pouvez spécifier des publications LTSC comme suit:

``` dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019
...
```

## <a name="matching-versions-using-docker-swarm"></a>Correspondance des versions à l’aide de Docker Swarm

L’élément Essaimr n’est pas actuellement intégré pour correspondre à la version de Windows qu’un conteneur utilise avec la même version. Si vous mettez à jour le service pour utiliser un nouveau conteneur, il s’exécutera correctement.

Si vous avez besoin d’exécuter plusieurs versions de Windows pendant une longue période, vous pouvez effectuer les deux approches suivantes: configurez les hôtes Windows pour qu’ils utilisent toujours l’isolation Hyper-V ou utiliser des contraintes d’étiquette.

### <a name="finding-a-service-that-wont-start"></a>Recherche d’un service qui ne démarre pas

Si un service ne démarre pas, vous verrez que `MODE` c' `replicated` est `REPLICAS` le cas, mais il sera bloqué sur 0. Pour déterminer si la version du système d’exploitation correspond au problème, exécutez les commandes suivantes:

Exécutez le **service docker LS** pour trouver le nom du service:

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

Exécutez le **service d’ancrage PS (nom du service)** pour obtenir l’État et les dernières tentatives:

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

Si vous voyez `starting container failed: ...`, vous pouvez voir le message d’erreur complet avec le **service d’ancrage PS--no-tronque (nom du conteneur)**:

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

Il s’agit de la même `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`erreur que.

### <a name="fix---update-the-service-to-use-a-matching-version"></a>Correctif: mettre à jour le service pour utiliser une version correspondante

Deux éléments sont à prendre en compte pour Docker Swarm. Si vous disposez d’un fichier de rédaction qui comporte un service qui utilise une image que vous n’avez pas créée, vous pouvez mettre à jour la référence en conséquence. Exemple :

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

L’autre considération consiste à faire en sorte que l’image que vous pointez soit celle que vous avez créée vous-même (par exemple, contoso/MyImage):

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

Dans ce cas, vous devez utiliser la méthode décrite dans les [Erreurs de versions](#errors-from-mismatched-versions) inadaptées pour modifier ce dockerfile à la place de la ligne de composition de l’ancrage.

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>Atténuation: utiliser l’isolation Hyper-V avec Docker Swarm

Il existe une proposition de prise en charge de l’isolement Hyper-V par le biais d’un conteneur, mais ce code n’est pas encore terminé. Vous pouvez suivre la progression sur [GitHub](https://github.com/moby/moby/issues/31616). Jusqu’à ce que cela soit prêt, les ordinateurs hôtes doivent être configurés de manière à toujours s’exécuter avec l’isolation Hyper-V.

Cela nécessite la modification de la configuration du service Docker, puis le redémarrage du moteur Docker.

1. Modifiez `C:\ProgramData\docker\config\daemon.json`
2. Ajouter une ligne contenant `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >Le fichier daemon. JSON n’existe pas par défaut. Si, après vérification dans le répertoire, cela s’avère être effectivement le cas, vous devez créer le fichier. Vous pouvez ensuite copier les éléments suivants:

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. Fermez et enregistrez le fichier, puis redémarrez le moteur de l’ancrage en exécutant les applets de commande suivantes dans PowerShell:

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. Après avoir redémarré le service, lancez vos conteneurs. Une fois qu’ils sont exécutés, vous pouvez vérifier le niveau d’isolement d’un conteneur en inspectant le conteneur avec l’applet de commande suivante:

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

Soit «process», soit «hyperv» sera renvoyé. Si vous avez modifié et défini votre fichier daemon.json comme décrit ci-dessus, il doit indiquer cette dernière éventualité.

### <a name="mitigation---use-labels-and-constraints"></a>Atténuation: utiliser des étiquettes et des contraintes

Voici comment utiliser les étiquettes et les contraintes pour faire correspondre les versions:

1. Ajoutez des étiquettes à chaque nœud.

    Sur chaque nœud, ajoutez deux étiquettes: `OS` et `OsVersion`. Cela suppose une exécution locale, mais pouvez modifier cela afin de les définir sur un hôte distant.

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    Par la suite, vous pouvez vérifier ces éléments en exécutant la commande d' **inspection du nœud** de l’ancrage qui doit afficher les étiquettes que vous venez d’ajouter:

    ```yaml
           "Spec": {
                "Labels": {
                   "OS": "windows",
                   "OsVersion": "10.0.16296"
               },
                "Role": "manager",
                "Availability": "active"
            }
    ```

2. Ajoutez une contrainte de service.

    À présent que vous avez étiqueté chaque nœud, vous pouvez mettre à jour des contraintes qui déterminent le positionnement des services. Dans l’exemple suivant, remplacez «contoso_service» par le nom de votre service réel:

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    Cela applique et limite l’emplacement d’exécution d’un nœud.

Pour en savoir plus sur l’utilisation des contraintes de service, voir la [référence de création de service](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint).

## <a name="matching-versions-using-kubernetes"></a>Correspondance des version à l’aide de Kubernetes

Le même problème décrit dans les [versions correspondantes utilisant l'](#matching-versions-using-docker-swarm) effet d’amarrage peut se produire lorsque les gousses sont planifiées dans Kubernetes. Pour éviter ce problème, vous pouvez procéder comme suit:

- Reconstruisez le conteneur en fonction de la même version de système d’exploitation en développement et en production. Pour savoir comment procéder, voir [choisir la version du système d’exploitation conteneur à utiliser](#choose-which-container-os-version-to-use).
- Utiliser des étiquettes de nœud et nodeSelectors pour vous assurer que les gousses sont planifiées sur des nœuds compatibles si les nœuds Windows Server 2016 et Windows Server version 1709 se trouvent dans le même cluster
- Utilisez des clusters distincts en fonction de la version du système d’exploitation

### <a name="finding-pods-failed-on-os-mismatch"></a>Échec de la recherche des pods en raison de l’incompatibilité des systèmes d’exploitation

Dans le cas présent, un déploiement comprenait un module qui a été planifié sur un nœud ayant une version de système d’exploitation incorrecte et sans l’isolation Hyper-V.

Le même message d’erreur s’affiche dans les événements répertoriés, avec `kubectl describe pod <podname>`. Après plusieurs tentatives, le statut Pod sera probablement `CrashLoopBackOff`.

```
$ kubectl -n plang describe pod fabrikamfiber.web-789699744-rqv6p

Name:           fabrikamfiber.web-789699744-rqv6p
Namespace:      plang
Node:           38519acs9011/10.240.0.6
Start Time:     Mon, 09 Oct 2017 19:40:30 +0000
Labels:         io.kompose.service=fabrikamfiber.web
                pod-template-hash=789699744
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-789699744","uid":"b5062a08-ad29-11e7-b16e-000d3a...
Status:         Running
IP:             10.244.3.169
Created By:     ReplicaSet/fabrikamfiber.web-789699744
Controlled By:  ReplicaSet/fabrikamfiber.web-789699744
Containers:
  fabrikamfiberweb:
    Container ID:       docker://eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a
    Image:              patricklang/fabrikamfiber.web:latest
    Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
    Port:               80/TCP
    State:              Waiting
      Reason:           CrashLoopBackOff
    Last State:         Terminated
      Reason:           ContainerCannotRun
      Message:          container eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{037b6606-bc9c-461f-ae02-829c28410798}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Layers":[{"ID":"f8bc427f-7aa3-59c6-b271-7331713e9451","Path":"C:\\ProgramData\\docker\\windowsfilter\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881"},{"ID":"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47","Path":"C:\\ProgramData\\docker\\windowsfilter\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5"},{"ID":"4f624ca7-2c6d-5c42-b73f-be6e6baf2530","Path":"C:\\ProgramData\\docker\\windowsfilter\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67"},{"ID":"88e360ff-32af-521d-9a3f-3760c12b35e2","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e"},{"ID":"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a","Path":"C:\\ProgramData\\docker\\windowsfilter\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461"},{"ID":"c2b3d728-4879-5343-a92a-b735752a4724","Path":"C:\\ProgramData\\docker\\windowsfilter\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3"},{"ID":"2973e760-dc59-5800-a3de-ab9d93be81e5","Path":"C:\\ProgramData\\docker\\windowsfilter\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75"},{"ID":"454a7d36-038c-5364-8a25-fa84091869d6","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0"},{"ID":"9b748c8c-69eb-55fb-a1c1-5688cac4efd8","Path":"C:\\ProgramData\\docker\\windowsfilter\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec"},{"ID":"bfde5c26-b33f-5424-9405-9d69c2fea4d0","Path":"C:\\ProgramData\\docker\\windowsfilter\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2"},{"ID":"bdabfbf5-80d1-57f1-86f3-448ce97e2d05","Path":"C:\\ProgramData\\docker\\windowsfilter\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf"},{"ID":"ad9b34f2-dcee-59ea-8962-b30704ae6331","Path":"C:\\ProgramData\\docker\\windowsfilter\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8"}],"HostName":"fabrikamfiber.web-789699744-rqv6p","MappedDirectories":[{"HostPath":"c:\\var\\lib\\kubelet\\pods\\b50f0027-ad29-11e7-b16e-000d3afd2878\\volumes\\kubernetes.io~secret\\default-token-rw9dn","ContainerPath":"c:\\var\\run\\secrets\\kubernetes.io\\serviceaccount","ReadOnly":true,"BandwidthMaximum":0,"IOPSMaximum":0}],"HvPartition":false,"EndpointList":null,"NetworkSharedContainerName":"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13","Servicing":false,"AllowUnqualifiedDNSQuery":false}
      Exit Code:        128
      Started:          Mon, 09 Oct 2017 20:27:08 +0000
      Finished:         Mon, 09 Oct 2017 20:27:08 +0000
    Ready:              False
    Restart Count:      10
    Environment:        <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
Conditions:
  Type          Status
  Initialized   True
  Ready         False
  PodScheduled  True
Volumes:
  default-token-rw9dn:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-rw9dn
    Optional:   false
QoS Class:      BestEffort
Node-Selectors: beta.kubernetes.io/os=windows
Tolerations:    <none>
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
  ---------     --------        -----   ----                    -------------                           --------        ------                  -------
  49m           49m             1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-789699744-rqv6p to 38519acs9011
  49m           49m             1       kubelet, 38519acs9011                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
  29m           29m             1       kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Failed to pull image "patricklang/fabrikamfiber.web:latest": rpc error: code = 2 desc = Error response from daemon: {"message":"Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io: no such host"}
  49m           3m              12      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
  33m           2m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Error: failed to start container "fabrikamfiberweb": Error response from daemon: {"message":"container fabrikamfiberweb encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {\"SystemType\":\"Container\",\"Name\":\"fabrikamfiberweb\",\"Owner\":\"docker\",\"IsDummy\":false,\"VolumePath\":\"\\\\\\\\?\\\\Volume{037b6606-bc9c-461f-ae02-829c28410798}\",\"IgnoreFlushesDuringBoot\":true,\"LayerFolderPath\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\fabrikamfiberweb\",\"Layers\":[{\"ID\":\"f8bc427f-7aa3-59c6-b271-7331713e9451\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881\"},{\"ID\":\"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5\"},{\"ID\":\"4f624ca7-2c6d-5c42-b73f-be6e6baf2530\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67\"},{\"ID\":\"88e360ff-32af-521d-9a3f-3760c12b35e2\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e\"},{\"ID\":\"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461\"},{\"ID\":\"c2b3d728-4879-5343-a92a-b735752a4724\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3\"},{\"ID\":\"2973e760-dc59-5800-a3de-ab9d93be81e5\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75\"},{\"ID\":\"454a7d36-038c-5364-8a25-fa84091869d6\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0\"},{\"ID\":\"9b748c8c-69eb-55fb-a1c1-5688cac4efd8\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec\"},{\"ID\":\"bfde5c26-b33f-5424-9405-9d69c2fea4d0\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2\"},{\"ID\":\"bdabfbf5-80d1-57f1-86f3-448ce97e2d05\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf\"},{\"ID\":\"ad9b34f2-dcee-59ea-8962-b30704ae6331\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8\"}],\"HostName\":\"fabrikamfiber.web-789699744-rqv6p\",\"MappedDirectories\":[{\"HostPath\":\"c:\\\\var\\\\lib\\\\kubelet\\\\pods\\\\b50f0027-ad29-11e7-b16e-000d3afd2878\\\\volumes\\\\kubernetes.io~secret\\\\default-token-rw9dn\",\"ContainerPath\":\"c:\\\\var\\\\run\\\\secrets\\\\kubernetes.io\\\\serviceaccount\",\"ReadOnly\":true,\"BandwidthMaximum\":0,\"IOPSMaximum\":0}],\"HvPartition\":false,\"EndpointList\":null,\"NetworkSharedContainerName\":\"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13\",\"Servicing\":false,\"AllowUnqualifiedDNSQuery\":false}"}
  33m           11s             151     kubelet, 38519acs9011                                           Warning         FailedSync              Error syncing pod
  32m           11s             139     kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         BackOff                 Back-off restarting failed container
```

### <a name="mitigation---using-node-labels-and-nodeselector"></a>Réduction-utilisation d’étiquettes de nœud et de nodeSelector

Exécutez **kubectl nœud Get** pour obtenir la liste de tous les nœuds. Après cela, vous pouvez exécuter **kubectl décrit le nœud (nom du nœud)** pour obtenir plus de détails.

Dans l’exemple suivant, deux nœuds Windows s’exécutent avec des versions différentes:

```
$ kubectl get node

NAME                        STATUS    AGE       VERSION
38519acs9010                Ready     21h       v1.7.7-7+e79c96c8ff2d8e
38519acs9011                Ready     4h        v1.7.7-25+bc3094f1d650a2
k8s-linuxpool1-38519084-0   Ready     21h       v1.7.7
k8s-master-38519084-0       Ready     21h       v1.7.7

$ kubectl describe node 38519acs9010

Name:                   38519acs9010
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_D2_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9010
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 01:41:02 +0000

...
  
System Info:
 Machine ID:                    38519acs9010
 System UUID:
 Boot ID:
 Kernel Version:                10.0 14393 (14393.1715.amd64fre.rs1_release_inmarket.170906-1810)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
 ...
 
$ kubectl describe node 38519acs9011
Name:                   38519acs9011
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_DS1_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9011
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 18:13:25 +0000
Conditions:
...

System Info:
 Machine ID:                    38519acs9011
 System UUID:
 Boot ID:
 Kernel Version:                10.0 16299 (16299.0.amd64fre.rs3_release.170922-1354)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
...

```

Nous allons utiliser cet exemple pour montrer comment faire correspondre les versions:

1. Prenez note de chaque nom de nœud `Kernel Version` et de vos informations système.

    Dans notre exemple, les informations se présentent comme suit:

    Nom         | Version
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. Ajoutez à chaque nœud une étiquette appelée `beta.kubernetes.io/osbuild`. Pour être pris en charge sans l’isolement Hyper-V, vous devez disposer des versions principales et secondaires de Windows Server 2016 (14393,1715 dans cet exemple). Windows Server version 1709 a uniquement besoin de la version principale (16299 dans cet exemple) pour correspondre.

    Dans cet exemple, la commande permettant d’ajouter les étiquettes est semblable à ce qui suit:

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. Vérifiez que les étiquettes sont disponibles en exécutant **kubectl Get Nodes--Show-labels**.

    Dans cet exemple, le résultat se présente comme suit:

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. Ajoutez des sélecteurs de nœud aux déploiements. Dans cet exemple, nous allons ajouter a `nodeSelector` à la spécification du conteneur avec `beta.kubernetes.io/os` = Windows et `beta.kubernetes.io/osbuild` = 14393. * ou 16299 pour correspondre au système d’exploitation de base utilisé par le conteneur.

    Voici un exemple complet pour l’exécution d’un conteneur conçu pour Windows Server2016:

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      annotations:
        kompose.cmd: kompose convert -f docker-compose-combined.yml
        kompose.version: 1.2.0 (99f88ef)
      creationTimestamp: null
      labels:
        io.kompose.service: fabrikamfiber.web
      name: fabrikamfiber.web
    spec:
      replicas: 1
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            io.kompose.service: fabrikamfiber.web
        spec:
          containers:
          - image: patricklang/fabrikamfiber.web:latest
            name: fabrikamfiberweb
            ports:
            - containerPort: 80
            resources: {}
          restartPolicy: Always
          nodeSelector:
            "beta.kubernetes.io/os": windows
            "beta.kubernetes.io/osbuild": "14393.1715"
    status: {}
    ```

    Le pod peut désormais lancer le déploiement mis à jour. Les sélecteurs de nœuds apparaissent également `kubectl describe pod <podname>`dans, de sorte que vous pouvez exécuter cette commande pour vérifier qu’elles ont été ajoutées.

    Le résultat de notre exemple est le suivant:

    ```
    $ kubectl -n plang describe po fa

    Name:           fabrikamfiber.web-1780117715-5c8vw
    Namespace:      plang
    Node:           38519acs9010/10.240.0.4
    Start Time:     Tue, 10 Oct 2017 01:43:28 +0000
    Labels:         io.kompose.service=fabrikamfiber.web
                    pod-template-hash=1780117715
    Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-1780117715","uid":"6a07aaf3-ad5c-11e7-b16e-000d3...
    Status:         Running
    IP:             10.244.1.84
    Created By:     ReplicaSet/fabrikamfiber.web-1780117715
    Controlled By:  ReplicaSet/fabrikamfiber.web-1780117715
    Containers:
      fabrikamfiberweb:
        Container ID:       docker://c94594fb53161f3821cf050d9af7546991aaafbeab41d333d9f64291327fae13
        Image:              patricklang/fabrikamfiber.web:latest
        Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
        Port:               80/TCP
        State:              Running
          Started:          Tue, 10 Oct 2017 01:43:42 +0000
        Ready:              True
        Restart Count:      0
        Environment:        <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
    Conditions:
      Type          Status
      Initialized   True
      Ready         True
      PodScheduled  True
    Volumes:
      default-token-rw9dn:
        Type:       Secret (a volume populated by a Secret)
        SecretName: default-token-rw9dn
        Optional:   false
    QoS Class:      BestEffort
    Node-Selectors: beta.kubernetes.io/os=windows
                    beta.kubernetes.io/osbuild=14393.1715
    Tolerations:    <none>
    Events:
      FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
      ---------     --------        -----   ----                    -------------                           --------        ------                  -------
      5m            5m              1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-1780117715-5c8vw to 38519acs9010
      5m            5m              1       kubelet, 38519acs9010                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Started                 Started container
    ```

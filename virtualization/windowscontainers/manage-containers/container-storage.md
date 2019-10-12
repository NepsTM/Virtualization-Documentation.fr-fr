---
title: Vue d’ensemble du stockage de conteneurs
description: Comment les conteneurs Windows Server peuvent-ils utiliser les hôtes et les autres types de stockage
keywords: conteneurs, volume, stockage, montage, montage lié
author: cwilhit
ms.openlocfilehash: fba08de884d59cc1b656895ec2b7078ba3975269
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209749"
---
# <a name="container-storage-overview"></a>Vue d’ensemble du stockage de conteneurs

<!-- Great diagram would be great! -->

Cette rubrique fournit une vue d’ensemble des différents modes d’utilisation du stockage sur Windows par les conteneurs. Les conteneurs se comportent différemment des machines virtuelles lorsqu’il s’agit d’un stockage. Par nature, les conteneurs sont conçus pour empêcher une application en cours d’exécution dans l’état d’écriture dans le système de fichiers de l’hôte. Les conteneurs utilisent un espace «scratch» par défaut, mais Windows fournit également un moyen de conserver le stockage.

## <a name="scratch-space"></a>Espace de travail

Les conteneurs Windows par défaut utilisent un stockage éphémère. Toutes les e/s de conteneur s’exécutent dans un «espace scratch» et chaque conteneur dispose d’une nouvelle propriété. La création de fichiers et les écritures de fichier sont capturées dans l’espace de travail et ne sont pas échappées à l’hôte. Lorsqu’une instance de conteneur est arrêtée, toutes les modifications apportées à l’espace de travail sont levées hors. Lorsqu’une nouvelle instance de conteneur est démarrée, un nouvel espace de travail est fourni pour l’instance.

## <a name="layer-storage"></a>Stockage par niveaux

Comme décrit dans [vue d’ensemble des conteneurs](../about/index.md), les images de conteneur sont un ensemble de fichiers exprimés sous la forme d’une série de couches. Le stockage de couches est l’ensemble des fichiers intégrés au conteneur. Chaque fois que vous effectuez un `docker pull`, puis un `docker run` de ce conteneur, vous obtenez un résultat identique.

### <a name="where-layers-are-stored-and-how-to-change-it"></a>Où sont stockées les niveaux et comment modifier cette configuration

Dans une installation par défaut, les niveaux sont stockés dans `C:\ProgramData\docker` et répartis dans les répertoires «image» et «windowsfilter». Vous pouvez changer l’emplacement de stockage des niveaux à l’aide de la configuration `docker-root`, comme illustré dans la documentation [Moteur Docker sur Windows](../manage-docker/configure-docker-daemon.md).

> [!NOTE]
> Seul NTFS est pris en charge pour le stockage de niveaux. ReFS n’est pas pris en charge.

Vous ne devez pas modifier les fichiers contenus dans les répertoires des niveaux: ils sont gérés avec soin à l’aide de commandes telles que:

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
- [docker load](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>Opérations prises en charge dans le stockage de niveaux

Les conteneurs en cours d’exécution peuvent utiliser la plupart des opérations de NTFS à l’exception des transactions. Cela implique de paramétrer des ACL et toutes les listes ACL sont vérifiées à l’intérieur du conteneur. Si vous souhaitez exécuter des processus en tant qu’utilisateurs multiples au sein d’un conteneur, vous pouvez créer des utilisateurs dans votre `Dockerfile` avec `RUN net user /create ...`, définir des ACL de fichier, puis configurer les processus à exécuter associés à chaque utilisateur à l’aide de la [directive Dockerfile USER](https://docs.docker.com/engine/reference/builder/#user).

## <a name="persistent-storage"></a>Stockage persistant

Les conteneurs Windows prennent en charge des mécanismes permettant de fournir un stockage persistant via les montages et les volumes bind. Pour en savoir plus, voir [stockage persistant dans les conteneurs](./persistent-storage.md).

## <a name="storage-limits"></a>Limites de stockage

Il est courant dans les applications Windows de lancer une requête sur la quantité d’espace disque disponible avant d’installer ou de créer de nouveaux fichiers, ou en tant que déclencheur pour le nettoyage des fichiers temporaires.  Pour optimiser la compatibilité des applications, le lecteur C: dans un conteneur Windows représente une taille libre virtuelle de 20 Go.

Certains utilisateurs peuvent souhaiter remplacer cette valeur par défaut et configurer l’espace libre sur une valeur plus petite ou plus grande. Cette opération peut être effectuée via l’option «Size» de la configuration «Storage-opt».

### <a name="examples"></a>Exemples

Ligne de commande: `docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

Vous pouvez ou modifier directement le fichier de configuration de l’ancrage:

```Docker Configuration File
"storage-opts": [
    "size=50GB"
  ]
```

> [!TIP]
> Cette méthode fonctionne également pour la version d’amarrage. Consultez le [document Configure docker](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file) pour plus d’informations sur la modification du fichier de configuration docker.

---
title: Stockage de conteneurs Windows Server
description: Comment les conteneurs Windows Server peuvent-ils utiliser les hôtes et les autres types de stockage
keywords: conteneurs, volume, stockage, montage, montage lié
author: patricklang
ms.openlocfilehash: ba30c436ddd61ec71b2c98d1a8cb24f97863d872
ms.sourcegitcommit: 6c8c70c8231943dda3c5af38e5530ea3dd91fc82
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/05/2018
ms.locfileid: "1934533"
---
# <a name="overview"></a>Vue d’ensemble

<!-- Great diagram would be great! -->


## <a name="layer-storage"></a>Stockage par niveaux

Il s’agit de tous les fichiers qui sont intégrés dans le conteneur. Chaque fois que vous effectuez un `docker pull`, puis un `docker run` de ce conteneur, vous obtenez un résultat identique.


### <a name="where-layers-are-stored-and-how-to-change-it"></a>Où sont stockées les niveaux et comment modifier cette configuration

Dans une installation par défaut, les niveaux sont stockés dans `C:\ProgramData\docker` et répartis dans les répertoires «image» et «windowsfilter». Vous pouvez changer l’emplacement de stockage des niveaux à l’aide de la configuration `docker-root`, comme illustré dans la documentation [Moteur Docker sur Windows](../manage-docker/configure_docker_daemon.md).

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


##  <a name="image-size"></a>Taille des images
Il est courant dans les applications Windows de lancer une requête sur la quantité d’espace disque disponible avant d’installer ou de créer de nouveaux fichiers, ou en tant que déclencheur pour le nettoyage des fichiers temporaires.  Avec l’objectif de renforcer la compatibilité des applications, le disqueC: d’un conteneur Windows représente un volume virtuel libre de 20Go.  Certains utilisateurs peuvent souhaiter remplacer cette valeur par défaut et paramétrer un espace libre plus petit ou plus grand. Pour ce faire, il suffit d’utiliser l’option «size» avec la configuration «storage-opt».

### <a name="examples"></a>Exemples
Ligne de commande: `docker run --storage-opt "size=50GB" microsoft/windowsservercore:1709 cmd`

Fichier de configuration de Docker
```
"storage-opts": [
    "size=50GB"
  ]
```
> Notez que cette méthode fonctionne pour la configuration Docker build.
Consultez le [document Configure docker](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file) pour plus d’informations sur la modification du fichier de configuration docker.


## <a name="persistent-volumes"></a>Volumes persistants

Un stockage persistant peut être attribué à des conteneurs de plusieurs façons:

- Montages liés
- Volumes nommés

Docker propose une excellente présentation de la façon dont [utiliser les volumes](https://docs.docker.com/engine/admin/volumes/volumes/) qu’il est préférable de lire au préalable. Le reste de cette page met l’accent sur les différences entre Linux et Windows et fournit des exemples sur Windows.


### <a name="bind-mounts"></a>Montages liés

[Les montages liés](https://docs.docker.com/engine/admin/volumes/bind-mounts/) permettent à un conteneur de partager un répertoire avec l’hôte. Cela s’avère utile lorsque vous avez besoin d’un espace de stockage de fichiers sur l’ordinateur local qui soit disponible quand vous redémarrez un conteneur, ou que vous puissiez partager avec plusieurs conteneurs. Si vous souhaitez que le conteneur s’exécute sur plusieurs ordinateurs avec un accès aux mêmes fichiers, alors un volume nommé ou un montage SMB doit être utilisé à la place.

#### <a name="permissions"></a>Autorisations

Le modèle d’autorisation utilisé pour les montages liés varie en fonction du niveau d’isolement pour votre conteneur.

À l’aide de conteneurs utilisant **l’isolement Hyper-V**, notamment les conteneurs Linux sur Windows Server version1709, utilisez un simple modèle d’autorisation en lecture seule ou en lecture-écriture.
Les fichiers sont accessibles sur l’ordinateur hôte à l’aide du compte `LocalSystem`. Si l’accès au conteneur vous est refusé, assurez-vous que `LocalSystem` a accès au répertoire sur l’ordinateur hôte.
Lorsque l’indicateur Lecture seule est utilisé, les modifications apportées au volume au sein du conteneur ne seront pas visibles ou persistantes dans le répertoire sur l’ordinateur hôte.

Les conteneurs Windows Server ayant recours à **l’isolement des processus** sont légèrement différents, car ils utilisent l’identité des processus au sein du conteneur pour accéder aux données, ce qui signifie que les ACL sont respectées.
L’identité du processus en cours d’exécution dans le conteneur («ContainerAdministrator» sur WindowsServerCore et «ContainerUser» dans les conteneurs Nano Server, par défaut) sera utilisée pour accéder aux fichiers et répertoires du volume monté au lieu de `LocalSystem` et l’accès devra être accordé pour l’exploitation des données.
Dans la mesure où ces identités existent uniquement dans le contexte du conteneur, et pas sur l’ordinateur hôte sur lequel les fichiers sont stockés, vous devez utiliser un groupe de sécurité connu, comme `Authenticated Users`, lors de la configuration des ACL pour accorder l’accès aux conteneurs.

> [!WARNING]
> Abstenez-vous de monter-lier des répertoires sensibles comme `C:\` dans un conteneur non approuvé. Cela permettrait de modifier des fichiers sur l’ordinateur hôte auxquels il ne serait normalement pas possible d’accéder, ce qui pourrait créer une faille de sécurité.

Exemple d’utilisation: 

- `docker run -v c:\ContainerData:c:\data:RO` pour un accès en lecture seule
- `docker run -v c:\ContainerData:c:\data:RW` pour un accès en lecture-écriture
- `docker run -v c:\ContainerData:c:\data` pour un accès en lecture-écriture (par défaut)

#### <a name="symlinks"></a>Liens symboliques

Les liens symboliques sont résolus dans le conteneur. Si vous liez-montez un chemin d’accès hôte vers un conteneur et que ce chemin est un lien symbolique ou contient des liens symboliques, le conteneur ne sera pas en mesure d’y accéder.

#### <a name="smb-mounts"></a>Montages SMB

Sur Windows Server version1709, une nouvelle fonctionnalité appelée «Mappage Global SMB» permet de monter un partage SMB sur l’ordinateur hôte, puis de transférer des répertoires via ce partage dans un conteneur. Le conteneur n’a pas besoin d’être configuré avec un serveur, un partage, un nom d’utilisateur ou un mot de passe spécifiques, ces éléments étant tous gérés sur l’ordinateur hôte. Le conteneur fonctionnera de la même façon que s’il possédait un stockage local.

##### <a name="configuration-steps"></a>Étapes de configuration

1. Sur l’hôte conteneur, mappez globalement le partage SMB distant: $creds = Get-Credential New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G: Cette commande utilise les informations d’identification pour s’authentifier auprès du serveur SMB distant. Ensuite, mappez le chemin d’accès du partage à distance à la lettre de lecteur G: (il peut s’agir de toute autre lettre de lecteur disponible). Il est désormais possible que les volumes de données des conteneurs créés sur cet hôte conteneur soient mappés à un chemin d’accès sur le lecteur G.

> Remarque: Lorsque le mappage global SMB est appliqué aux conteneurs, tous les utilisateurs de l’hôte conteneur peuvent accéder au partage distant. Toute application en cours d’exécution sur l’hôte conteneur aura également accès au partage distant mappé.

2. Créez des conteneurs avec des volumes de données mappés vers un partage SMB monté de manière globale: docker run -it --name demo -v g:\ContainerData:G:\AppData1 microsoft/windowsservercore:1709 cmd.exe

Au sein du conteneur, G:\AppData1 sera ensuite mappé au répertoire distant partagé «ContainerData». Toutes les données stockées sur un partage distant globalement mappé sera disponible pour les applications présentes dans le conteneur. Plusieurs conteneurs peuvent accéder en lecture/écriture à ces données partagées à l’aide de la même commande.

Cette prise en charge du mappage global SMB est une fonction côté client SMB qui peut fonctionner sur n’importe quel serveur SMB compatible, notamment:

- Serveur de fichiers ScaleOut basé sur des espaces de stockage Direct (S2D) ou un réseau SAN traditionnel
- Azure Files (partage SMB)
- Serveur de fichiers traditionnel
- Implémentation tierce du protocole SMB (ex: dispositifs NAS)

> [!NOTE]
> Le mappage global SMB ne prend pas en charge les partages DFS, DFSN, DFSR dans Windows Server version1709.

### <a name="named-volumes"></a>Volumes nommés

Les volumes nommés vous permettent de créer un volume par nom, de l’affecter à un conteneur et de le réutiliser plus tard via le même nom. Vous n’avez pas besoin de conserver le chemin d’accès effectif d’où il a été créé, son nom suffit. Le moteur Docker sur Windows dispose d’un plug-in intégré pour les volumes nommés qui permet de créer des volumes sur l’ordinateur local. Un plug-in supplémentaire est requis si vous souhaitez utiliser des volumes nommés sur plusieurs ordinateurs.

Exemples d’étapes:

1. `docker volume create unwound` - Création d’un volume nommé «Unwound»
2. `docker run -v unwound:c:\data microsoft/windowsservercore` - Démarrage d’un conteneur avec le volume mappé vers c:\data
3. Écriture de fichiers dans le répertoire c:\data du conteneur, puis arrêt du conteneur
4. `docker run -v unwound:c:\data microsoft/windowsservercore` - Démarrage d’un nouveau conteneur
5. Exécution de `dir c:\data`dans le nouveau conteneur - les fichiers sont toujours présents

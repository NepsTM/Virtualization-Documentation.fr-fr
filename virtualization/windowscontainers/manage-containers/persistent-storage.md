---
title: Stockage persistant dans les conteneurs
description: Comment les conteneurs Windows peuvent être stockés de manière permanente
keywords: conteneurs, volume, stockage, montage, montage lié
author: cwilhit
ms.openlocfilehash: 945a78d4ecb9c96da4de8f7246f84b6b444dd5b5
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909669"
---
# <a name="persistent-storage-in-containers"></a>Stockage persistant dans les conteneurs

<!-- Great diagram would be great! -->

Vous pouvez avoir des cas où il est important qu’une application soit en mesure de conserver des données dans un conteneur, ou que vous souhaitiez afficher des fichiers dans un conteneur qui n’étaient pas inclus au moment de la génération du conteneur. Un stockage persistant peut être donné aux conteneurs de deux manières :

- Montages liés
- Volumes nommés

Docker propose une excellente présentation de la façon dont [utiliser les volumes](https://docs.docker.com/engine/admin/volumes/volumes/) qu’il est préférable de lire au préalable. Le reste de cette page met l’accent sur les différences entre Linux et Windows et fournit des exemples sur Windows.

## <a name="bind-mounts"></a>Montages liés

[Les montages liés](https://docs.docker.com/engine/admin/volumes/bind-mounts/) permettent à un conteneur de partager un répertoire avec l’hôte. Cela s’avère utile lorsque vous avez besoin d’un espace de stockage de fichiers sur l’ordinateur local qui soit disponible quand vous redémarrez un conteneur, ou que vous puissiez partager avec plusieurs conteneurs. Si vous souhaitez que le conteneur s’exécute sur plusieurs ordinateurs avec un accès aux mêmes fichiers, alors un volume nommé ou un montage SMB doit être utilisé à la place.

### <a name="permissions"></a>Permissions

Le modèle d’autorisation utilisé pour les montages liés varie en fonction du niveau d’isolement pour votre conteneur.

Les conteneurs utilisant l' **isolation Hyper-V** utilisent un modèle d’autorisation simple en lecture seule ou en lecture-écriture. Les fichiers sont accessibles sur l’ordinateur hôte à l’aide du compte `LocalSystem`. Si l’accès au conteneur vous est refusé, assurez-vous que `LocalSystem` a accès au répertoire sur l’ordinateur hôte. Lorsque l’indicateur Lecture seule est utilisé, les modifications apportées au volume au sein du conteneur ne seront pas visibles ou persistantes dans le répertoire sur l’ordinateur hôte.

Les conteneurs Windows utilisant l' **isolation des processus** sont légèrement différents, car ils utilisent l’identité du processus dans le conteneur pour accéder aux données, ce qui signifie que les ACL du fichier sont honorées. L’identité du processus en cours d’exécution dans le conteneur (« ContainerAdministrator » sur Windows Server Core et « ContainerUser » dans les conteneurs Nano Server, par défaut) sera utilisée pour accéder aux fichiers et répertoires du volume monté au lieu de `LocalSystem` et l’accès devra être accordé pour l’exploitation des données.

Étant donné que ces identités existent uniquement dans le contexte du conteneur (et non sur l’hôte où les fichiers sont stockés), vous devez utiliser un groupe de sécurité connu comme `Authenticated Users` lors de la configuration des listes de contrôle d’accès (ACL) pour accorder l’accès aux conteneurs.

> [!WARNING]
> Abstenez-vous de monter-lier des répertoires sensibles comme `C:\` dans un conteneur non approuvé. Cela permettrait de modifier des fichiers sur l’ordinateur hôte auxquels il ne serait normalement pas possible d’accéder, ce qui pourrait créer une faille de sécurité.

Exemple d’utilisation :

- `docker run -v c:\ContainerData:c:\data:RO` pour un accès en lecture seule
- `docker run -v c:\ContainerData:c:\data:RW` pour l’accès en lecture-écriture
- `docker run -v c:\ContainerData:c:\data` pour l’accès en lecture-écriture (par défaut)

### <a name="symlinks"></a>Liens symboliques

Les liens symboliques sont résolus dans le conteneur. Si vous liez-montez un chemin d’accès hôte vers un conteneur et que ce chemin est un lien symbolique ou contient des liens symboliques, le conteneur ne sera pas en mesure d’y accéder.

### <a name="smb-mounts"></a>Montages SMB

Sur Windows Server version 1709 et versions ultérieures, la fonctionnalité appelée « mappage global SMB » permet de monter un partage SMB sur l’hôte, puis de transmettre des répertoires sur ce partage dans un conteneur. Le conteneur n’a pas besoin d’être configuré avec un serveur, un partage, un nom d’utilisateur ou un mot de passe spécifiques, ces éléments étant tous gérés sur l’ordinateur hôte. Le conteneur fonctionnera de la même façon que s’il possédait un stockage local.

#### <a name="configuration-steps"></a>Étapes de configuration

1. Sur l’hôte de conteneur, mappez globalement le partage SMB distant :
    ```
    $creds = Get-Credential
    New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
    ```
    Cette commande utilise les informations d’identification pour s’authentifier auprès du serveur SMB distant. Ensuite, mappez le chemin d’accès du partage à distance à la lettre de lecteur G: (il peut s’agir de toute autre lettre de lecteur disponible). Il est désormais possible que les volumes de données des conteneurs créés sur cet hôte conteneur soient mappés à un chemin d’accès sur le lecteur G.

    > [!NOTE]
    > Lorsque vous utilisez le mappage global SMB pour les conteneurs, tous les utilisateurs sur l’hôte de conteneur peuvent accéder au partage distant. Toute application en cours d’exécution sur l’hôte conteneur aura également accès au partage distant mappé.

2. Créez des conteneurs avec des volumes de données mappés vers un partage SMB monté de manière globale : docker run -it --name demo -v g:\ContainerData:G:\AppData1 microsoft/windowsservercore:1709 cmd.exe

    Au sein du conteneur, G:\AppData1 sera ensuite mappé au répertoire distant partagé « ContainerData ». Toutes les données stockées sur un partage distant globalement mappé sera disponible pour les applications présentes dans le conteneur. Plusieurs conteneurs peuvent accéder en lecture/écriture à ces données partagées à l’aide de la même commande.

Cette prise en charge du mappage global SMB est une fonction côté client SMB qui peut fonctionner sur n’importe quel serveur SMB compatible, notamment :

- Serveur de fichiers ScaleOut basé sur des espaces de stockage Direct (S2D) ou un réseau SAN traditionnel
- Azure Files (partage SMB)
- Serveur de fichiers traditionnel
- Implémentation tierce du protocole SMB (ex : dispositifs NAS)

> [!NOTE]
> Le mappage global SMB ne prend pas en charge les partages DFS, DFSN, DFSR dans Windows Server version 1709.

## <a name="named-volumes"></a>Volumes nommés

Les volumes nommés vous permettent de créer un volume par nom, de l’affecter à un conteneur et de le réutiliser plus tard via le même nom. Vous n’avez pas besoin de conserver le chemin d’accès effectif d’où il a été créé, son nom suffit. Le moteur Docker sur Windows dispose d’un plug-in intégré pour les volumes nommés qui permet de créer des volumes sur l’ordinateur local. Un plug-in supplémentaire est requis si vous souhaitez utiliser des volumes nommés sur plusieurs ordinateurs.

Exemples d’étapes :

1. `docker volume create unwound`-créer un volume nommé « non enroulé »
2. `docker run -v unwound:c:\data microsoft/windowsservercore`-démarrer un conteneur avec le volume mappé à c:\data
3. Écriture de fichiers dans le répertoire c:\data du conteneur, puis arrêt du conteneur
4. `docker run -v unwound:c:\data microsoft/windowsservercore`-démarrer un nouveau conteneur
5. Exécution de `dir c:\data`dans le nouveau conteneur - les fichiers sont toujours présents

> [!NOTE]
> Windows Server convertira les chemins d’accès cibles (le chemin d’accès à l’intérieur du conteneur) en minuscules ; i. e. `-v unwound:c:\MyData`, ou `-v unwound:/app/MyData` dans les conteneurs Linux, entraîne un répertoire à l’intérieur du conteneur de `c:\mydata`, ou `/app/mydata` dans des conteneurs Linux, qui est mappé (et créé, s’il n’existe pas).

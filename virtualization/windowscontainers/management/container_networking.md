---
title: "Mise en réseau de conteneurs Windows"
description: "Configurez la mise en réseau pour les conteneurs Windows."
keywords: docker, conteneurs
author: jmesser81
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
translationtype: Human Translation
ms.sourcegitcommit: c412171773e9c66569eab2252b5adfc187eedafd
ms.openlocfilehash: eb7d2c25d929cb51abfad17c26a89105f6574a48

---

# Mise en réseau de conteneur

Sur le plan de la mise en réseau, les conteneurs Windows fonctionnent de la même façon que des machines virtuelles. Chaque conteneur a une carte réseau virtuelle qui est connectée à un commutateur virtuel sur lequel le trafic entrant et sortant est transféré. Pour appliquer une isolation entre les conteneurs sur le même hôte, un compartiment réseau est créé pour chaque conteneur Windows Server et Hyper-V dans lequel est installée la carte réseau pour le conteneur. Les conteneurs Windows Server utilisent une carte réseau virtuelle hôte pour s’attacher au commutateur virtuel. Les conteneurs Hyper-V utilisent une carte réseau de machine virtuelle synthétique (non exposée à la machine virtuelle d’utilitaire) pour s’attacher au commutateur virtuel.

Les conteneurs Windows prennent en charge les quatre modes ou pilotes réseau suivants : *nat*, *transparent*, *l2bridge* et *l2tunnel*. Choisissez le mode réseau le mieux adapté à l’infrastructure de votre réseau physique et à la configuration requise du réseau (hôte unique ou hôtes multiples). 

Le moteur Docker crée un réseau nat par défaut à la première exécution du service Docker. Le préfixe IP interne par défaut créé est 172.16.0.0/12. 

> Remarque : Si votre IP hôte de conteneur est dans ce même préfixe, vous devez modifier le préfixe IP interne de NAT comme indiqué ci-dessous.

Les points de terminaison de conteneur seront attachés à ce réseau par défaut, et une adresse IP leur sera attribuée à partir du préfixe interne. Un seul réseau NAT est actuellement pris en charge dans Windows (une [requête de tirage](https://github.com/docker/docker/pull/25097) en attente peut toutefois vous aider à contourner cette limitation). 

Il est possible de créer des réseaux supplémentaires sur le même hôte de conteneur en utilisant un pilote différent (par exemple, transparent, l2bridge). Le tableau ci-dessous montre de quelle manière est fournie la connectivité réseau pour les connexions internes (conteneur-conteneur) et externes dans chaque mode.

- **NAT** : Chaque conteneur reçoit une adresse IP à partir d’un préfixe IP privé interne (par exemple, 172.16.0.0/12). Le réacheminement/mappage de ports à partir de l’hôte de conteneur vers des points de terminaison de conteneur est pris en charge.

- **Transparent** : Chaque point de terminaison de conteneur est directement connecté au réseau physique. Les adresses IP issues du réseau physique peuvent être attribuées de façon statique ou dynamique à l’aide d’un serveur DHCP externe.

- **L2 Bridge** : Chaque point de terminaison de conteneur se trouve dans le même sous-réseau IP que l’hôte de conteneur. Les adresses IP doivent être attribuées de façon statique à partir du même préfixe que l’hôte de conteneur. Tous les points de terminaison de conteneur sur l’hôte ont la même adresse MAC en raison de la traduction d’adresses Layer-2.

- **L2 Tunnel** - _ : Ce mode doit uniquement être utilisé dans une pile Microsoft Cloud Stack_.

> Pour savoir comment connecter des points de terminaison de conteneur à un réseau virtuel de superposition avec la pile Microsoft SDN, consultez la rubrique relative à l’[attachement de conteneurs à un réseau virtuel](location).

## Nœud unique

|  | Conteneur-conteneur | Conteneur-externe |
| :---: | :---------------     |  :---                |
| nat | Connexion reliée par un pont via un commutateur virtuel Hyper-V | routage via WinNAT avec application des traductions d’adresses | 
| transparent | Connexion reliée par un pont via un commutateur virtuel Hyper-V | accès direct au réseau physique | 
| l2bridge | Connexion reliée par un pont via un commutateur virtuel Hyper-V|  accès au réseau physique avec la traduction d’adresses MAC|  



## Nœuds multiples

|  | Conteneur-conteneur | Conteneur-externe |
| :---: | :----       | :---------- |
| nat | doit référencer le port et l’IP de l’hôte de conteneur externe ; routage via WinNAT avec application des traductions d’adresses | doit référencer l’hôte externe ; routage via WinNAT avec application des traductions d’adresses | 
| transparent | doit directement référencer le point de terminaison IP du conteneur | accès direct au réseau physique | 
| l2bridge | doit directement référencer le point de terminaison IP du conteneur| accès au réseau physique avec la traduction d’adresses MAC| 


## Création de réseau 

### Réseau NAT (par défaut)

Le moteur Docker Windows crée un réseau nat par défaut avec le préfixe IP 172.16.0.0/12. Un seul réseau nat est autorisé sur un hôte de conteneur Windows. Si un utilisateur souhaite créer un réseau nat avec un préfixe IP spécifique, il peut le faire de deux façons en modifiant les options définies dans le fichier config daemon.json de Docker (si ce fichier n’existe pas dans C:\ProgramData\Docker\config\daemon.json, créez-le).
 1. Utilisez l’option _"fixed-cidr": "< Préfixe IP > / Mask"_ pour créer le réseau nat par défaut avec le préfixe IP et la correspondance spécifiés
 2. Utilisez l’option _"bridge": "none"_ pour ne pas créer le réseau par défaut et permettre à un utilisateur de créer un réseau avec le pilote de son choix à l’aide de la commande *docker network create -d <driver>*

Avant d’appliquer l’une de ces options de configuration, le service Docker doit être arrêté et les réseaux nat existants doivent être supprimés.

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

Si l’utilisateur a ajouté l’option "fixed-cidr" dans le fichier daemon.json, le moteur Docker crée un réseau nat défini par l’utilisateur avec le préfixe IP et le masque personnalisés spécifiés. Si l’utilisateur a plutôt ajouté l’option "bridge:none", le réseau doit être créé manuellement.

```none
# Create a user-defined nat network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

Par défaut, les points de terminaison de conteneur sont connectés au réseau nat par défaut. Si le réseau nat par défaut n’a pas été créé (du fait que l’option "bridge:none" a été spécifiée dans le fichier daemon.json) ou si l’accès à un autre réseau défini par l’utilisateur est obligatoire, les utilisateurs peuvent spécifier le paramètre *--network* dans la commande docker run.

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### Mappage de ports

Pour accéder à des applications s’exécutant à l’intérieur d’un conteneur connecté à un réseau NAT, des mappages de ports doivent être créés entre l’hôte de conteneur et le point de terminaison de conteneur. Ces mappages doivent être spécifiés au moment de la CRÉATION du conteneur ou quand le conteneur est dans un état ARRÊTÉ.

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Create a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

Les mappages de ports dynamiques sont également autorisés en utilisant le paramètre -p ou la commande EXPOSE dans un fichier Dockerfile avec le paramètre -P. Un port éphémère est choisi au hasard sur l’hôte de conteneur et peut être inspecté lors de l’exécution de docker ps.

```none
C:\> docker run -itd -p 80 windowsservercore cmd

# Network services running on port TCP:80 in this container can be accessed externally on port TCP:14824
C:\> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
bbf72109b1fc        windowsservercore   "cmd"               6 seconds ago       Up 2 seconds        *0.0.0.0:14824->80/tcp*   drunk_stonebraker

# Container image specified EXPOSE 80 in Dockerfile - publish this port mapping
C:\> docker network 
```
> Dans WS2016 TP5 et les builds Windows Insider supérieures à 14300, une règle de pare-feu est automatiquement créée pour tous les mappages de ports NAT. Cette règle de pare-feu affecte globalement l’hôte de conteneur et n’est pas limitée à une carte réseau ni un point de terminaison de conteneur spécifique.

L’implémentation Windows NAT (WinNAT) présente quelques lacunes de fonctionnalité décrites dans le billet de blog [WinNAT capabilities and limitations](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations/) (Fonctionnalités et limitations de WinNAT) 
 1. Un seul réseau NAT (un seul préfixe IP interne) est pris en charge sur un hôte de conteneur
 2. Les points de terminaison de conteneur sont accessibles uniquement à partir de l’hôte de conteneur avec le port et l’adresse IP interne spécifiés

Des réseaux supplémentaires peuvent être créés avec des pilotes différents. 

> Les noms des pilotes réseau Docker s’écrivent en minuscules.

### Réseau transparent

Pour utiliser le mode de mise en réseau transparent, créez un réseau de conteneurs avec le nom de pilote « transparent ». 

```none
C:\> docker network create -d transparent MyTransparentNetwork
```

Si l’hôte de conteneur est virtualisé et que vous souhaitez utiliser DHCP pour l’attribution d’adresses IP, vous devez activer MACAddressSpoofing sur la carte réseau de machines virtuelles. Sinon, l’hôte Hyper-V bloque le trafic réseau provenant des conteneurs sur la machine virtuelle utilisant plusieurs adresses MAC.

```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> Pour créer plusieurs réseaux transparents, vous devez spécifier la carte réseau (virtuelle) à laquelle le commutateur virtuel Hyper-V externe (créé automatiquement) doit être lié.

Pour lier un réseau (attaché via le commutateur virtuel Hyper-V) à une interface réseau spécifique, utilisez l’option *-o com.docker.network.windowsshim.interface=<Interface>*

```none
# Create a transparent network which is attached to the "Ethernet 2" network interface
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

La valeur pour *com.docker.network.windowsshim.interface* est le *Nom* de la carte provenant de : 
```none
Get-NetAdapter
```

Les adresses IP pour les points de terminaison de conteneur connectés à un réseau transparent peuvent être attribuées de façon statique ou dynamique à partir d’un serveur DHCP externe.

Quand vous utilisez l’attribution IP statique, assurez-vous d’abord que les paramètres *--subnet* et *--gateway* sont spécifiés au moment de la création du réseau. L’adresse IP de sous-réseau et de passerelle doit être la même que les paramètres réseau de l’hôte de conteneur (le réseau physique).

```none
# Create a transparent network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 TransparentNet3
```
Spécifiez une adresse IP en utilisant l’option *--ip* dans la commande docker run

```none
C:\> docker run -it --network=TransparentNet3 --ip 10.123.174.105 <image> <cmd>
```

> Assurez-vous que cette adresse IP n’est pas déjà attribuée à un autre périphérique réseau sur le réseau physique

Vous n’avez pas besoin de spécifier les mappages de ports, car les points de terminaison de conteneur ont directement accès au réseau physique

### L2 Bridge 

Pour utiliser le mode de mise en réseau pont de couche 2, créez un réseau de conteneurs avec le nom de pilote « l2bridge ». Vous devez également spécifier un sous-réseau et une passerelle correspondant au réseau physique.

```none
C:\> docker network create -d l2bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyBridgeNetwork
```

Seule l’attribution IP statique est prise en charge pour les réseaux l2bridge. 

> Si vous utilisez un réseau l2bridge sur une structure SDN, seule l’attribution IP dynamique est prise en charge. Pour plus d’informations, consultez la rubrique relative à l’[attachement de conteneurs à un réseau virtuel](location).

## Autres opérations et configurations

### Répertorier les réseaux disponibles

```none
# list container networks
C:\> docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
0a297065f06a        nat                 nat                 local
d42516aa0250        none                null                local
```

### Supprimer un réseau

Utilisez `docker network rm` pour supprimer un réseau de conteneurs.

```none
C:\> docker network rm "<network name>"
```

Cette commande supprime tous les commutateurs virtuels Hyper-V utilisés par le réseau de conteneurs ainsi que toutes les traductions d’adresses réseau (WinNAT - instances NetNat) créées.

### Inspection du réseau 

Pour afficher les conteneurs qui sont connectés à un réseau spécifique et les adresses IP associées à ces points de terminaison de conteneur, vous pouvez exécuter la commande suivante.

```none
C:\> docker network inspect <network name>
```

### Plusieurs réseaux de conteneurs

Plusieurs réseaux de conteneurs peuvent être créés sur un hôte de conteneur unique avec les mises en garde suivantes :
* Un seul réseau NAT peut être créé par hôte de conteneur.
* Plusieurs réseaux qui utilisent un vSwitch externe pour la connectivité (par exemple, Transparent, L2 Bridge, L2 Transparent) doivent chacun utiliser leur propre carte réseau.

### Sélection du réseau

Quand vous créez un conteneur Windows, vous pouvez spécifier un réseau auquel la carte réseau de conteneur est connectée. Si aucun réseau n’est spécifié, le réseau NAT par défaut est utilisé.

Pour pouvoir attacher un conteneur au réseau NAT non défini par défaut, utilisez l’option --network avec la commande docker run.

```none
C:\> docker run -it --network=MyTransparentNet windowsservercore cmd
```

### Adresse IP statique

```none
C:\> docker run -it --network=MyTransparentNet --ip=10.80.123.32 windowsservercore cmd
```

L’attribution d’adresses IP statiques est effectuée directement sur la carte réseau du conteneur et ne doit être réalisée que quand le conteneur est dans un état ARRÊTÉ. Ni un « ajout à chaud » de cartes réseau de conteneurs ni un apport de modifications à la pile réseau ne sont pris en charge pendant l’exécution du conteneur.


## Mises en garde et pièges

### Pare-feu

L’hôte de conteneur nécessite la création de règles de pare-feu spécifiques pour activer ICMP (Ping) et DHCP. ICMP et DHCP sont nécessaires aux conteneurs Windows Server pour effectuer un test ping entre deux conteneurs sur le même hôte et pour recevoir dynamiquement des adresses IP attribuées via DHCP. Dans TP5, ces règles sont créées via le script Install-ContainerHost.ps1. Après TP5, ces règles seront créées automatiquement. Toutes les règles de pare-feu correspondant aux règles de réacheminement de port NAT sont créées automatiquement et nettoyées lors de l’arrêt du conteneur.

### Fonctionnalités non prises en charge

Les fonctionnalités de mise en réseau suivantes ne sont pas prises en charge pour le moment via l’interface de ligne de commande Docker
 * liaison de conteneurs (par exemple, --link)
 * résolution d’adresses IP en fonction du nom/service pour les conteneurs. _Cette fonctionnalité sera bientôt possible lors d’une mise à jour de maintenance_
 * pilote réseau de superposition par défaut

Les options réseau suivantes ne sont pas prises en charge sur Windows Docker à ce stade :
 * --add-host
 * --dns
 * --dns-opt
 * --dns-search
 * -h, --hostname
 * --net-alias
 * --aux-address
 * --internal
 * --ip-range

 > Il existe un bogue connu dans Windows Server 2016 Technical Preview 5 et les builds récentes de WIP (Windows Insider Preview) où la mise à niveau vers une nouvelle build aboutit à un commutateur virtuel et un réseau de conteneurs en double. Pour contourner ce problème, exécutez le script suivant.
```none
$KeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
$keys = get-childitem $KeyPath
foreach($key in $keys)
{
   if ($key.GetValue("FriendlyName") -eq 'nat')
   {
      $newKeyPath = $KeyPath+"\"+$key.PSChildName
      Remove-Item -Path $newKeyPath -Recurse
   }
}
remove-netnat -Confirm:$false
Get-ContainerNetwork | Remove-ContainerNetwork
Get-VmSwitch -Name nat | Remove-VmSwitch # Note: failure is expected
Stop-Service docker
Set-Service docker -StartupType Disabled
```
> Redémarrez l’hôte, puis exécutez les étapes restantes :
```none
Get-NetNat | Remove-NetNat -Confirm $false
Set-Service docker -StartupType automatic
Start-Service docker 
```



<!--HONumber=Aug16_HO3-->



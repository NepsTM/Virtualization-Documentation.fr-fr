



# Mise en réseau de conteneur

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.**

Sur le plan de la mise en réseau, les conteneurs Windows fonctionnent de la même façon que des machines virtuelles. Chaque conteneur possède une carte réseau virtuelle qui est connectée à un commutateur virtuel sur lequel le trafic entrant et sortant est transféré. Deux types de configurations de réseau sont disponibles.

- **Mode traduction d’adresses réseau (NAT)** : chaque conteneur est connecté à un commutateur virtuel interne et reçoit une adresse IP interne. Une configuration NAT traduit cette adresse interne en l’adresse externe de l’hôte du conteneur.

- **Mode transparent** : chaque conteneur est connecté à un commutateur virtuel externe et reçoit une adresse IP d’un serveur DHCP.

Ce document décrit en détail l’avantage et la configuration de chacun de ces modes.

## Mode de mise en réseau NAT

**Traduction d’adresses réseau (NAT)** : cette configuration se compose d’un commutateur réseau interne avec un type de NAT et WinNat. Dans cette configuration, le conteneur hôte possède une adresse IP « externe » accessible sur un réseau. Une adresse « interne » est assignée à chaque conteneur, qui n’est pas accessible sur un réseau. Pour rendre un conteneur accessible dans cette configuration, un port externe de l’ordinateur hôte est mappé à un port interne du conteneur. Ces mappages sont stockés dans une table de mappage de ports NAT. Le conteneur est accessible via l’adresse IP et le port externe de l’hôte, qui transfère le trafic vers l’adresse IP interne et le port du conteneur. L’avantage du mode NAT est que l’hôte du conteneur peut être étendu à des centaines de conteneurs, tout en n’utilisant qu’une seule adresse IP disponible en externe. En outre, le mode NAT permet à plusieurs conteneurs d’héberger des applications qui peuvent nécessiter des ports de communication identiques.

### Configuration de l’hôte

Pour configurer l’hôte du conteneur pour la traduction d’adresses réseau, procédez comme suit.

Créez un commutateur virtuel de type « NAT » et configurez-le avec un sous-réseau interne. Pour plus d’informations sur la commande **New-VMSwitch**, consultez les [informations de référence sur New-VMSwitch](https://technet.microsoft.com/en-us/library/hh848455.aspx).

```powershell
New-VMSwitch -Name "NAT" -SwitchType NAT -NATSubnetAddress "172.16.0.0/12"
```
Créez l’objet traduction d’adresses réseau. Il s’agit de l’objet responsable de la traduction d’adresses NAT. Pour plus d’informations sur la commande **New-NetNat**, consultez les [informations de référence sur New-NetNat](https://technet.microsoft.com/en-us/library/dn283361.aspx).

```powershell
New-NetNat -Name NAT -InternalIPInterfaceAddressPrefix "172.16.0.0/12" 
```

### Configuration du conteneur

Lorsque vous créez un conteneur Windows, vous pouvez sélectionner un commutateur virtuel pour celui-ci. Lorsque le conteneur est connecté à un commutateur virtuel configuré pour utiliser NAT, le conteneur reçoit une adresse traduite.

Cet exemple crée un conteneur connecté à un commutateur virtuel compatible avec NAT.

```powershell
New-Container -Name DemoNAT -ContainerImageName WindowsServerCore -SwitchName "NAT"
```

Une fois le conteneur démarré, l’adresse IP est visible dans le conteneur.

```powershell
[DemoNAT]: PS C:\> ipconfig

Windows IP Configuration
Ethernet adapter vEthernet (Virtual Switch-527ED2FB-D56D-4852-AD7B-E83732A032F5-0):
   Connection-specific DNS Suffix  . : contoso.com
   Link-local IPv6 Address . . . . . : fe80::384e:a23d:3c4b:a227%16
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.240.0.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

Pour plus d’informations sur le démarrage et la connexion à un conteneur Windows, consultez [Gestion des conteneurs](./manage_containers.md).

### Mappage de ports

Pour accéder à des applications à l’intérieur d’un conteneur « compatible avec NAT », des mappages de ports doivent être créés entre le conteneur et son hôte. Pour créer le mappage, vous avez besoin de l’adresse IP du conteneur, du port de conteneur « interne » et d’un port d’hôte « externe ».

Cet exemple montre comment créer un mappage entre le port **80** de l’hôte et le port **80** d’un conteneur dont l’adresse IP est **172.16.0.2**.

```powershell
Add-NetNatStaticMapping -NatName "Nat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
```

Cet exemple montre comment créer un mappage entre le port **82** de l’hôte du conteneur et le port **80** d’un conteneur dont l’adresse IP est **172.16.0.3**.

```powershell
Add-NetNatStaticMapping -NatName "Nat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.3 -InternalPort 80 -ExternalPort 82
```
> Une règle de pare-feu correspondante est requise pour chaque port externe. Celle-ci peut être créée avec la commande `New-NetFirewallRule`. Pour plus d’informations, consultez les [informations de référence sur New-NetFirewallRule](https://technet.microsoft.com/en-us/library/jj554908.aspx).

Une fois le mappage de ports créé, une application de conteneurs est accessible via l’adresse IP de l’hôte du conteneur (physique ou virtuel) et un port externe exposé. Par exemple, le schéma ci-dessous décrit une configuration NAT avec une demande ciblant le port externe **82** de l’hôte du conteneur. Sur la base du mappage de ports, cette demande renvoie l’application hébergée dans le conteneur 2.

![](./media/nat1.png)

Vue de la demande à partir d’un navigateur Internet.

![](./media/portmapping.png)

## Mode mise en réseau transparente

**Mise en réseau transparente** : cette configuration se compose d’un commutateur réseau externe. Dans cette configuration, chaque conteneur reçoit une adresse IP d’un serveur DHCP, et est accessible via cette adresse IP. L’avantage est qu’aucune table de mappage de ports n’est conservée.

### Configuration de l’hôte

Pour configurer le conteneur système afin que des conteneurs puissent recevoir une adresse IP d’un serveur DHCP, créez un commutateur virtuel connecté à une carte réseau physique ou virtuelle.

L’exemple suivant crée un commutateur virtuel portant le nom DHCP, à l’aide d’une carte réseau nommée Ethernet.

```powershell
New-VMSwitch -Name DHCP -NetAdapterName Ethernet
```

Si l’hôte du conteneur est lui-même un ordinateur virtuel, vous devez activer MacAddressSpoofing sur la carte réseau utilisée avec le commutateur du conteneur. L’exemple suivant montre comment accomplir cela sur une machine virtuelle nommée `DemoVm`.

```powershell
Get-VMNetworkAdapter -VMName DemoVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
Le commutateur virtuel externe peut maintenant être connecté à un conteneur, permettant à celui-ci de recevoir une adresse IP d’un serveur DHCP. Dans cette configuration, les applications hébergées à l’intérieur du conteneur sont accessibles à l’adresse IP assignée au conteneur.

## Configuration du Docker

Lors du démarrage du démon Docker, vous pouvez sélectionner un pont réseau. Lors de l’exécution du Docker sur Windows, il s’agit du commutateur virtuel externe ou NAT. L’exemple suivant montre comment démarrer le démon Docker en spécifiant un commutateur virtuel nommé `Virtual Switch`.

```powershell
Docker daemon -D -b “Virtual Switch” -H 0.0.0.0:2375
```

Si vous avez déployé l’hôte du conteneur et le Docker à l’aide des scripts fournis dans Windows Container Quick Starts, un commutateur virtuel interne est créé avec un type de NAT, et un service Docker est créé et préconfiguré pour utiliser ce commutateur. Pour modifier le commutateur virtuel que le service Docker utilise, vous devez arrêter le service Docker, modifier un fichier de configuration, puis redémarrer le service.

Pour arrêter le service, utilisez la commande PowerShell suivante :

```powershell
Stop-Service docker
```

Le fichier de configuration se trouve dans `C:\programdata\docker\runDockerDaemon.cmd`. Modifiez la ligne suivante, en remplaçant `Virtual Switch` par le nom du commutateur virtuel que le service Docker doit utiliser.

```powershell
docker daemon -D -b “New Switch Name"
```
Enfin, démarrez le service.

```powershell
Start-Service docker
```

## Gérer les cartes réseau

Quelle que soit la configuration réseau (NAT ou Transparente), plusieurs commandes PowerShell sont disponibles pour la gestion des cartes réseau de conteneur et les connexions de commutateur virtuel.

Gestion d’une carte réseau de conteneurs

- Add-ContainerNetworkAdapter : ajoute une carte réseau à un conteneur.
- Set-ContainerNetworkAdapter : modifie la carte réseau d’un conteneur.
- Remove-ContainerNetworkAdapter : supprime la carte réseau d’un conteneur.
- Get-ContainerNetworkAdapter : renvoie des données sur la carte réseau d’un conteneur.

Gérez la connexion entre une carte réseau de conteneurs et un commutateur virtuel.

- Connect-ContainerNetworkAdapter : connecte un conteneur à un commutateur virtuel.
- Disconnect-ContainerNetworkAdapter : déconnecte un conteneur d’un commutateur virtuel.

Pour plus d’informations sur chacune de ces commandes, consultez les [informations de référence PowerShell sur le conteneur](https://technet.microsoft.com/en-us/library/mt433069.aspx).






<!--HONumber=Feb16_HO4-->



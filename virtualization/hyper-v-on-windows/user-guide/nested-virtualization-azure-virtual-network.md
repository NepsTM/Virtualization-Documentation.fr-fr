---
title: Configuration d’ordinateurs virtuels imbriqués pour communiquer directement avec les ressources d’un réseau virtuel Azure
description: Virtualisation imbriquée
keywords: Windows 10, Hyper-v, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: 2f1c6a124ba4f2f9d199d3cc5bb38c9082f72b3d
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681139"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Configurer des ordinateurs virtuels imbriqués pour communiquer avec des ressources dans un réseau virtuel Azure

Pour plus d’informations sur le déploiement et la configuration des machines virtuelles imbriquées dans Azure, vous devez accéder à ces VM par le biais d’un commutateur NAT. Cela présente plusieurs limitations:

1. Les VM imbriquées ne peuvent pas accéder aux ressources locales ou à partir d’un réseau virtuel Azure.
2. Les ressources locales ou les ressources dans Azure peuvent uniquement accéder aux VM imbriquées par le biais d’un NAT, ce qui signifie que plusieurs invités ne peuvent pas partager le même port.

Ce document traite du déploiement par le biais du protocole RRAS, des itinéraires définis par l’utilisateur, du sous-réseau dédié à la traduction d’adresses réseau sortante pour autoriser l’accès à Internet invité et un espace d’adressage «flottant» pour permettre aux VM imbriquées de se comporter et de communiquer comme n’importe quelle autre machine virtuelle. déployé directement sur un VNet dans Azure.

Avant de démarrer ce guide, procédez comme suit:

1. Lisez les [instructions fournies ici](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization) sur virtualisation imbriquée.
2. Lire l’ensemble de cet article avant de l’implémenter.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>Présentation de haut niveau de ce que nous faisons et pourquoi
* Nous allons créer un ordinateur virtuel doté d’imbrication doté de deux cartes réseau. 
* Une carte réseau sera utilisée pour fournir nos ordinateurs virtuels imbriqués avec un accès Internet via tar et l’autre carte réseau sera utilisée pour acheminer le trafic de notre commutateur interne vers des ressources extérieures à l’hyperviseur. Chaque carte réseau doit faire partir d’un domaine de routage différent, c’est-à-dire d’un sous-réseau différent.
* Cela signifie que nous aurons besoin d’un réseau virtuel à trois sous-réseaux minimum. Une pour tar, une pour le routage sur réseau local et une autre qui n’est pas utilisée, mais qui est «réservée» pour nos VM imbriquées. Les noms que nous utilisons pour les sous-réseaux dans ce document sont «NAT», «Hyper-V-LAN» et «fantômes».
* La taille de ces sous-réseaux est à votre discrétion, mais il existe quelques éléments à prendre en considération. La taille des sous-réseaux «fantômes» détermine le nombre de systèmes IPs pour vos ordinateurs virtuels imbriqués. Par ailleurs, la taille des sous-réseaux «tar» et «Hyper-V-LAN» détermine le nombre de systèmes IPs pour les hyperviseurs. Par conséquent, il est possible de créer des sous-réseaux vraiment petits si vous envisagez de disposer d’un ou de deux hyperviseurs.
* Arrière-plan: les ordinateurs virtuels imbriqués ne recevront pas le protocole DHCP du VNet auquel leur hôte est connecté, même si vous configurez un commutateur interne ou externe. 
  * Cela signifie que l’hôte Hyper-V doit fournir le protocole DHCP.
* L’hôte Hyper-V ne connaît pas les baux actuellement attribués sur le VNet, donc pour éviter une situation dans laquelle l’hôte affecte une adresse IP déjà en cours, nous devons attribuer un bloc d’IPs pour une utilisation seule de l’hôte Hyper-V. Cela nous permettra d’éviter un scénario IP en double.
  * Le bloc de prévention des intrusions que nous choisis correspond à un sous-réseau au sein de la même VNet que votre Hyper-V.
  * La raison pour laquelle nous voulons qu’elle corresponde à un sous-réseau existant consiste à gérer les annonces BGP sur une ExpressRoute. Si nous venons de créer une plage d’adresses IP pour l’hôte Hyper-V à utiliser, nous devons créer une série d’itinéraires statiques pour permettre aux clients sur-locaux de communiquer avec les VM imbriquées. Cela signifie qu’il n’est pas nécessaire d’avoir besoin d’une plage d’adresses IP pour les VM imbriquées, puis de créer tous les itinéraires nécessaires pour diriger les clients vers l’hôte Hyper-V de cette plage.
* Nous allons créer un commutateur interne dans Hyper-V, puis nous affecterons l’interface nouvellement créée à une adresse IP dans une plage que nous avons définie pour le protocole DHCP. Cette adresse IP devient la passerelle par défaut de nos VM imbriquées et sera utilisée pour le routage entre le commutateur interne et la carte réseau de l’hôte connecté à notre VNet.
* Nous allons installer le rôle routage et accès distant sur l’hôte, ce qui transforme votre hôte en routeur.  Cela est nécessaire pour autoriser la communication entre les ressources extérieures à l’hôte et nos VM imbriquées.
* Nous allons savoir aux autres ressources comment accéder à ces VM imbriquées. Cela nécessite que nous puissions créer une table d’itinéraires définie par l’utilisateur qui contient un itinéraire statique pour la plage d’adresses IP dans laquelle résident les ordinateurs virtuels imbriqués. Ce routage statique pointe sur l’adresse IP de Hyper-V.
* Vous pouvez ensuite placer ce UDR sur le sous-réseau des passerelles de telle sorte que les clients qui se trouvent sur site sachent comment joindre nos VM imbriquées.
* Ce UDR est également placé sur tout autre sous-réseau d’Azure, qui nécessite une connectivité aux ordinateurs virtuels imbriqués.
* Pour plusieurs hôtes Hyper-V, vous créez des sous-réseaux supplémentaires «flottants» et ajoutez un itinéraire statique supplémentaire à UDR.
* Lorsque vous désaffectez un hôte Hyper-V, vous devez supprimer/réutiliser le sous-réseau «flottant» et supprimer l’itinéraire statique de notre UDR, ou s’il s’agit du dernier hôte Hyper-V, supprimez UDR.

## <a name="creating-the-host"></a>Création de l’hôte

J’ai une brillance sur les valeurs de configuration qui appartiennent à des préférences personnelles, telles que le nom de l’ordinateur virtuel, le groupe de ressources, etc.

1. Accédez à portal.azure.com
2. Cliquez sur «créer une ressource» dans le coin supérieur gauche
3. Sélectionnez «Windows Server 2016 VM» dans la colonne populaires
4. Dans l’onglet «notions de base», n’hésitez pas à sélectionner une taille d’ordinateur virtuel compatible avec la virtualisation imbriquée.
5. Accéder à l’onglet «mise en réseau»
6. Créer un réseau virtuel avec la configuration suivante
    * Espace d’adressage de VNet: 10.0.0.0/22
    * Sous-réseau 1
        * Nom: NAT
        * Espace d’adressage: 10.0.0.0/24
    * Sous-réseau 2
        * Nom: Hyper-V-LAN
        * Espace d’adressage: 10.0.1.0/24
    * Sous-réseau 3
        * Nom: Ghosté
        * Espace d’adressage: 10.0.2.0/24
    * Sous-réseau 4
        * Nom: Azure-VMs
        * Espace d’adressage: 10.0.3.0/24
7. Vérifiez que vous avez sélectionné le sous-réseau NAT pour l’ordinateur virtuel
8. Accédez à «révision + création» et sélectionnez «créer».

## <a name="create-the-second-network-interface"></a>Créer la deuxième interface réseau
1. Après la configuration de l’ordinateur virtuel, accédez-y au portail Azure.
2. Arrêter l’ordinateur virtuel
3. Une fois arrêté, accédez à «mise en réseau» sous paramètres
4. "Attacher une interface réseau"
5. "Créer une interface réseau"
6. Donnez-lui un nom (peu importe votre nom, mais veillez à le rappeler)
7. Sélectionner «Hyper-V-LAN» pour le sous-réseau
8. Assurez-vous de sélectionner le groupe de ressources dans lequel réside votre hôte.
9. Creat
10. Vous revenez à l’écran précédent, vous devez sélectionner l’interface réseau que vous venez de créer, puis sélectionner "OK".
11. Revenez au volet «vue d’ensemble» et redémarrez votre machine virtuelle une fois l’action précédente terminée.
12. Naviguez jusqu’à la deuxième carte réseau que nous venons de créer, vous pouvez la Rechercher dans le groupe de ressources que vous avez sélectionné précédemment.
13. Accédez à «configurations IP» et activez/désactivez l’option «transfert IP» vers «activé», puis enregistrez la modification.

## <a name="setting-up-hyper-v"></a>Configuration de Hyper-V
1. Distant dans votre hôte
2. Ouvrir une invite PowerShell avec élévation de privilèges
3. Exécutez la commande suivante `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`
4. Cette opération va redémarrer l’hôte.
5. Reconnectez-vous à l’hôte pour poursuivre la configuration.

## <a name="creating-our-virtual-switch"></a>Création de notre commutateur virtuel

1. Ouvrez PowerShell en mode administratif.
2. Créer un commutateur interne: `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. Assignez l’interface nouvellement créée par adresse IP: `New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>Installer et configurer le protocole DHCP

*De nombreux utilisateurs ne manquez pas ce composant quand ils essaient d’accéder à la virtualisation imbriquée pour la première fois. À la différence de l’emplacement local où vos ordinateurs virtuels invités recevront le protocole DHCP du réseau sur lequel réside votre hôte, les VM imbriquées dans Azure doivent être fournies via le protocole DHCP sur l’hôte sur lequel elles s’exécutent. Comme vous le souhaitez, vous devez affecter de manière statique une adresse IP à chaque VM imbriquée, qui n’est pas évolutive.*

1. Installez le rôle DHCP: `Install-WindowsFeature DHCP -IncludeManagementTools`
2. Créer l’étendue DHCP: `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`
3. Configurer les options DNS et de passerelle par défaut pour l’étendue: `Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`
    * Veillez à entrer un serveur DNS valide si vous souhaitez que la résolution de nom fonctionne. Le cas échéant, j’utilise le [DNS récurrent d’Azure](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances).

## <a name="installing-remote-access"></a>Installation de l’accès à distance

1. Ouvrez le gestionnaire de serveur, puis sélectionnez Ajouter des rôles et fonctionnalités.
2. Sélectionnez «suivant» jusqu’à atteindre «rôles serveur».
3. Activez l’option «accès à distance» et cliquez sur «suivant» jusqu’à ce que vous atteigniez «services de rôle».
4. Cochez la case «routage», sélectionnez «Ajouter des fonctionnalités», puis cliquez sur «suivant», puis sur «installer». Terminez l’Assistant et attendez la fin de l’installation.

## <a name="configuring-remote-access"></a>Configuration de l’accès à distance

1. Ouvrez le gestionnaire de serveur et sélectionnez «Outils», puis sélectionnez «routage et accès distant».
2. Sur le côté gauche de la fenêtre de routage et de gestion de l’accès à distance, vous verrez une icône contenant le nom de votre serveur en regard de celui-ci, cliquez avec le bouton droit sur ce bouton et sélectionnez «configurer et activer le service de routage et accès distant».
3. Dans l’Assistant, sélectionnez «suivant», cliquez sur le bouton radial pour «configuration personnalisée», puis sélectionnez «suivant».
4. Vérifiez «NAT» et «routage LAN», puis sélectionnez «suivant», puis «terminer». Si vous êtes invité à démarrer le service, procédez ainsi.
5. À présent, accédez au nœud «IPv4» et développez-le afin que le nœud «NAT» soit disponible.
6. Cliquez avec le bouton droit sur «traducteur», puis sélectionnez «nouvelle interface». et sélectionnez «Ethernet», il s’agit de votre première carte réseau avec l’adresse IP «10.0.0.4».
7. À présent, nous devons créer des itinéraires statiques pour forcer le trafic LAN vers la deuxième carte réseau. Pour ce faire, accédez au nœud «itinéraires statiques» sous «IPv4».
8. Nous allons ensuite créer les itinéraires suivants.
    * Itinéraire 1
        * Interface: Ethernet
        * Destination: 10.0.0.0
        * Masque réseau: 255.255.255.0
        * Passerelle: 10.0.0.1
        * Métrique: 256
        * Remarque: nous avons mis en place cette section pour permettre à la carte réseau principale de répondre au trafic destiné à son interface. Si ce n’est pas le cas, l’itinéraire suivant entraînerait du trafic pour la carte réseau (NIC) 1 to go. Cela crée un itinéraire asymétrique. 10.0.0.1 est l’adresse IP attribuée par Azure au sous-réseau NAT. Azure utilise la première adresse IP disponible dans une plage comme passerelle par défaut. Par conséquent, si vous avez utilisé 192.168.0.0/24 pour votre sous-réseau NAT, la passerelle sera 192.168.0.1. Dans le routage de l’itinéraire WINS plus spécifique, cela signifie que cette route va remplacer l’itinéraire ci-dessous.

    * Route 2
        * Interface: Ethernet 2
        * Destination: 10.0.0.0
        * Masque réseau: 255.255.252.0
        * Passerelle: 10.0.1.1
        * Métrique: 256
        * Remarque: il s’agit d’une voie pour le trafic destiné à notre réseau Azure VNet. Le trafic est alors épuisé. Vous devez ajouter des itinéraires supplémentaires pour les autres plages auxquelles vous souhaitez que vos ordinateurs virtuels imbriqués y aient accès. Par conséquent, si vous êtes sur un réseau locaux, vous devez disposer d’un autre itinéraire pour envoyer le trafic vers la deuxième NIC de notre Hypervisor.

## <a name="creating-a-route-table-within-azure"></a>Création d’une table d’itinéraires dans Azure

Consultez [cet article](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-portal) pour en savoir plus sur la création et la gestion d’itinéraires dans Azure.

1. Accédez à https://portal.azure.com.
2. Dans le coin supérieur gauche, sélectionnez «créer une ressource».
3. Dans le champ de recherche, tapez «table d’itinéraires», puis appuyez sur entrée.
4. Le résultat le plus élevé sera la table route, sélectionnez cette option, puis sélectionnez «créer».
5. Nommez la table d’itinéraires, dans mon cas, je le appelle «itinéraires-pour-nestd-VM».
6. Vérifiez que vous avez bien sélectionné l’abonnement dans lequel résident vos hôtes Hyper-V.
7. Créez un groupe de ressources ou sélectionnez-en un et assurez-vous que la région dans laquelle vous créez la table d’itinéraires est la même région que celle de votre hôte Hyper-V.
8. Sélectionnez «créer».

## <a name="configuring-the-route-table"></a>Configuration de la table d’itinéraires

1. Accédez à la table d’itinéraires que nous venons de créer. Pour cela, vous pouvez rechercher le nom de la table d’itinéraires à partir de la barre de recherche dans le centre supérieur du portail.
2. Après avoir sélectionné la table d’itinéraires, sélectionnez itinéraires dans la Blade.
3. Sélectionnez «Ajouter».
4. Donnez un nom à votre itinéraire, j’ai été suivi de «imbriqué-VMs».
5. Pour l’entrée de préfixe d’adresse, entrez la plage d’adresses IP de votre sous-réseau «flottant». Le cas échéant, il s’agit de 10.0.2.0/24.
6. Pour «type de tronçon suivant», sélectionnez «appareil virtuel», puis entrez l’adresse IP de la deuxième carte réseau d’hébergement Hyper-V, qui sera 10.0.1.4, puis sélectionnez «OK».
7. À présent, à partir de la Blade, sélectionnez «sous-réseaux», qui se trouve juste en dessous de «routes».
8. Sélectionnez "Associate", puis sélectionnez notre VNet "nestable" et sélectionnez le sous-réseau "Azure-VMs", puis sélectionnez "OK".
9. Procédez de la même manière pour le sous-réseau sur lequel se trouve notre hôte Hyper-V, ainsi que pour d’autres sous-réseaux qui doivent accéder aux VM imbriquées. S’il est connecté 

# <a name="end-state-configuration-reference"></a>Référence de configuration de l’état de fin
L’environnement de ce guide est soumis aux configurations suivantes. Cette section est inteded à utiliser comme référence.

1. Informations du réseau virtuel Azure
    * Configuration de haut niveau de VNet.
        * Nom: imbriqué-sympa
        * Espace d’adressage: 10.0.0.0/22
        * Remarque: il s’agit de quatre sous-réseaux. De plus, ces plages ne sont pas définies en pierres. N’hésitez pas à traiter votre environnement comme vous le souhaitez. 

    * Première configuration de niveau élevé de sous-réseau.
        * Nom: NAT
        * Espace d’adressage: 10.0.0.0/24
        * Remarque: il s’agit de l’emplacement de la carte réseau principal de votre hôte Hyper-V. Il sera utilisé pour gérer le protocole NAT sortant pour les ordinateurs virtuels imbriqués. Il s’agit de la passerelle vers Internet pour vos VM imbriquées.

    * Deuxième configuration de niveau de sous-réseau.
        * Nom: Hyper-V-LAN
        * Espace d’adressage: 10.0.1.0/24
        * Remarque: notre hôte Hyper-V dispose d’une deuxième carte réseau qui sera utilisée pour gérer le routage entre les machines virtuelles imbriquées et les ressources non Internet externes à l’hôte Hyper-V.

    * Configuration de troisième sous-réseau de niveau supérieur.
        * Nom: Ghosté
        * Espace d’adressage: 10.0.2.0/24
        * Remarque: il s’agit d’un sous-réseau «flottant». L’espace d’adressage sera consommé par nos VM imbriquées et il existe pour gérer les annonces d’itinéraire sur site. Aucun VMs ne sera déployé dans ce sous-réseau.

    * Quatrième configuration de niveau de sous-réseau haut.
        * Nom: Azure-VMs
        * Espace d’adressage: 10.0.3.0/24
        * Remarque: sous-réseau contenant Azure VM.

1. Notre hôte Hyper-V est doté des configurations NIC suivantes.
    * RÉSEAU principal 
        * Adresse IP: 10.0.0.4
        * Masque de sous-réseau: 255.255.255.0
        * Passerelle par défaut: 10.0.0.1
        * DNS: configuré pour le protocole DHCP
        * Transfert IP activé: non

    * Carte réseau secondaire
        * Adresse IP: 10.0.1.4
        * Masque de sous-réseau: 255.255.255.0
        * Passerelle par défaut: vide
        * DNS: configuré pour le protocole DHCP
        * Transfert IP activé: Oui

    * Carte réseau créée par Hyper-V pour commutateur virtuel interne
        * Adresse IP: 10.0.2.1
        * Masque de sous-réseau: 255.255.255.0
        * Passerelle par défaut: vide

3. Notre table d’itinéraires aura une seule règle.
    * Règle 1
        * Nom: imbriqué-VMs
        * Destination: 10.0.2.0/24
        * Tronçon suivant: appareil virtuel-10.0.1.4

## <a name="conclusion"></a>Conclusion

Vous devriez maintenant être en mesure de déployer une machine virtuelle (même une VM 32 bits) sur votre hôte Hyper-V et de l’rendre accessible depuis local et dans Azure.

---
title: Configurer des machines virtuelles imbriquées pour qu'elles communiquent directement avec les ressources d'un réseau virtuel Azure
description: Virtualisation imbriquée
keywords: Windows 10, Hyper-V, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: b287ea091ad283ddf57727f315c7086865375ce7
ms.sourcegitcommit: e9b3c9dcf7b5c9b9222edbc344764fb038529739
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/03/2020
ms.locfileid: "84334099"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Configurer des machines virtuelles imbriquées pour qu'elles communiquent avec les ressources d'un réseau virtuel Azure

Les instructions d'origine relatives au déploiement et à la configuration de machines virtuelles imbriquées dans Azure exigent que vous accédiez à ces machines par le biais d'un commutateur NAT. Cela se traduit par les contraintes suivantes :

1. Les machines virtuelles imbriquées n'ont pas accès aux ressources disponibles localement ou sur un réseau virtuel Azure.
2. Les ressources disponibles localement ou dans Azure n'ont accès aux machines virtuelles imbriquées que par le biais d'un NAT, ce qui signifie qu'un même port ne peut pas être partagé par plusieurs invités.

Ce document décrit un déploiement dans lequel nous utilisons le service RRAS, des itinéraires définis par l'utilisateur, un sous-réseau dédié au NAT sortant pour autoriser l'accès Internet invité, et un espace d'adressage « flottant » pour permettre aux machines virtuelles imbriquées de se comporter et de communiquer comme n'importe quelle autre machine virtuelle directement déployée sur un réseau virtuel Azure.

Avant d'entamer ce guide :

1. Lisez les [instructions disponibles ici](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization) sur la virtualisation imbriquée.
2. Lisez l'intégralité de cet article avant l'implémentation.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>Présentation
* Nous allons créer une machine virtuelle imbricable dotée de deux cartes réseau. 
* Une carte réseau sera utilisée pour fournir un accès Internet à nos machines virtuelles imbriquées via NAT et l'autre pour acheminer le trafic de notre commutateur interne vers des ressources externes à l'hyperviseur. Chaque carte réseau devra se trouver dans un domaine de routage distinct, c'est-à-dire dans un sous-réseau différent.
* Cela signifie que nous aurons besoin d'un réseau virtuel comportant au moins trois sous-réseaux. Un pour NAT, un pour le routage LAN, et un autre qui ne sera pas utilisé mais qui sera « réservé » pour nos machines virtuelles imbriquées. Les noms des sous-réseaux utilisés dans ce document sont « NAT », « Hyper-V-LAN » et « Ghosted ».
* La taille de ces sous-réseaux est à votre entière discrétion, mais certaines considérations sont à prendre en compte. La taille des sous-réseaux « Ghosted » détermine le nombre d'adresses IP dont vous disposez pour vos machines virtuelles imbriquées. De même, la taille des sous-réseaux « NAT » et « Hyper-V-LAN » détermine le nombre d'adresses IP dont vous disposez pour les hyperviseurs. Techniquement, vous pouvez donc créer des sous-réseaux de petite taille si vous prévoyiez de n'utiliser qu'un ou deux hyperviseurs.
* Background : les machines virtuelles imbriquées ne recevront PAS le protocole DHCP du réseau virtuel auquel leur hôte est connecté, même si vous configurez un commutateur interne ou externe. 
  * Cela signifie que l'hôte Hyper-V doit fournir le protocole DHCP.
* L'hôte Hyper-V n'a pas connaissance des baux actuellement attribués sur le réseau virtuel. Par conséquent, pour éviter que l'hôte attribue une adresse IP qui existe déjà, nous devons allouer un bloc d'adresses IP que seul l'hôte Hyper-V pourra utiliser. Nous éviterons ainsi les adresses IP en double.
  * Le bloc d'adresses IP choisi correspondra à un sous-réseau du même réseau virtuel que votre Hyper-V.
  * Il est impératif qu'il corresponde à un sous-réseau existant pour pouvoir gérer les annonces BGP sur une instance d'ExpressRoute. Si nous venons de créer une plage d'adresses IP pour l'hôte Hyper-V à utiliser, nous devons créer une série d'itinéraires statiques afin de permettre aux clients locaux de communiquer avec les machines virtuelles imbriquées. Il ne s'agit donc pas d'une exigence stricte puisque vous POUVEZ créer une plage d'adresses IP pour les machines virtuelles imbriquées, puis créer tous les itinéraires nécessaires afin de diriger les clients vers l'hôte Hyper-V correspondant à cette plage.
* Nous créerons un commutateur interne sur l'hôte Hyper-V, puis nous attribuerons à l'interface nouvellement créée une adresse IP située dans une plage réservée au protocole DHCP. Cette adresse IP deviendra la passerelle par défaut de nos machines virtuelles imbriquées et sera utilisée pour le routage entre le commutateur interne et la carte réseau de l'hôte connecté à notre réseau virtuel.
* Nous installerons le rôle Routage et accès à distance sur l'hôte, ce qui transformera notre hôte en routeur.  Cette opération est nécessaire pour permettre la communication entre les ressources externes à l'hôte et nos machines virtuelles imbriquées.
* Nous indiquerons aux autres ressources comment accéder à ces machines virtuelles imbriquées. Cela nécessite la création d'une table Itinéraire défini par l'utilisateur qui contient un itinéraire statique pour la plage d'adresses IP dans laquelle résident les machines virtuelles imbriquées. Cet itinéraire statique pointera vers l'adresse IP de l'hôte Hyper-V.
* Vous placerez ensuite cette table Itinéraire défini par l'utilisateur sur le sous-réseau de la passerelle afin que les clients issus d'un site local sachent comment accéder à nos machines virtuelles imbriquées.
* Vous placerez également cette table sur tout autre sous-réseau d'Azure qui doit être connecté aux machines virtuelles imbriquées.
* En présence de plusieurs hôtes Hyper-V, vous devez créer des sous-réseaux « flottants » supplémentaires et ajouter un itinéraire statique supplémentaire à la table Itinéraire défini par l'utilisateur.
* Lorsque vous désactivez un hôte Hyper-V, vous supprimez/réaffectez notre sous-réseau « flottant » et supprimez cet itinéraire statique de notre table Itinéraire défini par l'utilisateur, ou s'il s'agit du dernier hôte Hyper-V, vous supprimez complètement la table.

## <a name="creating-the-host"></a>Création de l'hôte

Je ferai abstraction de toutes les valeurs de configuration basées sur les préférences personnelles, comme le nom de la machine virtuelle, le groupe de ressources et ainsi de suite.

1. Accédez à portal.azure.com.
2. Cliquez sur « Créer une ressource » en haut à gauche.
3. Sélectionnez « Machine virtuelle Windows Server 2016 » dans la colonne Populaires.
4. Sous l'onglet « Informations de base », veillez à sélectionner une taille de machine virtuelle qui prend en charge la virtualisation imbriquée.
5. Accédez à l'onglet « Mise en réseau ».
6. Créez un réseau virtuel en appliquant la configuration suivante :
    * Espace d'adressage du réseau virtuel : 10.0.0.0/22
    * Sous-réseau 1
        * Nom : NAT
        * Espace d'adressage : 10.0.0.0/24
    * Sous-réseau 2
        * Nom : Hyper-V-LAN
        * Espace d'adressage : 10.0.1.0/24
    * Sous-réseau 3
        * Nom : Ghosted
        * Espace d'adressage : 10.0.2.0/24
    * Sous-réseau 4
        * Nom : Azure-VMs
        * Espace d'adressage : 10.0.3.0/24
7. Vérifiez que vous avez sélectionné le sous-réseau NAT pour la machine virtuelle.
8. Accédez à « Examiner et créer », puis sélectionnez « Créer ».

## <a name="create-the-second-network-interface"></a>Création de la deuxième interface réseau
1. Au terme de l'approvisionnement de la machine virtuelle, accédez-y sur le portail Azure.
2. Arrêtez la machine virtuelle.
3. Une fois la machine virtuelle arrêtée, accédez à « Mise en réseau » sous Paramètres.
4. Sélectionnez « Attacher l'interface réseau ».
5. Sélectionnez « Créer l'interface réseau ».
6. Donnez-lui un nom (peu importe lequel, mais mémorisez-le).
7. Sélectionnez « Hyper-V-LAN » pour le sous-réseau.
8. Veillez à sélectionner le groupe de ressources dans lequel votre hôte réside.
9. Sélectionnez « Créer ».
10. Vous revenez alors à l'écran précédent. Sélectionnez l'interface réseau nouvellement créée, puis sélectionnez « OK ».
11. Revenez au volet « Vue d'ensemble ». Au terme de l'action précédente, redémarrez votre machine virtuelle.
12. Accédez à la deuxième carte réseau que nous venons de créer. Vous la trouverez dans le groupe de ressources que vous avez sélectionné précédemment.
13. Accédez à « Configurations IP », définissez « Transfert IP » sur « Activé », puis enregistrez la modification.

## <a name="setting-up-hyper-v"></a>Configuration de l'hôte Hyper-V
1. Connectez-vous à distance à votre hôte.
2. Ouvrez une invite PowerShell avec élévation de privilèges.
3. Exécutez la commande suivante : `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`
4. L'hôte redémarre.
5. Reconnectez-vous à l'hôte pour poursuivre la configuration.

## <a name="creating-our-virtual-switch"></a>Création de notre commutateur virtuel

1. Ouvrez PowerShell en mode Administrateur.
2. Créez un commutateur interne : `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. Attribuez une adresse IP à l'interface nouvellement créée : `New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>Installation et configuration du protocole DHCP

*Ce composant manque à beaucoup d'utilisateurs lorsqu'ils essaient d'utiliser la virtualisation imbriquée pour la première fois. Contrairement à ce qui se passe localement, où vos machines virtuelles invitées reçoivent le protocole DHCP du réseau sur lequel votre hôte réside, les machines virtuelles imbriquées dans Azure doivent recevoir le protocole DHCP via l'hôte sur lequel elles s'exécutent. Sinon, vous devez attribuer statiquement une adresse IP à chaque machine virtuelle imbriquée, qui n'est pas évolutif.*

1. Installez le rôle DHCP : `Install-WindowsFeature DHCP -IncludeManagementTools`
2. Créez l'étendue DHCP : `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`
3. Configurez les options DNS et Passerelle par défaut pour l'étendue : `Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`
    * Veillez à entrer un serveur DNS valide si vous souhaitez que la résolution de noms fonctionne. Dans ce cas, j'utilise [DNS récursif d'Azure](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances).

## <a name="installing-remote-access"></a>Installation de l'accès à distance

1. Ouvrez le Gestionnaire de serveur et sélectionnez « Ajouter des rôles et des fonctionnalités ».
2. Sélectionnez « Suivant » jusqu'à ce que vous accédiez à « Rôles de serveur ».
3. Cochez « Accès à distance », puis cliquez sur « Suivant » jusqu'à ce que vous accédiez à « Services de rôle ».
4. Cochez « Routage », sélectionnez « Ajouter des fonctionnalités », puis sélectionnez « Suivant » et « Installer ». Terminez l'Assistant et attendez la fin de l'installation.

## <a name="configuring-remote-access"></a>Configuration de l'accès à distance

1. Ouvrez Gestionnaire de serveur, sélectionnez « Outils », puis sélectionnez « Routage et accès à distance ».
2. Sur le côté gauche du panneau de gestion Routage et accès à distance, vous verrez une icône accompagnée du nom de vos serveurs. Cliquez sur cette icône avec le bouton droit et sélectionnez « Configurer et activer le routage et l'accès à distance ».
3. Dans l’Assistant, sélectionnez « Suivant », « Configuration personnalisée », puis « Suivant ».
4. Cochez « NAT » et « Routage LAN », puis sélectionnez « Suivant » et « Terminer ». Si vous êtes invité à démarrer le service, faites-le.
5. À présent, accédez au nœud « IPv4 » et développez-le afin que le nœud « NAT » soit disponible.
6. Cliquez avec le bouton droit sur « NAT », sélectionnez « Nouvelle interface... », puis sélectionnez « Ethernet ». Il doit s'agir de votre première carte réseau avec l'adresse IP « 10.0.0.4 ». Sélectionnez ensuite « Interface publique », connectez-vous à Internet et activez NAT sur cette interface. 
7. Nous devons maintenant créer des itinéraires statiques pour forcer le trafic LAN sur la deuxième carte réseau. Pour ce faire, accédez au nœud « Itinéraires statiques » sous « IPv4 ».
8. Nous allons créer les itinéraires suivants.
    * Itinéraire 1
        * Interface : Ethernet
        * Destination : 10.0.0.0
        * Masque de réseau : 255.255.255.0
        * Passerelle : 10.0.0.1
        * Métrique : 256
        * Remarque : nous l'avons placé ici pour permettre à la carte réseau principale de répondre au trafic qui lui est destiné via sa propre interface. Sinon, l'itinéraire suivant acheminerait le trafic destiné à la carte réseau 1 vers la carte réseau 2. L'itinéraire serait alors asymétrique. 10.0.0.1 est l'adresse IP qu'Azure attribue au sous-réseau NAT. Azure utilise la première adresse IP disponible dans une plage comme passerelle par défaut. Par conséquent, si vous avez utilisé 192.168.0.0/24 pour votre sous-réseau NAT, la passerelle sera 192.168.0.1. En matière de routage, l'itinéraire le plus précis l'emporte, ce qui signifie que cet itinéraire remplacera l'itinéraire ci-dessous.

    * Itinéraire 2
        * Interface : Ethernet 2
        * Destination : 10.0.0.0
        * Masque de réseau : 255.255.252.0
        * Passerelle : 10.0.1.1
        * Métrique : 256
        * Remarque : il s'agit d'un itinéraire universel pour le trafic destiné à notre réseau virtuel Azure. Il forcera l'acheminement du trafic vers la deuxième carte réseau. Vous devez ajouter des itinéraires supplémentaires pour les autres plages auxquelles vous souhaitez que vos machines virtuelles imbriquées accèdent. Par conséquent, si votre réseau local est 172.16.0.0/22, vous pourrez utiliser un autre itinéraire pour acheminer ce trafic vers la deuxième carte réseau de notre hyperviseur.

## <a name="creating-a-route-table-within-azure"></a>Création d'une table d'itinéraires dans Azure

Pour plus d'informations sur la création et la gestion d'itinéraires dans Azure, consultez [cet article](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-portal).

1. Accédez à https://portal.azure.com.
2. Dans le coin supérieur gauche, sélectionnez « Créer une ressource ».
3. Dans le champ de recherche, entrez « Table d'itinéraires » et appuyez sur Entrée.
4. Le premier résultat est Table d'itinéraires. Sélectionnez-le, puis sélectionnez Créer.
5. Donnez un nom à la table d'itinéraires. Je l'ai ici nommée « Routes-for-nested-VMs ».
6. Veillez à sélectionner l'abonnement dans lequel vos hôtes Hyper-V résident.
7. Créez un groupe de ressources ou sélectionnez un groupe existant, et assurez-vous que la région dans laquelle vous créez la table d'itinéraires est la même que celle dans laquelle réside votre hôte Hyper-V.
8. Sélectionnez « Créer ».

## <a name="configuring-the-route-table"></a>Configuration de la table d'itinéraires

1. Accédez à la table d'itinéraires que nous venons de créer. Pour ce faire, recherchez son nom à partir de la barre de recherche située en haut et au centre du portail.
2. Une fois la table d'itinéraires sélectionnée, accédez à « Itinéraires » depuis le panneau.
3. Sélectionnez « Ajouter ».
4. Donnez un nom à votre itinéraire. Je l'ai ici appelé « Nested-VMs ».
5. Pour le préfixe de l'adresse, entrez la plage d'adresses IP de notre sous-réseau « flottant ». Dans ce cas, il s'agit de 10.0.2.0/24.
6. Dans « Type de tronçon suivant », sélectionnez « Appliance virtuelle », puis entrez l'adresse IP de la deuxième carte réseau des hôtes Hyper-V, à savoir 10.0.1.4, puis sélectionnez « OK ».
7. Dans le panneau, sélectionnez maintenant « Sous-réseaux », sous « Itinéraires ».
8. Sélectionnez successivement « Associer », notre réseau virtuel « Nested-Fun », le sous-réseau « Machines virtuelles Azure », puis « OK ».
9. Suivez le même processus pour le sous-réseau sur lequel notre hôte Hyper-V réside ainsi que pour tous les autres sous-réseaux qui doivent accéder aux machines virtuelles imbriquées. En cas de connexion. 

## <a name="end-state-configuration-reference"></a>Référence de configuration de l'état final

Les configurations suivantes s'appliquent à l'environnement décrit dans ce guide. Cette section est destinée à être utilisée comme référence.

1. Informations sur le réseau virtuel Azure.
    * Configuration générale du réseau virtuel
        * Nom : Nested-Fun
        * Espace d'adressage : 10.0.0.0/22
        * Remarque : il sera constitué de quatre sous-réseaux. En outre, ces plages ne sont pas figées. N'hésitez pas à configurer votre environnement comme vous le souhaitez. 

    * Configuration générale du premier sous-réseau.
        * Nom : NAT
        * Espace d'adressage : 10.0.0.0/24
        * Remarque : c'est là que réside la carte réseau principale de nos hôtes Hyper-V. Il sera utilisé pour gérer les NAT sortants des machines virtuelles imbriquées. Il s'agira de la passerelle d'accès à Internet de vos machines virtuelles imbriquées.

    * Configuration générale du deuxième sous-réseau.
        * Nom : Hyper-V-LAN
        * Espace d'adressage : 10.0.1.0/24
        * Remarque :  notre hôte Hyper-V disposera d'une deuxième carte réseau qui sera utilisée pour gérer le routage entre les machines virtuelles imbriquées et les ressources non Internet externes à l'hôte Hyper-V.

    * Configuration générale du troisième sous-réseau.
        * Nom : Ghosted
        * Espace d'adressage : 10.0.2.0/24
        * Remarque :  il s'agira d'un sous-réseau « flottant ». L'espace d'adressage sera utilisé par nos machines virtuelles imbriquées pour gérer les annonces de routage vers les sites locaux. Aucune machine virtuelle ne sera effectivement déployée sur ce sous-réseau.

    * Configuration générale du quatrième sous-réseau.
        * Nom : Azure-VMs
        * Espace d'adressage : 10.0.3.0/24
        * Remarque : sous-réseau contenant les machines virtuelles Azure.

1. Les configurations ci-dessous s'appliquent aux cartes réseau de notre hôte Hyper-V.
    * Carte réseau principale 
        * Adresse IP : 10.0.0.4
        * Masque de sous-réseau : 255.255.255.0
        * Passerelle par défaut : 10.0.0.1
        * DNS : configuré pour DHCP
        * Transfert IP activé : Non

    * Carte réseau secondaire
        * Adresse IP : 10.0.1.4
        * Masque de sous-réseau : 255.255.255.0
        * Passerelle par défaut : Vide
        * DNS : configuré pour DHCP
        * Transfert IP activé : Oui

    * Carte réseau créée par l'hôte Hyper-V pour le commutateur virtuel interne
        * Adresse IP : 10.0.2.1
        * Masque de sous-réseau : 255.255.255.0
        * Passerelle par défaut : Vide

3. Notre table d'itinéraires disposera d'une seule règle.
    * Règle 1
        * Nom : Nested-VMs
        * Destination : 10.0.2.0/24
        * Tronçon suivant : Appliance virtuelle - 10.0.1.4

## <a name="conclusion"></a>Conclusion

Vous devriez maintenant être en mesure de déployer une machine virtuelle (y compris une machine virtuelle 32 bits) sur votre hôte Hyper-V et de la rendre accessible à partir de sites locaux ainsi que dans Azure.

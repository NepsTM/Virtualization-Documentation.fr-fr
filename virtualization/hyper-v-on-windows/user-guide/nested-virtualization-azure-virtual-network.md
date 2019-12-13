---
title: Configuration des machines virtuelles imbriquées pour communiquer directement avec les ressources d’un réseau virtuel Azure
description: Virtualisation imbriquée
keywords: Windows 10, Hyper-v, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: efd180c458457da1cea6b379e21ba3a37083d15a
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910959"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Configurer des machines virtuelles imbriquées pour communiquer avec les ressources d’un réseau virtuel Azure

Les recommandations d’origine relatives au déploiement et à la configuration d’ordinateurs virtuels imbriqués dans Azure nécessitent l’accès à ces machines virtuelles via un commutateur NAT. Cela présente plusieurs limitations :

1. Les machines virtuelles imbriquées ne peuvent pas accéder aux ressources locales ou au sein d’un réseau virtuel Azure.
2. Les ressources locales ou les ressources dans Azure peuvent uniquement accéder aux machines virtuelles imbriquées via un NAT, ce qui signifie que plusieurs invités ne peuvent pas partager le même port.

Ce document vous guide tout au long d’un déploiement dans lequel nous utilisons le service RRAS, les itinéraires définis par l’utilisateur, un sous-réseau dédié à NAT sortant pour autoriser l’accès Internet invité et un espace d’adressage « flottant » pour permettre aux machines virtuelles imbriquées de se comporter et de communiquer comme n’importe quelle autre machine virtuelle. déployé directement sur un réseau virtuel dans Azure.

Avant de commencer ce guide, veuillez :

1. Lisez les [conseils fournis ici](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization) sur la virtualisation imbriquée.
2. Lisez l’intégralité de cet article avant l’implémentation.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>Présentation de haut niveau de ce que nous faisons et pourquoi
* Nous allons créer une machine virtuelle pouvant être imbriquée avec deux cartes réseau. 
* Une carte d’interface réseau sera utilisée pour fournir à nos machines virtuelles imbriquées un accès à Internet via NAT et l’autre NIC sera utilisée pour acheminer le trafic de notre commutateur interne vers les ressources externes à l’hyperviseur. Chaque carte réseau doit se trouver dans un autre domaine de routage, ce qui signifie qu’il s’agit d’un sous-réseau différent.
* Cela signifie que nous aurons besoin d’un réseau virtuel avec au moins trois sous-réseaux. Une pour NAT, une pour le routage LAN et une autre qui n’est pas utilisée, mais est « réservée » pour nos machines virtuelles imbriquées. Les noms que nous utilisons pour les sous-réseaux dans ce document sont, « NAT », « Hyper-V-LAN » et « fantôme ».
* La taille de ces sous-réseaux est à votre entière discrétion, mais certaines considérations sont à prendre en compte. La taille des sous-réseaux « fantômes » détermine le nombre d’adresses IP pour vos machines virtuelles imbriquées. En outre, la taille des sous-réseaux « NAT » et « Hyper-V-LAN » détermine le nombre d’adresses IP pour les hyperviseurs. Techniquement, vous pouvez créer des sous-réseaux vraiment petits si vous envisagez de disposer d’un ou de deux hyperviseurs.
* Arrière-plan : les machines virtuelles imbriquées ne recevront pas de protocole DHCP à partir du réseau virtuel auquel leur hôte est connecté, même si vous configurez un commutateur interne ou externe. 
  * Cela signifie que l’hôte Hyper-V doit fournir le protocole DHCP.
* L’hôte Hyper-V n’a pas connaissance des baux actuellement attribués sur le réseau virtuel. par conséquent, afin d’éviter une situation dans laquelle l’hôte attribue déjà une adresse IP existante, nous devons allouer un bloc d’adresses IP à utiliser uniquement par l’hôte Hyper-V. Cela nous permettra d’éviter un scénario d’adresse IP en double.
  * Le bloc d’adresses IP que vous choisissez correspond à un sous-réseau dans le même réseau virtuel que celui dans lequel se trouve Hyper-V.
  * La raison pour laquelle nous voulons que cela correspond à un sous-réseau existant consiste à gérer les publications BGP sur un ExpressRoute. Si nous venons de créer une plage d’adresses IP pour l’hôte Hyper-V à utiliser, nous devrions créer une série d’itinéraires statiques pour autoriser les clients local à communiquer avec les machines virtuelles imbriquées. Cela signifie qu’il ne s’agit pas d’une exigence matérielle, car vous pouvez créer une plage d’adresses IP pour les machines virtuelles imbriquées, puis créer tous les itinéraires nécessaires pour diriger les clients vers l’hôte Hyper-V pour cette plage.
* Nous allons créer un commutateur interne dans Hyper-V, puis affecter à l’interface nouvellement créée une adresse IP dans une plage réservée pour DHCP. Cette adresse IP deviendra la passerelle par défaut pour nos machines virtuelles imbriquées et sera utilisée pour le routage entre le commutateur interne et la carte réseau de l’ordinateur hôte connecté à notre réseau virtuel.
* Nous allons installer le rôle routage et accès distant sur l’ordinateur hôte, ce qui transformera notre hôte en routeur.  Cela est nécessaire pour permettre la communication entre les ressources externes à l’hôte et nos machines virtuelles imbriquées.
* Nous communiquerons à d’autres ressources comment accéder à ces machines virtuelles imbriquées. Cela nécessite la création d’une table de routage définie par l’utilisateur qui contient un itinéraire statique pour la plage d’adresses IP dans laquelle résident les machines virtuelles imbriquées. Cet itinéraire statique pointe vers l’adresse IP pour Hyper-V.
* Vous allez ensuite placer ce UDR sur le sous-réseau de passerelle afin que les clients provenant d’un site local sachent comment atteindre nos machines virtuelles imbriquées.
* Vous allez également placer ce UDR sur tout autre sous-réseau dans Azure qui nécessite une connexion aux machines virtuelles imbriquées.
* Pour plusieurs ordinateurs hôtes Hyper-V, vous devez créer des sous-réseaux « flottants » supplémentaires et ajouter un itinéraire statique supplémentaire au UDR.
* Lorsque vous désaffectez un hôte Hyper-V, vous supprimez/réaffectez le sous-réseau « flottant » et supprimez cet itinéraire statique de notre UDR, ou s’il s’agit du dernier ordinateur hôte Hyper-V, supprimez complètement le UDR.

## <a name="creating-the-host"></a>Création de l'hôte

Je vais vous conformer à toutes les valeurs de configuration qui sont des préférences personnelles, telles que le nom de la machine virtuelle, le groupe de ressources, etc.

1. Accédez à portal.azure.com
2. Cliquez sur « créer une ressource » en haut à gauche
3. Sélectionnez « Windows Server 2016 VM » dans la colonne populaires
4. Dans l’onglet « notions de base », veillez à sélectionner une taille de machine virtuelle capable d’effectuer une virtualisation imbriquée.
5. Passer à l’onglet « mise en réseau »
6. Créer un nouveau réseau virtuel avec la configuration suivante
    * Espace d’adressage du réseau virtuel : 10.0.0.0/22
    * Sous-réseau 1
        * Nom : NAT
        * Espace d’adressage : 10.0.0.0/24
    * Sous-réseau 2
        * Nom : Hyper-V-LAN
        * Espace d’adressage : 10.0.1.0/24
    * Sous-réseau 3
        * Nom : fantôme
        * Espace d’adressage : 10.0.2.0/24
    * Sous-réseau 4
        * Nom : Azure-machines virtuelles
        * Espace d’adressage : 10.0.3.0/24
7. Vérifiez que vous avez sélectionné le sous-réseau NAT pour la machine virtuelle.
8. Accédez à « examiner + créer », puis sélectionnez « créer ».

## <a name="create-the-second-network-interface"></a>Créer la deuxième interface réseau
1. Une fois l’approvisionnement de la machine virtuelle terminé, accédez-y dans le portail Azure.
2. Arrêtez la machine virtuelle.
3. Une fois arrêté, accédez à « mise en réseau » sous paramètres.
4. « Attacher l’interface réseau »
5. « Créer une interface réseau »
6. Donnez-lui un nom (peu importe ce que vous lui attribuez, mais veillez à le mémoriser)
7. Sélectionnez « Hyper-V-LAN » pour le sous-réseau
8. Veillez à sélectionner le groupe de ressources dans lequel votre hôte réside
9. Créés
10. Vous revenez à l’écran précédent, veillez à sélectionner l’interface réseau nouvellement créée, puis sélectionnez « OK ».
11. Revenez au volet « vue d’ensemble » et redémarrez votre machine virtuelle une fois l’action précédente terminée.
12. Accédez à la deuxième carte réseau que nous venons de créer, vous pouvez la trouver dans le groupe de ressources que vous avez sélectionné précédemment
13. Accédez à « configurations IP » et basculez « transfert IP » vers « activé », puis enregistrez la modification.

## <a name="setting-up-hyper-v"></a>Configuration d’Hyper-V
1. À distance dans votre hôte
2. Ouvrir une invite PowerShell avec élévation de privilèges
3. Exécutez la commande suivante : `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`
4. Cette opération redémarre l’ordinateur hôte
5. Reconnectez-vous à l’hôte pour poursuivre le programme d’installation

## <a name="creating-our-virtual-switch"></a>Création du commutateur virtuel

1. Ouvrez PowerShell en mode administratif.
2. Créer un commutateur interne : `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. Affectez à l’interface nouvellement créée une adresse IP : `New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>Installer et configurer DHCP

*De nombreux utilisateurs manquent ce composant lorsqu’ils essaient d’utiliser la virtualisation imbriquée. Contrairement à l’emplacement local où vos machines virtuelles invitées recevront DHCP à partir du réseau sur lequel votre hôte réside, les machines virtuelles imbriquées dans Azure doivent être fournies par le protocole DHCP via l’hôte sur lequel ils s’exécutent. Cela ou vous devez affecter statiquement une adresse IP à chaque machine virtuelle imbriquée, qui n’est pas évolutive.*

1. Installer le rôle DHCP : `Install-WindowsFeature DHCP -IncludeManagementTools`
2. Créer l’étendue DHCP : `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`
3. Configurez les options DNS et de passerelle par défaut pour l’étendue : `Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`
    * Veillez à entrer un serveur DNS valide si vous souhaitez que la résolution de noms fonctionne. Dans ce cas, j’utilise le [DNS récursif d’Azure](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances).

## <a name="installing-remote-access"></a>Installation de l’accès à distance

1. Ouvrez Gestionnaire de serveur et sélectionnez « Ajouter des rôles et des fonctionnalités ».
2. Sélectionnez « suivant » jusqu’à ce que vous obteniez « rôles de serveur ».
3. Cochez « accès à distance », puis cliquez sur « suivant » jusqu’à ce que vous obteniez « services de rôle ».
4. Cochez « routage », sélectionnez « Ajouter des fonctionnalités », puis sélectionnez « suivant » et « installer ». Terminez l’Assistant et attendez que l’installation se termine.

## <a name="configuring-remote-access"></a>Configuration de l’accès à distance

1. Ouvrez Gestionnaire de serveur et sélectionnez « Outils », puis sélectionnez « routage et accès distant ».
2. Sur le côté gauche du panneau de gestion routage et accès à distance, vous verrez une icône avec le nom de vos serveurs en regard de celle-ci, cliquez avec le bouton droit et sélectionnez « configurer et activer le routage et l’accès à distance ».
3. Dans l’Assistant, sélectionnez « suivant », vérifiez le bouton radial pour « configuration personnalisée », puis sélectionnez « suivant ».
4. Vérifiez « NAT » et « routage LAN », puis sélectionnez « suivant » et « terminer ». Si vous êtes invité à démarrer le service, faites-le.
5. À présent, accédez au nœud « IPv4 » et développez-le afin que le nœud « NAT » soit disponible.
6. Cliquez avec le bouton droit sur « NAT », puis sélectionnez « nouvelle interface... ». et sélectionnez « Ethernet ». il doit s’agir de votre première carte réseau avec l’adresse IP « 10.0.0.4 ».
7. Nous devons maintenant créer des itinéraires statiques pour forcer le trafic LAN sur la seconde carte réseau. Pour ce faire, accédez au nœud « itinéraires statiques » sous « IPv4 ».
8. Nous allons créer les itinéraires suivants.
    * Itinéraire 1
        * Interface : Ethernet
        * Destination : 10.0.0.0
        * Masque réseau : 255.255.255.0
        * Passerelle : 10.0.0.1
        * Métrique : 256
        * Remarque : nous avons placé cela ici pour permettre à la carte réseau principale de répondre au trafic destiné à sa propre interface. Si ce n’est pas le cas, l’itinéraire suivant entraînerait le trafic destiné à la carte réseau 1 pour passer à la carte réseau 2. Cela créerait un itinéraire asymétrique. 10.0.0.1 est l’adresse IP qu’Azure attribue au sous-réseau NAT. Azure utilise la première adresse IP disponible dans une plage comme passerelle par défaut. Par conséquent, si vous avez utilisé 192.168.0.0/24 pour votre sous-réseau NAT, la passerelle est 192.168.0.1. Dans routage, l’itinéraire plus spécifique gagne, ce qui signifie que cet itinéraire remplace l’itinéraire ci-dessous.

    * Itinéraire 2
        * Interface : Ethernet 2
        * Destination : 10.0.0.0
        * Masque de réseau : 255.255.252.0
        * Passerelle : 10.0.1.1
        * Métrique : 256
        * Remarque : il s’agit d’une capture de l’ensemble du trafic destiné à notre réseau virtuel Azure. Il forcera le trafic sortant de la seconde carte réseau. Vous devez ajouter des itinéraires supplémentaires pour les autres plages auxquelles vous souhaitez que vos machines virtuelles imbriquées accèdent. Par conséquent, si vous êtes sur un réseau local est 172.16.0.0/22, vous souhaiterez peut-être avoir un autre itinéraire pour envoyer ce trafic vers la deuxième carte réseau de notre hyperviseur.

## <a name="creating-a-route-table-within-azure"></a>Création d’une table de routage dans Azure

Pour plus d’informations, consultez [cet article](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-portal) pour en savoir plus sur la création et la gestion d’itinéraires dans Azure.

1. Accédez à https://portal.azure.com.
2. Dans le coin supérieur gauche, sélectionnez créer une ressource.
3. Dans le champ de recherche, tapez « table de routage » et appuyez sur entrée.
4. Le résultat supérieur est table de routage, sélectionnez cette option, puis sélectionnez créer.
5. Nommez la table d’itinéraires, dans mon cas, je l’ai nommée « routes-for-imbriqued-machines virtuelles ».
6. Veillez à sélectionner l’abonnement dans lequel se trouvent les hôtes Hyper-V.
7. Créez un groupe de ressources ou sélectionnez un groupe existant et assurez-vous que la région dans laquelle vous créez la table de routage correspond à la région dans laquelle se trouve votre hôte Hyper-V.
8. Sélectionnez « Create  (Créer) ».

## <a name="configuring-the-route-table"></a>Configuration de la table de routage

1. Accédez à la table de routage que nous venons de créer. Pour ce faire, recherchez le nom de la table de routage dans la barre de recherche en haut au centre du portail.
2. Une fois que vous avez sélectionné la table de routage, accédez à « routes » dans le panneau.
3. Sélectionner « Ajouter ».
4. Donnez un nom à votre itinéraire, j’ai effectué une « imbrication de machines virtuelles ».
5. Pour le préfixe d’adresse, entrez la plage d’adresses IP de notre sous-réseau « flottant ». Dans ce cas, il s’agit de 10.0.2.0/24.
6. Pour « type de tronçon suivant », sélectionnez « appliance virtuelle », puis entrez l’adresse IP de la seconde carte réseau hôtes Hyper-V, qui serait 10.0.1.4, puis sélectionnez « OK ».
7. Maintenant, dans le panneau, sélectionnez « sous-réseaux », ce qui se trouve juste sous « routes ».
8. Sélectionnez « Associate », puis sélectionnez notre réseau virtuel « imbriqué-fun », puis sélectionnez le sous-réseau « Azure-machines virtuelles », puis sélectionnez « OK ».
9. Procédez de la même façon pour le sous-réseau sur lequel réside l’hôte Hyper-V, ainsi que pour tous les autres sous-réseaux qui doivent accéder aux machines virtuelles imbriquées. En cas de connexion 

# <a name="end-state-configuration-reference"></a>Référence de configuration d’état final
L’environnement de ce guide présente les configurations ci-dessous. Cette section est inteded à utiliser comme référence.

1. Informations sur le réseau virtuel Azure.
    * Configuration de réseau virtuel à haut niveau.
        * Nom : imbriqué-fun
        * Espace d’adressage : 10.0.0.0/22
        * Remarque : ce sera constitué de quatre sous-réseaux. En outre, ces plages ne sont pas définies en pierre. N’hésitez pas à répondre à votre environnement. 

    * Configuration générale du premier sous-réseau.
        * Nom : NAT
        * Espace d’adressage : 10.0.0.0/24
        * Remarque : c’est là que notre carte d’interface réseau principale héberge Hyper-V. Ce sera utilisé pour gérer les NAT sortants pour les machines virtuelles imbriquées. Il s’agit de la passerelle vers Internet pour vos machines virtuelles imbriquées.

    * Configuration de niveau supérieur du deuxième sous-réseau.
        * Nom : Hyper-V-LAN
        * Espace d’adressage : 10.0.1.0/24
        * Remarque : notre hôte Hyper-V aura une deuxième carte réseau qui sera utilisée pour gérer le routage entre les machines virtuelles imbriquées et les ressources non-Internet externes à l’hôte Hyper-V.

    * Configuration générale du troisième sous-réseau.
        * Nom : fantôme
        * Espace d’adressage : 10.0.2.0/24
        * Remarque : il s’agit d’un sous-réseau « flottant ». L’espace d’adressage sera consommé par nos machines virtuelles imbriquées et existe pour gérer les publications d’itinéraires vers le site local. Aucune machine virtuelle n’est réellement déployée dans ce sous-réseau.

    * Configuration de niveau supérieur du quatrième sous-réseau.
        * Nom : Azure-machines virtuelles
        * Espace d’adressage : 10.0.3.0/24
        * Remarque : sous-réseau contenant des machines virtuelles Azure.

1. Notre hôte Hyper-V a les configurations de carte réseau ci-dessous.
    * Carte réseau principale 
        * Adresse IP : 10.0.0.4
        * Subnet Mask: 255.255.255.0
        * Passerelle par défaut : 10.0.0.1
        * DNS : configuré pour DHCP
        * Transfert IP activé : non

    * Carte réseau secondaire
        * Adresse IP : 10.0.1.4
        * Subnet Mask: 255.255.255.0
        * Passerelle par défaut : vide
        * DNS : configuré pour DHCP
        * Transfert IP activé : Oui

    * Carte réseau créée par Hyper-V pour le commutateur virtuel interne
        * Adresse IP : 10.0.2.1
        * Subnet Mask: 255.255.255.0
        * Passerelle par défaut : vide

3. Notre table de routage n’a qu’une seule règle.
    * Règle 1
        * Nom : imbriqué-machines virtuelles
        * Destination : 10.0.2.0/24
        * Tronçon suivant : appliance virtuelle-10.0.1.4

## <a name="conclusion"></a>Conclusion

Vous devez maintenant être en mesure de déployer un ordinateur virtuel (même une machine virtuelle 32 bits !) sur votre hôte Hyper-V et de le faire accéder localement et dans Azure.

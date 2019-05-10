---
title: Configuration des machines virtuelles imbriquées pour communiquer directement avec les ressources d’un réseau virtuel Azure
description: Virtualisation imbriquée
keywords: Windows 10, hyper-v, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: 2771989b7745605fb3ce4f95e162ae8b03180b0f
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621577"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Configurer des ordinateurs virtuels imbriquées pour communiquer avec les ressources d’un réseau virtuel Azure

Les instructions d’origine sur le déploiement et la configuration des machines virtuelles imbriquées dans Azure nécessite que vous accédez à ces machines virtuelles par le biais d’un commutateur NAT. Cela présente plusieurs limitations:

1. Machines virtuelles imbriquées ne peuvent pas accéder aux ressources locales ou au sein d’un réseau virtuel Azure.
2. Ressources locales ou des ressources dans Azure accessibles uniquement les machines virtuelles imbriquées par le biais d’un périphérique NAT, ce qui signifie que plusieurs invités ne peuvent pas partager le même port.

Ce document guideront à travers un déploiement dans laquelle nous provoquent l’utilisation de RRAS, des itinéraires définis par utilisateur, un sous-réseau dédié à NAT sortante pour autoriser l’accès en invité internet et un espace d’adressage «flottante» pour autoriser les machines virtuelles imbriquées à se comporter et à communiquer comme toute autre machine virtuelle déployé directement à une basculée dans Azure.

Avant de commencer ce guide, veuillez:

1. Lisez les [conseils fournis ici](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization) sur la virtualisation imbriquée.
2. Lisez cet article entière avant la mise en œuvre.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>Vue d’ensemble de niveau élevé de ce que nous faisons et pourquoi
* Nous allons créer un ordinateur virtuel capable de d’imbrication a deux cartes réseau. 
* Une que carte réseau sera utilisé pour fournir des machines virtuelles imbriquées notre ayant accès à internet via NAT et l’autre carte réseau servira à acheminer le trafic à partir de notre commutateur interne aux ressources externes à l’hyperviseur. Chaque carte réseau sera doivent se trouver dans un autre domaine de routage, ce qui signifie qu’un sous-réseau différent.
* Cela signifie que nous avons besoin d’un réseau virtuel avec un minimum trois sous-réseaux. Une pour NAT, un pour le routage LAN et un qui n’est pas utilisé, mais est «réservé» pour nos machines virtuelles imbriquées. Les noms que nous utilisons pour les sous-réseaux dans ce document sont, «NAT», «Hyper-V-LAN» et «Images».
* La taille de ces sous-réseaux est à votre convenance, mais il existe certaines considérations. La taille des sous-réseaux «Images» détermine le nombre d’adresses IP vous disposez pour vos machines virtuelles imbriquées. En outre, la taille des sous-réseaux «NAT» et «Hyper-V-LAN» détermine le nombre d’adresses IP vous disposez pour les hyperviseurs. Par conséquent, vous pouvez faire techniquement très petites sous-réseaux ici si vous ont été planification uniquement sur la présence d’un ou deux hyperviseurs.
* En arrière-plan: Recevoir imbriquée machines virtuelles pas DHCP à partir de la basculée leur hôte est connecté au même si vous configurez une interne ou un commutateur externe. 
  * Cela signifie que l’hôte Hyper-V doit fournir le service DHCP.
* L’hôte Hyper-V n’est pas prenant en charge des baux actuellement affectées sur le basculée, afin d’éviter une situation dans laquelle l’hôte attribue une adresse IP déjà existant, nous devons allouer un bloc des adresses IP pour une utilisation simplement par l’ordinateur hôte Hyper-V. Cela va nous permettre d’éviter un scénario IP en double.
  * Le bloc d’adresses IP nous choisissons correspondra à un sous-réseau au sein de la même basculée votre Hyper-V est en cours.
  * La raison que nous voulons que cette option pour correspondre à un sous-réseau existant consiste à gérer les publicités BGP sur un ExpressRoute. Si nous compose simplement une plage IP pour l’hôte Hyper-V à utiliser, nous devons créer une série d’itinéraires statiques pour permettre aux clients sur site pour communiquer avec les machines virtuelles imbriquées. Cela ne signifie pas que cela n’est pas une condition requise par dur comme vous pourriez constituent une plage IP pour les machines virtuelles imbriquées et ensuite créer tous les itinéraires nécessaires pour diriger les clients vers l’hôte Hyper-V pour cette plage.
* Nous allons créer un commutateur interne au sein de Hyper-V et puis nous allons désigner l’interface nouvellement créé une adresse IP dans une plage que nous réservons pour DHCP. Cette adresse IP est devenue la passerelle par défaut pour nos machines virtuelles imbriquées et être utilisé pour l’itinéraire entre le commutateur interne et la carte réseau de l’hôte qui est connecté à notre basculée.
* Nous allons installer le rôle de routage et d’accès à distance sur l’hôte, ce qui activera notre hôte dans un routeur.  Cela est nécessaire pour permettre la communication entre les ressources externes à l’hôte et nos machines virtuelles imbriquées.
* Nous indiquera comment accéder à ces machines virtuelles imbriquées autres ressources. Cela nécessite que nous créons une table de routage défini par l’utilisateur qui contient un itinéraire statique pour la plage IP résidant dans les machines virtuelles imbriquées. Cet itinéraire statique pointe vers l’adresse IP pour l’Hyper-V.
* Vous placera ensuite cette UDR sur le sous-réseau de passerelle afin que les clients en provenance de locaux sachent comment joindre nos machines virtuelles imbriquées.
* Vous serez également placer ce UDR sur n’importe quel autre sous-réseau au sein d’Azure qui nécessite une connectivité pour les machines virtuelles imbriquées.
* Pour plusieurs hôtes Hyper-V vous créer des sous-réseaux «flottantes» supplémentaires et ajouter un itinéraire statique supplémentaires à la UDR.
* Lorsque vous retirez un hôte Hyper-V vous serez supprimer/réaffecter notre sous-réseau «flottante» et supprimer cet itinéraire statique dans notre UDR ou s’il s’agit du dernier hôte Hyper-V, retirez le UDR.

## <a name="creating-the-host"></a>Création de l’hôte

J’ai s’attarder sur les valeurs de configuration qui sont jusqu'à préférences personnelles, telles que la machine virtuelle nom, groupe de ressources, etc...

1. Accédez à portal.azure.com
2. Cliquez sur «Créer une ressource» dans le coin supérieur gauche
3. Sélectionnez «Fenêtre Server 2016 VM» dans la colonne populaires
4. Sous l’onglet «Basics» veillez à sélectionner une taille de mémoire virtuelle qui est capable de la virtualisation imbriquée
5. Déplacer vers l’onglet «Mise en réseau»
6. Créer un nouveau réseau virtuel avec la configuration suivante
    * Espace d’adressage basculée: 10.0.0.0/22
    * Sous-réseau 1
        * Nom: NAT
        * Espace d’adresse: 10.0.0.0/24
    * Sous-réseau 2
        * Nom: Hyper-V-LAN
        * Espace d’adresse: 10.0.1.0/24
    * Sous-réseau 3
        * Nom: dupliqué
        * Espace d’adresse: 10.0.2.0/24
    * Sous-réseau 4
        * Nom: Machines virtuelles Azure
        * Espace d’adresse: 10.0.3.0/24
7. Vérifiez que vous avez sélectionné le sous-réseau NAT pour la machine virtuelle
8. Accédez à «révision + créer» et sélectionnez «Créer»

## <a name="create-the-second-network-interface"></a>Créer la deuxième interface réseau
1. Après la machine virtuelle a fini d’approvisionnement Parcourir à celui-ci dans le portail Azure
2. Arrêter l’ordinateur virtuel
3. Une fois arrêté atteindre la «Mise en réseau» sous Paramètres
4. «Attacher l’interface réseau»
5. «Créer l’interface réseau»
6. Lui donner un nom (n’a pas d’importance ce que vous nommez, mais veillez à vous en souvenir)
7. Sélectionnez «Hyper-V-LAN» pour le sous-réseau
8. Veillez à sélectionner le même groupe de ressources votre hôte réside dans
9. «Créer»
10. Ceci vous ramène à l’écran précédent, veillez à sélectionner l’Interface réseau nouvellement créé et sélectionnez «OK»
11. Revenez dans le volet de «Présentation» et redémarrez votre machine virtuelle une fois que l’action précédente est terminée.
12. Accédez à la seconde carte que nous venons de créer, vous pouvez le trouver dans le groupe de ressources que vous avez précédemment sélectionné
13. Accédez à «configurations IP» et activer/désactiver «Transfert IP» sur «Activé» et puis enregistrez la modification

## <a name="setting-up-hyper-v"></a>Configuration d’Hyper-V
1. À distance à votre ordinateur hôte
2. Ouvrez une invite PowerShell avec élévation de privilèges
3. Exécutez la commande suivante `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`
4. Cette opération va redémarrer l’ordinateur hôte
5. Vous reconnecter à l’hôte de continuer avec le reste de l’installation

## <a name="creating-our-virtual-switch"></a>Création de notre commutateur virtuel

1. Ouvrez PowerShell en mode d’administration.
2. Créer un commutateur interne: `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. Affectez l’interface nouvellement créé une adresse IP: `New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>Installer et configurer DHCP

*De nombreuses personnes manquer ce composant lorsqu’il essaie tout d’abord d’obtenir l’utilisation de la virtualisation imbriquée. Contrairement à local où vos ordinateurs virtuels invités recevront DHCP à partir du réseau résidant sur votre ordinateur hôte machines virtuelles imbriquées dans Azure doivent être fournis DHCP via l’hôte, sur qu'ils s’exécutent. Qui ou vous devez affecter statiquement une adresse IP à chaque machine virtuelle imbriquée, ce qui n’est pas évolutif.*

1. Installez le rôle DHCP: `Install-WindowsFeature DHCP -IncludeManagementTools`
2. Créer l’étendue DHCP: `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`
3. Configurer les options DNS et passerelle par défaut pour l’étendue: `Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`
    * Veillez à un serveur DNS valide d’entrée si vous souhaitez que la résolution de nom pour fonctionner. Dans ce cas, j’utilise [récursive d’Azure DNS](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances).

## <a name="installing-remote-access"></a>L’installation d’accès à distance

1. Ouvrez le Gestionnaire de serveur et sélectionnez «Ajouter des rôles et fonctionnalités».
2. Sélectionnez «Suivant» jusqu'à ce que vous atteigniez «Rôles serveur».
3. Vérifiez les «Accès à distance» et cliquez sur «Suivant» jusqu'à ce que vous atteigniez «Les Services de rôle».
4. Vérifier «Routage», sélectionnez «Ajouter des fonctionnalités» et sélectionnez «Suivant» et «Installer». Terminez l’Assistant et attendez pour l’installation se termine.

## <a name="configuring-remote-access"></a>Configuration de l’accès à distance

1. Ouvrez le Gestionnaire de serveur, puis sélectionnez «Outils» et sélectionnez «Routage et accès à distance».
2. Sur le côté droit du Panneau de gestion de routage et d’accès à distance vous voir une icône avec votre nom de serveurs en regard de celle-ci, avec le bouton droit cliquez ici et sélectionnez «Configurer et activer le routage et accès à distance».
3. Sélectionnez «Suivant» à l’Assistant, cochez la case d’option pour «Configuration personnalisée» et sélectionnez «Suivant».
4. Vérifier «NAT» et «Routage LAN» puis sélectionnez «suivant» et «Terminerez». Si elle vous demande de démarrer le service, puis effectuer cette opération.
5. Maintenant, accédez au nœud «IPv4» et développer afin que le nœud «NAT» est mis à disposition.
6. Cliquez avec le bouton droit sur «NAT», sélectionnez «Nouvelle Interface» Sélectionnez «Ethernet», il doit s’agir de votre première carte réseau avec l’adresse IP de «10.0.0.4»
7. Nous devons maintenant créer des itinéraires statiques pour forcer le trafic réseau local à la seconde carte réseau. Pour cela, vous devez en accédant au nœud «Itinéraires statiques» sous «IPv4».
8. Une fois qu’il nous allons créer les itinéraires suivants.
    * Itinéraire 1
        * Interface: Ethernet
        * Destination: 10.0.0.0
        * Masque de réseau: 255.255.255.0
        * Passerelle: 10.0.0.1
        * Métrique: 256
        * Remarque: Nous placez ce code ici pour permettre à la carte réseau principale répondre au trafic destiné à sa sortie sa propre interface. Si nous n’avions pas ce ici l’itinéraire suivant conduira le trafic destiné à la carte réseau 1 accéder à la carte réseau 2. Cela crée un itinéraire asymétrique. 10.0.0.1 est l’adresse IP qui Azure affecte au sous-réseau NAT. Azure utilise la première IP disponibles dans une plage comme la passerelle par défaut. Par conséquent, si vous n’avez utilisé 192.168.0.0/24 pour votre sous-réseau NAT, la passerelle sera 192.168.0.1. Dans le routage l’itinéraire plus spécifique wins, ce qui signifie que cet itinéraire seront remplacent le ci-dessous itinéraire.

    * Itinéraire 2
        * Interface: Ethernet 2
        * Destination: 10.0.0.0
        * Masque de réseau: 255.255.252.0
        * Passerelle: 10.0.1.1
        * Métrique: 256
        * Remarque: Il s’agit d’une capture que toutes les acheminer le trafic destiné à notre basculée Azure. Cela force le trafic à la seconde carte réseau. Vous devez ajouter des itinéraires supplémentaires pour les autres plages que vous souhaitez que vos machines virtuelles imbriquées pour accéder à. Par conséquent, si vous êtes sur site réseau est 172.16.0.0/22, vous ne souhaitez pas posséder un autre itinéraire d’envoyer le trafic correspondant à la seconde carte réseau de notre hyperviseur.

## <a name="creating-a-route-table-within-azure"></a>Création d’une table de routage dans Azure

Faire référence à [cet article](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-portal) pour en savoir plus profondeur lire sur la création et la gestion des itinéraires dans Azure.

1. Accédez à https://portal.azure.com.
2. Dans le coin supérieur gauche, sélectionnez «Créer une ressource».
3. Dans le champ de recherche, tapez «Table de routage» et appuyez sur ENTRÉE.
4. Le résultat supérieur s’être la Table de routage, sélectionnez cette option et sélectionnez «Créer»
5. Nom de la Table de routage, dans mon cas j’ai nommé «Itinéraires-de-imbriqués-machines virtuelles».
6. Assurez-vous que vous sélectionnez l’abonnement même résidant dans vos hôtes Hyper-V.
7. Soit créer un nouveau groupe de ressources ou sélectionnez-en une et n’oubliez pas que la région que vous créez dans la Table de routage est la même région résidant sur votre ordinateur hôte Hyper-V.
8. Sélectionnez «Créer».

## <a name="configuring-the-route-table"></a>Configuration de la table de routage

1. Accédez à la Table de routage que nous venons de créer. Vous pouvez le faire en recherchant le nom de la Table de routage à partir de la barre de recherche en haut du portail.
2. Une fois que vous avez sélectionné la Table de routage accéder aux «Itinéraires» dans le panneau.
3. Sélectionnez «Ajouter».
4. Attribuez un nom à votre itinéraire, je suis avec «Imbriqué-VM».
5. Adresse préfixe saisie la plage IP pour notre sous-réseau «flottante». Dans ce cas, il serait 10.0.2.0/24.
6. Pour «Saut suivant de type» sélectionner «Appliance virtuelle» et puis entrez l’adresse IP adresse pour Hyper-V héberge la seconde carte réseau, ce qui serait être 10.0.1.4 et sélectionnez «OK».
7. Maintenant à partir d’au sein de la sélection de panneau «Sous-réseaux», il s’agit juste en dessous de «Itinéraires».
8. Sélectionnez «Associer», puis notre basculée «Imbriqué-amusant» et sélectionnez ensuite le sous-réseau «Machines virtuelles Azure» et puis sélectionnez «OK».
9. Répétez ce processus même pour le sous-réseau que notre hôte Hyper-V doit résider sur ainsi que pour les autres sous-réseaux qui doivent y accéder les machines virtuelles imbriquées. Si connecté 

# <a name="end-state-configuration-reference"></a>Référence de configuration d’état final
L’environnement dans ce guide présente les configurations ci-dessous. Cette section est prévu pour être utilisé en tant que référence.

1. Informations de réseau virtuel Azure.
    * Configuration de niveau élevé basculée.
        * Nom: Imbriqués-amusant
        * Espace d’adresse: 10.0.0.0/22
        * Remarque: Ce sera être constitué de quatre sous-réseaux. En outre, ces plages ne sont pas définies dans le noyau. N’hésitez pas à résoudre votre environnement comme vous le souhaitez. 

    * Première sous-réseau Configuration de niveau élevé.
        * Nom: NAT
        * Espace d’adresse: 10.0.0.0/24
        * Remarque: Il s’agit d’où notre Hyper-V héberge la carte réseau principale réside. Il sera utilisé pour gérer le trafic sortant NAT pour les machines virtuelles imbriquées. Il sera la passerelle à internet pour que vos machines virtuelles imbriquées.

    * Deuxième sous-réseau Configuration de niveau élevé.
        * Nom: Hyper-V-LAN
        * Espace d’adresse: 10.0.1.0/24
        * Remarque: Notre hôte Hyper-V disposera d’une seconde carte réseau qui sera utilisée pour gérer le routage entre les machines virtuelles imbriquées et les ressources non internet externes à l’hôte Hyper-V.

    * Troisième sous-réseau Configuration de niveau élevé.
        * Nom: dupliqué
        * Espace d’adresse: 10.0.2.0/24
        * Remarque: Il s’agit d’un sous-réseau «flottant». L’espace d’adresse sera utilisée par nos machines virtuelles imbriquées et il n’existe pour gérer les annonces d’itinéraires à local. Aucune ordinateurs virtuels ne seront effectivement déployés dans ce sous-réseau.

    * Quatrième sous-réseau Configuration de niveau élevé.
        * Nom: Machines virtuelles Azure
        * Espace d’adresse: 10.0.3.0/24
        * Remarque: Sous-réseau contenant des machines virtuelles Azure.

1. Notre hôte Hyper-V a les configurations de carte réseau ci-dessous.
    * Carte réseau principale 
        * Adresse IP: 10.0.0.4
        * Le masque de sous-réseau: 255.255.255.0
        * Passerelle par défaut: 10.0.0.1
        * DNS: Configuré pour DHCP
        * Activé de la transmission IP: No

    * Carte réseau secondaire
        * Adresse IP: 10.0.1.4
        * Le masque de sous-réseau: 255.255.255.0
        * Passerelle par défaut: vide
        * DNS: Configuré pour DHCP
        * Transfert IP activé: Oui

    * Créé de carte réseau pour commutateur virtuel interne Hyper-V
        * Adresse IP: 10.0.2.1
        * Le masque de sous-réseau: 255.255.255.0
        * Passerelle par défaut: vide

3. Notre Table de routage aura une règle unique.
    * Règle 1
        * Nom: Imbriqués-VM
        * Destination: 10.0.2.0/24
        * Tronçon suivant: Appliance virtuelle - 10.0.1.4

## <a name="conclusion"></a>Conclusion

Vous devez maintenant être en mesure de déployer une machine virtuelle (même une machine virtuelle 32 bits!) sur votre ordinateur hôte Hyper-V et qu’il est accessible à partir de locaux et dans Azure.

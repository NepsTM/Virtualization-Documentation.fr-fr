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
ms.openlocfilehash: abe6f0da68ff90af0b2b5e675f70f106d42ca81c
ms.sourcegitcommit: 8db42caaace760b7eeb1367b631b38e7904a9f26
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2018
ms.locfileid: "8962307"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Configurer des ordinateurs virtuels imbriquées pour communiquer avec les ressources d’un réseau virtuel Azure

Les instructions d’origine sur le déploiement et la configuration des machines virtuelles imbriquées dans Azure nécessite que vous accédez à ces machines virtuelles par le biais d’un commutateur NAT. Cela présente plusieurs limitations:

1. Machines virtuelles imbriquées ne peuvent pas accéder aux ressources locales ou au sein d’un réseau virtuel Azure.
2. Ressources locales ou des ressources dans Azure accessibles uniquement les machines virtuelles imbriquées par le biais d’un périphérique NAT, ce qui signifie que plusieurs invités ne peuvent pas partager le même port.

Ce document guideront à travers un déploiement dans laquelle nous provoquent l’utilisation de RRAS, certains itinéraires définis par utilisateur et un espace d’adressage «flottante» pour autoriser des machines virtuelles imbriquées à se comporter et à communiquer comme n’importe quel autre ordinateur virtuel déployé directement à une basculée dans Azure.

Avant de commencer ce guide, veuillez:

1. Lisez les [conseils fournis ici](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/nested-virtualization) sur la virtualisation imbriquée, créer vos VM capable d’imbrication et installez le rôle Hyper-V au sein de ces machines virtuelles. Ne continuez pas au-delà de configurer le rôle Hyper-V.
2. Lisez cet article entière avant la mise en œuvre.

Ce guide repose sur les hypothèses suivantes sur l’environnement cible:

1. Nous sommes fonctionne dans une [topologie en étoile](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke), avec notre hub est connecté à un ExpressRoute.
1. Notre réseau étoile est attribué à l’espace d’adresse de 10.0.0.0/23, qui est constitué en deux /24 sous-réseaux.
    * 10.0.0.0/24 – sous-réseau où réside notre hôte Hyper-V.
    * 10.0.1.0/24: il s’agit d’un sous-réseau «flottant». Cet espace d’adresse sera utilisé par nos machines virtuelles imbriquées et il n’existe pour gérer les annonces d’itinéraires à local.
    * L’étoile basculée intelligemment nommé «Étoile».

1. Notre plage IP de réseaux hub n’est pas pertinente, mais savez que son nom est «Hub».
1. Notre Hyper-V est affectée à l’adresse 10.0.0.4/24.
1. Nous avons un serveur DNS à 10.0.0.10/24, ce n’est pas obligatoire, mais une hypothèse de notre procédure pas à pas.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>Vue d’ensemble de niveau élevé de ce que nous sommes en train de faire et pourquoi

* En arrière-plan: Recevoir imbriquée machines virtuelles pas DHCP à partir de la basculée leur hôte est connecté au même si vous configurez une interne ou un commutateur externe. 
  * Cela signifie que l’hôte Hyper-V doit fournir le service DHCP.
* Nous allouera un bloc d’adresses IP pour une utilisation juste par l’ordinateur hôte Hyper-V.  L’hôte Hyper-V n’est pas prenant en charge des baux actuellement affectées sur le basculée, afin d’éviter une situation dans laquelle l’hôte attribue une adresse IP déjà existant, nous devons allouer un bloc des adresses IP pour une utilisation simplement par l’ordinateur hôte Hyper-V. Cela va nous permettre d’éviter un scénario IP en double.
  * Le bloc d’adresses IP nous choisissons correspondra à un sous-réseau au sein de le basculée résidant sur votre ordinateur hôte Hyper-V.
  * La raison que nous voulons que cette option pour correspondre à un sous-réseau existant consiste à gérer les publicités BGP sur le ExpressRoute. Si nous compose simplement une plage IP pour l’hôte Hyper-V à utiliser, nous devons créer une série d’itinéraires statiques pour permettre aux clients sur site pour communiquer avec les machines virtuelles imbriquées. Cela ne signifie pas que cela n’est pas une condition requise par dur comme vous pourriez constituent une plage IP pour les machines virtuelles imbriquées et ensuite créer tous les itinéraires nécessaires pour diriger les clients vers l’hôte Hyper-V pour cette plage.
* Nous allons créer un commutateur interne au sein de Hyper-V et puis nous allons désigner l’interface nouvellement créé une adresse IP dans une plage que nous réservons pour DHCP. Cette adresse IP est devenue la passerelle par défaut pour nos machines virtuelles imbriquées et être utilisé pour l’itinéraire entre le commutateur interne et la carte réseau de l’hôte qui est connecté à notre basculée.
* Nous allons installer le rôle de routage et d’accès à distance sur l’hôte, ce qui activera notre hôte dans un routeur.  Cela est nécessaire pour permettre la communication entre les ressources externes à l’hôte et nos machines virtuelles imbriquées.
* Nous allons pour indiquer les autres ressources comment accéder à ces machines virtuelles imbriquées. Cela nécessite que nous créons une table de routage défini par l’utilisateur qui contient un itinéraire statique pour la plage IP résidant dans les machines virtuelles imbriquées. Cet itinéraire statique pointe vers l’adresse IP pour l’Hyper-V.
* Vous placera ensuite cette UDR sur le sous-réseau de passerelle afin que les clients en provenance de locaux sachent comment joindre nos machines virtuelles imbriquées.
* Vous serez également placer ce UDR sur n’importe quel autre sous-réseau au sein d’Azure qui nécessite une connectivité pour les machines virtuelles imbriquées.
* Pour plusieurs hôtes Hyper-V vous créer des sous-réseaux «flottantes» supplémentaires et ajouter un itinéraire statique supplémentaires à la UDR.
* Lorsque vous retirez un hôte Hyper-V vous serez supprimer/réaffecter notre sous-réseau «flottante» et supprimer cet itinéraire statique dans notre UDR ou s’il s’agit du dernier hôte Hyper-V, retirez le UDR.

## <a name="creating-our-virtual-switch"></a>Création de notre commutateur virtuel

1. Ouvrez PowerShell en mode d’administration.
2. Créer un commutateur interne: `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. Affectez l’interface nouvellement créé une adresse IP: `New-NetIPAddress –IPAddress 10.0.1.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>Installer et configurer DHCP

*De nombreuses personnes manquer ce composant lorsqu’il essaie tout d’abord d’obtenir l’utilisation de la virtualisation imbriquée. Contrairement à local où vos ordinateurs virtuels invités recevront DHCP à partir du réseau résidant sur votre ordinateur hôte machines virtuelles imbriquées dans Azure doivent être fournis DHCP via l’hôte, sur qu'ils s’exécutent. Qui ou vous devez affecter statiquement une adresse IP à chaque machine virtuelle imbriquée, ce qui n’est pas évolutif.*

1. Installez le rôle DHCP: `Install-WindowsFeature DHCP -IncludeManagementTools`
2. Créer l’étendue DHCP: `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.1.2 -EndRange 10.0.1.254 -SubnetMask 255.255.255.0`
3. Configurer les options DNS et passerelle par défaut pour l’étendue: `Set-DhcpServerV4OptionValue -DnsServer 10.0.0.10 -Router 10.0.1.1`
    * Veillez à entrer un serveur DNS valide. J’ai se produire dans ce cas de disposer d’un serveur sur le réseau 10.0.0.0/24 qui diffuse DNS Windows.

## <a name="installing-remote-access"></a>L’installation d’accès à distance

* Ouvrez le Gestionnaire de serveur et sélectionnez «Ajouter des rôles et fonctionnalités».
* Sélectionnez «Suivant» jusqu'à ce que vous atteigniez «Rôles serveur».
* Vérifiez les «Accès à distance» et cliquez sur «Suivant» jusqu'à ce que vous atteigniez «Les Services de rôle».
* Vérifier «Routage», sélectionnez «Ajouter des fonctionnalités» et sélectionnez «Suivant» et «Installer». Terminez l’Assistant et attendez pour l’installation se termine.

## <a name="configuring-remote-access"></a>Configuration de l’accès à distance

* Ouvrez le Gestionnaire de serveur, puis sélectionnez «Outils» et sélectionnez «Routage et accès à distance».
* Sur le côté droit du Panneau de gestion de routage et d’accès à distance vous voir une icône avec votre nom de serveurs en regard de celle-ci, avec le bouton droit cliquez ici et sélectionnez «Configurer et activer le routage et accès à distance».
* Sélectionnez «Suivant» dans l’Assistant, cochez la case d’option pour «Connexion sécurisée entre deux réseaux privés» et sélectionnez «Suivant».
* Sélectionnez la case d’option pour «Non» lorsque vous êtes invité si vous souhaitez utiliser des connexions à la demande, puis sélectionnez «Suivant» et puis sélectionnez «Terminer».

## <a name="creating-a-route-table-within-azure"></a>Création d’une Table de routage dans Azure

Faire référence à [cet article](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal) pour en savoir plus profondeur lire sur la création et la gestion des itinéraires dans Azure.

* Accédez à https://portal.azure.com.
* Dans le coin supérieur gauche, sélectionnez «Créer une ressource».
* Dans le champ de recherche, tapez «Table de routage» et appuyez sur ENTRÉE.
* Le résultat supérieur s’être la Table de routage, sélectionnez cette option et sélectionnez «Créer»
* Nom de la Table de routage, dans mon cas j’ai nommé «Itinéraires-de-imbriqués-machines virtuelles».
* Assurez-vous que vous sélectionnez l’abonnement même résidant dans vos hôtes Hyper-V.
* Soit créer un nouveau groupe de ressources ou sélectionnez-en une et n’oubliez pas que la région que vous créez dans la Table de routage est la même région résidant sur votre ordinateur hôte Hyper-V.
* Sélectionnez «Créer».

## <a name="configuring-the-route-table"></a>Configuration de la Table de routage

* Accédez à la Table de routage que nous venons de créer. Vous pouvez le faire en recherchant le nom de la Table de routage à partir de la barre de recherche en haut du portail.
* Une fois que vous avez sélectionné la Table de routage accéder aux «Itinéraires» dans le panneau.
* Sélectionnez «Ajouter».
* Attribuez un nom à votre itinéraire, je suis avec «Imbriqué-VM».
* Adresse préfixe saisie la plage IP pour notre sous-réseau «flottante». Dans ce cas, il serait 10.0.1.0/24.
* Pour «Type de saut suivant», sélectionnez «Appliance virtuelle», puis entrez l’adresse IP de l’hôte Hyper-V, qui serait être 10.0.0.4 et sélectionnez «OK».
* Maintenant à partir d’au sein de la sélection de panneau «Sous-réseaux», il s’agit juste en dessous de «Itinéraires».
* Sélectionnez «Associer», puis sélectionnez notre réseau virtuel «Hub» et puis sélectionnez le «GatewaySubnet» et sélectionnez «OK».
* Répétez ce processus même pour le sous-réseau que notre hôte Hyper-V doit résider sur ainsi que pour les autres sous-réseaux qui doivent y accéder les machines virtuelles imbriquées.

## <a name="conclusion"></a>Conclusion

Vous devez maintenant être en mesure de déployer une machine virtuelle (voire une machine virtuelle 32 bits!) sur votre ordinateur hôte Hyper-V et qu’il est accessible à partir de locaux et dans Azure.
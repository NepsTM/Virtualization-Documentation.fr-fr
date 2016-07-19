---
title: "Introduction à Hyper-V sur Windows 10"
description: "Introduction à Hyper-V sur Windows 10."
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
translationtype: Human Translation
ms.sourcegitcommit: 53539a0325b3f07e542ca1dd0a4352239e8a65b3
ms.openlocfilehash: 25295b8a2888e25090439a3490c9ff7c3214f23a

---

# Introduction à Hyper-V sur Windows 10

Nombreux sont les développeurs de logiciels, professionnels de l’informatique ou passionnés de technologies qui doivent exécuter plusieurs systèmes d’exploitation.  Au lieu d’affecter du matériel spécifique à chaque système d’exploitation, vous pouvez utiliser Hyper-V pour exécuter plusieurs machines virtuelles sur un même ordinateur Windows.

> Microsoft Virtual PC disparaîtra en avril 2017. Hyper-V sur Windows 10 Enterprise et Windows 10 Professionnel sera la solution de remplacement prise en charge.  

## Raisons justifiant la virtualisation
La virtualisation permet à tout utilisateur d’exécuter plusieurs systèmes d’exploitation, configurations logicielles et configurations matérielles sur la même machine physique.  Hyper-V offre la technologie de virtualisation et les outils nécessaires pour gérer vos machines virtuelles.

Hyper-V peut être utilisé de nombreuses manières. Exemple :

* Exécuter un logiciel nécessitant une version antérieure de Windows ou des systèmes d’exploitation non-Windows. 

* Se familiariser avec d’autres systèmes d’exploitation. Hyper-V permet de créer et de supprimer très facilement différents systèmes d’exploitation.

* Tester des logiciels sur plusieurs systèmes d’exploitation à l’aide de plusieurs machines virtuelles. Grâce à Hyper-V, vous pouvez les exécuter sur un seul ordinateur de bureau ou portable. Ces machines virtuelles peuvent être exportées, puis importées dans un autre système Hyper-V, notamment Azure.

* Résoudre les problèmes liés aux machines virtuelles à partir de n’importe quel déploiement Hyper-V. Vous pouvez exporter une machine virtuelle à partir de votre environnement de production, l’ouvrir sur un ordinateur de bureau exécutant Hyper-V, résoudre les problèmes liés à la machine virtuelle, puis la réexporter dans l’environnement de production. 

* La mise en réseau virtuelle vous permet de créer un environnement avec plusieurs machines de test/développement/démonstration sans risquer d’affecter le réseau de production.

## Configuration requise
Hyper-V est uniquement disponible dans les éditions Professionnel, Entreprise et Éducation de Windows 8 et versions ultérieures.

Il nécessite un système 64 bits avec la traduction d’adresses de second niveau. La traduction d’adresses de second niveau est une fonctionnalité présente dans la génération actuelle de processeurs 64 bits Intel et AMD.  Vous devez également exécuter une version 64 bits de Windows.  
Cela étant dit, Hyper-V prend en charge les systèmes d’exploitation 32 bits et 64 bits au sein des machines virtuelles.

Vous pouvez exécuter trois ou quatre machines virtuelles de base sur un hôte disposant de 4 Go de RAM. Pour exécuter plus de machines virtuelles, davantage de ressources sont nécessaires. En revanche, vous pouvez créer des machines virtuelles de grande taille avec 32 processeurs et 512 Go de RAM, selon votre matériel physique.

Pour plus d’informations sur la configuration requise pour Hyper-V et la procédure à suivre pour vérifier que Hyper-V s’exécute sur votre machine, consultez [Procédure pas à pas : Configuration requise pour Hyper-V sur Windows 10](..\quick_start\walkthrough_install.md)


## Systèmes d’exploitation que vous pouvez exécuter dans une machine virtuelle
Le terme « invité » fait référence à une machine virtuelle et « hôte » fait référence à l’ordinateur qui exécute la machine virtuelle. Hyper-V sur Windows prend en charge de nombreux systèmes d’exploitation invités différents, dont plusieurs versions de Linux, FreeBSD et Windows. 

À titre de rappel, vous devez avoir une licence valide pour les systèmes d’exploitation que vous utilisez dans les machines virtuelles. 

Pour plus d’informations sur les systèmes d’exploitation pris en charge comme invités dans Hyper-V sur Windows, voir [Systèmes d’exploitation invités Windows pris en charge](supported_guest_os.md) et [Machines virtuelles Linux et FreeBSD sur Hyper-V](https://technet.microsoft.com/library/dn531030.aspx). 


## Différences entre Hyper-V sur Windows et Hyper-V sur Windows Server
Certaines fonctionnalités se comportent différemment dans Hyper-V sur Windows et Hyper-V sur Windows Server. 

Le modèle de gestion de mémoire est différent pour Hyper-V sur Windows. Sur un serveur, la mémoire Hyper-V est gérée en partant du principe que seules les machines virtuelles sont exécutées sur le serveur. Dans Hyper-V sur Windows, la mémoire est gérée en prévision du fait que la plupart des machines clientes exécutent des logiciels sur l’hôte en plus des machines virtuelles. Par exemple, un développeur peut exécuter Visual Studio ainsi que plusieurs machines virtuelles sur le même ordinateur.

### Fonctionnalités Hyper-V disponibles dans Windows Server uniquement
Certaines fonctionnalités incluses dans Hyper-V sur Windows Server ne figurent pas dans Hyper-V sur Windows. Par exemple :

* Virtualisation de GPU à l’aide de RemoteFX 
* Migration dynamique des machines virtuelles d’un hôte vers un autre
* Réplication Hyper-V
* Fibre Channel virtuel
* Mise en réseau SR-IOV
* .VHDX partagé

## Limitations
L’utilisation de la virtualisation présente certaines limitations. Les fonctionnalités ou applications qui dépendent d’un matériel spécifique ne fonctionnent pas correctement dans une machine virtuelle. Par exemple, les jeux ou applications qui nécessitent un traitement avec des GPU risquent de ne pas fonctionner correctement. Par ailleurs, les applications basées sur les minuteurs sous 10 ms, notamment les applications de mixage de concert et les minuteurs de haute précision, peuvent être confrontées à des problèmes d’exécution dans une machine virtuelle.

En outre, si Hyper-V est activé, les applications de haute précision sensibles à la latence peuvent également être confrontées à des problèmes d’exécution dans l’hôte.  En effet, quand la virtualisation est activée, le système d’exploitation hôte est également exécuté sur la couche de virtualisation Hyper-V, tout comme les systèmes d’exploitation invités. Toutefois, contrairement aux invités, le système d’exploitation hôte présente la particularité d’avoir un accès direct à l’ensemble du matériel, ce qui signifie que les applications avec une configuration matérielle requise spéciale continuent de fonctionner sans problème dans le système d’exploitation hôte.

## Étape suivante
[Procédure pas à pas : Installer Hyper-V sur Windows 10](..\quick_start\walkthrough_install.md) 

Consultez [Nouveautés](whats_new.md) dans Hyper-V sur Windows 10.




<!--HONumber=Jul16_HO2-->



---
title: "Introduction à Hyper-V sur Windows10"
description: "Introduction à Hyper-V, à la virtualisation et aux technologies connexes."
keywords: windows10, hyper-v
author: scooley
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
ms.openlocfilehash: 307cd592a9deda41fd2a892d49eadbc5ae436d84
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/21/2017
---
# Introduction à Hyper-V sur Windows10

> Hyper-V remplace MicrosoftVirtual PC. 

Nombreux sont les développeurs de logiciels, professionnels de l’informatique ou passionnés de technologies qui doivent exécuter plusieurs systèmes d’exploitation. Au lieu d’affecter du matériel spécifique à chaque système d’exploitation, vous pouvez utiliser Hyper-V pour exécuter un système d’exploitation ou un système informatique sous forme de machine virtuelle sous Windows.  

![](media/HyperVNesting.png)

Hyper-V fournit spécifiquement des capacités de virtualisation matérielle.  Cela signifie que chaque machine virtuelle s’exécute sur du matériel virtuel.  Hyper-V vous permet de créer des disques durs virtuels, des commutateurs virtuels et un certain nombre d’autres périphériques virtuels qui peuvent être ajoutés aux machines virtuelles.

## Raisons justifiant la virtualisation

La virtualisation vous permet ce qui suit:  
* Exécuter un logiciel nécessitant une version antérieure de Windows ou des systèmes d’exploitation autres que Windows. 

* Se familiariser avec d’autres systèmes d’exploitation. Hyper-V permet de créer et de supprimer très facilement différents systèmes d’exploitation.

* Tester des logiciels sur plusieurs systèmes d’exploitation à l’aide de plusieurs machines virtuelles. Grâce à Hyper-V, vous pouvez les exécuter sur un seul ordinateur de bureau ou portable. Ces machines virtuelles peuvent être exportées, puis importées dans un autre système Hyper-V, notamment Azure.

* Résoudre les problèmes liés aux machines virtuelles à partir de n’importe quel déploiement Hyper-V. Vous pouvez exporter une machine virtuelle à partir de votre environnement de production, l’ouvrir sur un ordinateur de bureau exécutant Hyper-V, résoudre les problèmes liés à la machine virtuelle, puis la réexporter dans l’environnement de production. 

* La mise en réseau virtuelle vous permet de créer un environnement avec plusieurs machines de test/développement/démonstration sans risquer d’affecter le réseau de production.

## Configuration système requise
Hyper-V est disponible sur les versions 64bits des éditions Professionnel, Entreprise et Éducation de Windows8 et versions ultérieures.  Il n’est pas disponible sur Windows Édition familiale.  

>  Mettez à niveau l’édition Windows10 Famille vers Windows10 Professionnel en ouvrant **Paramètres** > **Mise à jour et sécurité** > **Activation**. Vous pouvez alors visiter le magasin et acheter la mise à niveau.

La plupart des ordinateurs exécuteront Hyper-V. Toutefois, les machines virtuelles nécessitent, elles, d'importantes ressources car elles exécutent un système d’exploitation complet.  Vous pouvez généralement exécuter une ou plusieurs machines virtuelles sur un ordinateur équipé de 4Go de RAM. Vous aurez cependant besoin de plus de ressources pour des machines virtuelles supplémentaires ou pour installer et exécuter des logiciels gourmands en ressources tels que des jeux, des logiciels de montage vidéo ou de conception d'ingénierie. 

Votre ordinateur devra disposer de la technologie SLAT (Second Level Address Translation), présente sur la génération actuelle de processeurs 64bits Intel et AMD.  Vous devrez également exécuter une version 64bits de Windows.

Pour en savoir plus sur la configuration requise pour Hyper-V et la procédure à suivre pour vérifier que Hyper-V s’exécute sur votre machine, consultez les [Références de la configuration requise pour Hyper-V](..\reference\hyper-v-requirements.md).

## Systèmes d’exploitation que vous pouvez exécuter dans une machine virtuelle
Le terme «invité» fait référence à une machine virtuelle et «hôte» fait référence à l’ordinateur qui exécute la machine virtuelle. Hyper-V sur Windows prend en charge de nombreux systèmes d’exploitation invités différents, dont plusieurs versions de Linux, FreeBSD et Windows. 

À titre de rappel, vous devez avoir une licence valide pour les systèmes d’exploitation que vous utilisez dans les machines virtuelles. 

Pour en savoir plus sur les systèmes d’exploitation pris en charge comme invités dans Hyper-V sur Windows, voir [Systèmes d’exploitation invités Windows pris en charge](supported-guest-os.md) et [Systèmes d’exploitation invités Linux pris en charge](https://technet.microsoft.com/library/dn531030.aspx). 


## Différences entre Hyper-V sur Windows et Hyper-V sur Windows Server
Certaines fonctionnalités se comportent différemment dans Hyper-V sur Windows et Hyper-V sur Windows Server. 

Le modèle de gestion de mémoire est différent pour Hyper-V sur Windows. Sur un serveur, la mémoire Hyper-V est gérée en partant du principe que seules les machines virtuelles sont exécutées sur le serveur. Dans Hyper-V sur Windows, la mémoire est gérée en prévision du fait que la plupart des machines clientes exécutent des logiciels sur l’hôte en plus des machines virtuelles. Par exemple, un développeur peut exécuter Visual Studio, ainsi que plusieurs machines virtuelles, sur le même ordinateur.

Certaines fonctionnalités incluses dans Hyper-V sur Windows Server ne figurent pas dans Hyper-V sur Windows. Par exemple:

* Virtualisation de GPU à l’aide de RemoteFX 
* Migration dynamique des machines virtuelles d’un hôte vers un autre
* Réplication Hyper-V
* Fibre Channel virtuel
* Mise en réseau SR-IOV
* .VHDX partagé

## Limitations
L’utilisation de la virtualisation présente certaines limitations. Les fonctionnalités ou applications qui dépendent d’un matériel spécifique ne fonctionnent pas correctement dans une machine virtuelle. Par exemple, les jeux ou applications qui nécessitent un traitement avec des GPU risquent de ne pas fonctionner correctement. Par ailleurs, les applications basées sur les minuteurs sous 10ms, notamment les applications de mixage de concert et les minuteurs de haute précision, peuvent être confrontées à des problèmes d’exécution dans une machine virtuelle.

En outre, si Hyper-V est activé, les applications de haute précision sensibles à la latence peuvent également être confrontées à des problèmes d’exécution dans l’hôte.  En effet, quand la virtualisation est activée, le système d’exploitation hôte est également exécuté sur la couche de virtualisation Hyper-V, tout comme les systèmes d’exploitation invités. Toutefois, contrairement aux invités, le système d’exploitation hôte présente la particularité d’avoir un accès direct à l’ensemble du matériel, ce qui signifie que les applications avec une configuration matérielle requise spéciale continuent de fonctionner sans problème dans le système d’exploitation hôte.

## Étape suivante
[Installer Hyper-V sur Windows10](..\quick-start\enable-hyper-v.md) 

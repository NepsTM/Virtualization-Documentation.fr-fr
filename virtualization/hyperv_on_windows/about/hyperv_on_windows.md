---
title: Introduction à Hyper-V sur Windows 10
description: Introduction à Hyper-V sur Windows 10.
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &1413932348 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
---

# Introduction à Hyper-V sur Windows 10

Nombreux sont les développeurs de logiciels, professionnels de l’informatique ou passionnés de technologies qui doivent exécuter plusieurs systèmes d’exploitation, parfois sur différentes machines. Mais tout le monde ne dispose pas de l’espace suffisant pour loger toutes ces machines. Au lieu de dédier un matériel physique à l’exécution de chacun de vos ordinateurs, exécutez-les comme <g id="2" ctype="x-em">machines virtuelles</g> Hyper-V. Cette technologie de virtualisation vous fera économiser du temps et de l’espace.

> Microsoft Virtual PC disparaîtra en avril 2017. Hyper-V sur Windows 10 le remplacera.

## Utilisations de la virtualisation

La virtualisation permet à tout utilisateur de gérer facilement plusieurs environnements de test comprenant de nombreux systèmes d’exploitation, des configurations logicielles et des configurations matérielles. Hyper-V offre la virtualisation sur Windows ainsi qu’un mécanisme simple pour basculer rapidement entre ces environnements sans encourir de frais matériels supplémentaires.

Hyper-V peut être utilisé de nombreuses manières. Exemple :

- Un environnement de test comportant plusieurs machines virtuelles peut être créé sur un seul ordinateur de bureau ou portable. Une fois le test terminé, ces machines virtuelles peuvent être exportées, puis importées dans un autre système Hyper-V.

- Les développeurs peuvent utiliser Hyper-V sur leur ordinateur pour tester un logiciel sur plusieurs systèmes d’exploitation. Par exemple, si vous avez une application qui doit être testée sur un système d’exploitation Windows 8, Windows 7 et Linux, vous pouvez créer plusieurs machines virtuelles sur votre système de développement, chacune contenant chacun de ces systèmes d’exploitation.

- Vous pouvez utiliser Hyper-V sur Windows 10 pour résoudre les problèmes des machines virtuelles à partir de n’importe quel déploiement Hyper-V. Vous pouvez exporter une machine virtuelle à partir de votre environnement de production, l’ouvrir sur votre bureau exécutant Hyper-V, effectuer le dépannage requis, puis la replacer dans l’environnement de production.

- La mise en réseau virtuelle vous permet de créer un environnement avec plusieurs machines de test/développement/démonstration sans risquer d’affecter le réseau de production.

- Les passionnés peuvent utiliser Hyper-V pour essayer d’autres systèmes d’exploitation. Hyper-V permet d’afficher et de supprimer très facilement différents systèmes d’exploitation.

- Vous pouvez utiliser Hyper-V sur un ordinateur portable pour une démonstration des versions antérieures de Windows ou des systèmes d’exploitation autres que Windows.


## Configuration requise

Hyper-V nécessite un système 64 bits avec la traduction d’adresses de second niveau. La traduction d’adresses de second niveau est une fonctionnalité présente dans la génération actuelle de processeurs 64 bits Intel et AMD. Vous aurez également besoin d’une version 64 bits de Windows 8 ou supérieur et d’au moins 4 Go de RAM. Hyper-V prend en charge la création de systèmes d’exploitation 32 bits et 64 bits sur les machines virtuelles.

La mémoire dynamique d’Hyper-V permet d’allouer et de libérer la mémoire requise par la machine virtuelle dynamiquement (vous spécifiez un minimum et un maximum) ainsi que de partager la mémoire inutilisée entre les machines virtuelles. Vous pouvez exécuter 3 ou 4 machines virtuelles sur un ordinateur qui dispose de 4 Go de RAM, mais vous aurez besoin de davantage de RAM pour 5 machines virtuelles ou plus. En revanche, vous pouvez également créer des machines virtuelles de grande taille avec 32 processeurs et 512 Go de RAM, selon votre matériel physique.

## Systèmes d’exploitation que vous pouvez exécuter dans une machine virtuelle

Le terme « invité » fait référence à une machine virtuelle et « hôte » fait référence à l’ordinateur qui exécute la machine virtuelle. Hyper-V sur Windows prend en charge de nombreux systèmes d’exploitation invités différents, dont plusieurs versions de Linux, FreeBSD et Windows. Pour plus d’informations sur les systèmes d’exploitation pris en charge comme invités dans Hyper-V sur Windows, voir <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Systèmes d’exploitation invités Windows pris en charge</g><g id="2CapsExtId3" ctype="x-title"></g></g> et <g id="4CapsExtId1" ctype="x-link"><g id="4CapsExtId2" ctype="x-linkText">Machines virtuelles Linux et FreeBSD sur Hyper-V</g><g id="4CapsExtId3" ctype="x-title"></g></g>.

## Différences entre Hyper-V sur Windows et Hyper-V sur Windows Server

Certaines fonctionnalités se comportent différemment dans Hyper-V sur Windows et Hyper-V sur Windows Server. Notamment :

- Le modèle de gestion de mémoire est différent pour Hyper-V sur Windows. Sur un serveur, la mémoire Hyper-V est gérée en partant du principe que seules les machines virtuelles sont exécutées sur le serveur. Dans Hyper-V sur Windows, la mémoire est gérée en prévoyant que la plupart des machines clientes exécutent des logiciels en plus des machines virtuelles. Par exemple, un développeur peut exécuter Visual Studio ainsi que plusieurs machines virtuelles sur le même ordinateur.

- SR-IOV sur un invité 64 bits fonctionne normalement, mais la version 32 bits ne fonctionne pas et n’est pas prise en charge.

### Fonctionnalités Windows Server non disponibles dans Windows Hyper-V

Certaines fonctionnalités incluses dans Hyper-V sur Windows Server ne figurent pas dans Hyper-V sur Windows. Notamment :

- Virtualisation de GPU à l’aide de RemoteFX

- Migration dynamique des machines virtuelles d’un hôte vers un autre

- Réplication Hyper-V

- Fibre Channel virtuel

- Mise en réseau SR-IOV

- .VHDX partagé

> <g id="1" ctype="x-strong">Avertissement</g> : les machines virtuelles s’exécutant sur Hyper-V ne gèrent pas automatiquement le passage d’une connexion câblée à une connexion sans fil. Vous devez modifier les paramètres de carte réseau des machines virtuelles manuellement.

## Limitations

L’utilisation de la virtualisation présente certaines limitations. Les fonctionnalités ou applications qui dépendent d’un matériel spécifique ne fonctionnent pas correctement dans une machine virtuelle. Par exemple, les jeux ou applications qui nécessitent un traitement avec des GPU (sans fournir de procédure de secours logicielle) risquent de ne pas fonctionner correctement. Par ailleurs, les applications basées sur les minuteurs sous 10 ms, telles que les applications de haute précision sensibles à la latence comme les applications de mixage de concert notamment, peuvent être confrontées à des problèmes d’exécution dans une machine virtuelle.

En outre, si la virtualisation est activée, les applications de haute précision sensibles à la latence peuvent également être confrontées à des problèmes d’exécution dans le système d’exploitation hôte. (En effet, quand la virtualisation est activée, le système d’exploitation hôte est également exécuté sur la couche de virtualisation Hyper-V, tout comme les systèmes d’exploitation invités. Toutefois, contrairement aux invités, le système d’exploitation hôte présente la particularité d’avoir un accès direct à l’ensemble du matériel, ce qui signifie que les applications avec une configuration matérielle requise spéciale continuent de fonctionner sans problème dans le système d’exploitation hôte.)

À titre de rappel, vous devez avoir une licence valide pour les systèmes d’exploitation que vous utilisez dans les machines virtuelles.

## Étape suivante

<g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Procédure pas à pas : Hyper-V sur Windows 10</g><g id="1CapsExtId3" ctype="x-title"></g></g>

Consultez <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Nouveautés</g><g id="2CapsExtId3" ctype="x-title"></g></g> dans Hyper-V sur Windows 10.







<!--HONumber=May16_HO2-->



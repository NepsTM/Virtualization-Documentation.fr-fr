---
title: Configuration requise pour Hyper-V sur Windows10
description: Configuration requise pour Hyper-V sur Windows10
keywords: Windows10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6e5e6b01-7a9d-4123-8cc7-f986e10cd372
ms.openlocfilehash: 8f7e609e1e7c23181bed64e45c9c6160e425d4b6
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/21/2017
---
# Configuration requise pour Hyper-V sur Windows10

Hyper-V est disponible sur les versions 64bits des éditions Professionnel, Entreprise et Éducation de Windows8 et versions ultérieures.  Hyper-V nécessite la technologie SLAT (Second Level Address Translation), présente dans la génération actuelle de processeurs 64bits Intel et AMD.

Vous pouvez exécuter trois ou quatre machines virtuelles de base sur un hôte disposant de 4Go de RAM. Pour exécuter plus de machines virtuelles, davantage de ressources sont nécessaires. En revanche, vous pouvez créer des machines virtuelles de grande taille avec 32processeurs et 512Go de RAM, selon votre matériel physique.

## Configuration requise pour le système d’exploitation

Le rôle Hyper-V peut être activé sur ces versions de Windows10:

- Windows10 Entreprise
- Windows10 Professionnel
- Windows10 Éducation

Le rôle Hyper-V **ne peut pas** être installé sur:

- Windows10 Famille
- Windows10 Mobile
- Windows10 Mobile Entreprise

>Windows10 Famille peut être mis à niveau vers Windows10 Professionnel. Pour cela, ouvrez **Paramètres** > **Mise à jour et sécurité** > **Activation**. Vous pouvez alors visiter le magasin et acheter la mise à niveau.

## Configuration matérielle requise

Bien que ce document ne fournisse pas la liste complète du matériel compatible avec Hyper-V, vous devez disposer des éléments suivants:
    
- Processeur 64 bits avec traduction d’adresse de second niveau (SLAT).
- Processeur prenant en charge les extensions de mode du moniteur de machine virtuelle (VT-c sur les processeurs Intel).
- Au minimum 4Go de mémoire. Étant donné que les machines virtuelles partagent la mémoire avec l’hôte Hyper-V, vous devez fournir suffisamment de mémoire pour gérer la charge de travail virtuelle attendue.

Les éléments suivants doivent être activés dans le BIOS du système:
- Technologie de virtualisation (cette appellation peut varier selon le fabricant de la carte mère).
- Prévention de l’exécution des données appliquée par le matériel.

## Vérifier la compatibilité matérielle

Pour vérifier la compatibilité, ouvrez PowerShell ou une invite de commandes (cmd.exe), puis tapez **systeminfo**. Si tous les éléments de la configuration requise pour Hyper-V ont la valeur **Oui**, votre système peut exécuter le rôle Hyper-V. Si l’un des éléments a la valeur **Non**, passez en revue la configuration requise figurant dans ce document et apportez des ajustements dans la mesure du possible.

![](media/SystemInfo-upd.png)

Si vous exécutez **systeminfo** sur un hôte Hyper-V existant, la section Configuration requise pour Hyper-V indique ce qui suit:

```
Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V are not be displayed.
```
---
title: &800776548 Configuration requise pour Hyper-V sur Windows 10
description: Configuration requise pour Hyper-V sur Windows 10
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &635667630 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6e5e6b01-7a9d-4123-8cc7-f986e10cd372
---

# Configuration requise pour Hyper-V sur Windows 10

Pour faire fonctionner Hyper-V sur Windows 10, votre matériel et votre système d’exploitation doivent répondre à un ensemble spécifique de configurations. Ce document présente la configuration requise pour Hyper-V, et indique comment vous pouvez vérifier la compatibilité de votre système.

## Systèmes d'exploitation requis

Le rôle Hyper-V peut être activé sur ces versions de Windows 10 :

- Windows 10 Entreprise
- Windows 10 Professionnel
- Windows 10 Éducation

Le rôle Hyper-V <g id="2" ctype="x-strong">ne peut pas</g> être installé sur :

- Windows 10 Famille
- Windows 10 Mobile
- Windows 10 Mobile Entreprise

>Windows 10 Famille peut être mis à niveau vers Windows 10 Professionnel. Pour cela, ouvrez <g id="2" ctype="x-strong">Paramètres</g> > <g id="4" ctype="x-strong">Mise à jour et sécurité</g> > <g id="6" ctype="x-strong">Activation</g>. Vous pouvez alors visiter le magasin et acheter la mise à niveau.

## Configuration matérielle requise

Bien que ce document ne fournisse pas la liste complète du matériel compatible avec Hyper-V, vous devez disposer des éléments suivants :

- Processeur 64 bits avec traduction d’adresse de second niveau (SLAT).
- Processeur prenant en charge les extensions de mode du moniteur de machine virtuelle (VT-c sur les processeurs Intel).
- Au minimum 4 Go de mémoire. Étant donné que les machines virtuelles partagent la mémoire avec l’hôte Hyper-V, vous devez fournir suffisamment de mémoire pour gérer la charge de travail virtuelle attendue.

Les éléments suivants doivent être activés dans le BIOS système :
- Technologie de virtualisation (cette appellation peut varier selon le fabricant de la carte mère).
- Prévention de l’exécution des données appliquée par le matériel.

## Vérifier la compatibilité matérielle

Pour vérifier la compatibilité, ouvrez PowerShell ou une invite de commandes (cmd.exe), puis tapez <g id="2" ctype="x-strong">systeminfo.exe</g>. Si tous les éléments de la configuration requise pour Hyper-V ont la valeur <g id="2" ctype="x-strong">Oui</g>, votre système peut exécuter le rôle Hyper-V. Si l’un des éléments a la valeur <g id="2" ctype="x-strong">Non</g>, passez en revue la configuration requise figurant dans ce document et apportez des ajustements dans la mesure du possible.

<g id="1" ctype="x-linkText"></g>

Si vous exécutez <g id="2" ctype="x-strong">systeminfo.exe</g> sur un hôte Hyper-V existant, la section Configuration requise pour Hyper-V indique ce qui suit :

```
Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V are not be displayed.
```

## Étape suivante : installer Hyper-V

<g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Installer Hyper-V</g><g id="1CapsExtId3" ctype="x-title"></g></g>






<!--HONumber=May16_HO1-->



---
title: Extraits de code PowerShell
description: Extraits de code PowerShell
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: dc33c703-c5bc-434e-893b-0c0976b7cb88
translationtype: Human Translation
ms.sourcegitcommit: e14ede0a2b13de08cea0a955b37a21a150fb88cf
ms.openlocfilehash: 4b8a6905e3497b5fbecf938ea35b6cc57ae37be2

---

# Extraits de code PowerShell

PowerShell est un outil impressionnant de script, de gestion et d’automatisation pour Hyper-V.  Voici une boîte à outils pour explorer certains des avantages qu’il offre.

Toute la gestion Hyper-V nécessite des exécutions en tant qu’administrateur. Tous les scripts et extraits de code doivent donc être exécutés en tant qu’administrateur à partir d’un compte administrateur Hyper-V.

Si vous n’êtes pas sûr de disposer des autorisations appropriées, tapez `Get-VM`. Si la commande s’exécute sans erreur, vous pouvez continuer.


## Outils PowerShell Direct
Tous les scripts et les extraits de code de cette section s’appuient sur les principes de base suivants.

**Configuration requise** :  
*  PowerShell Direct.  Système d’exploitation Windows 10 invité et hôte.

**Variables communes** :  
`$VMName` chaîne contenant le nom de la machine virtuelle.  Afficher la liste des machines virtuelles disponibles avec `Get-VM`  
`$cred` informations d’identification pour le système d’exploitation invité.  Peut être rempli à l’aide de la commande `$cred = Get-Credential`  

### Vérifier que l’invité a démarré

Le Gestionnaire Hyper-V ne vous donne pas de visibilité sur le système d’exploitation invité. Il est donc souvent difficile de savoir si le système d’exploitation invité a démarré ou non.

Voici deux vues de la même fonctionnalité, d’abord en tant qu’extrait de code, puis en tant que fonction PowerShell.

Extrait de code :  
``` PowerShell
if((Invoke-Command -VMName $VMName -Credential $cred {"Test"}) -ne "Test"){Write-Host "Not Booted"} else {Write-Host "Booted"}
```  

Function:  
``` PowerShell
function waitForPSDirect([string]$VMName, $cred){
   Write-Output "[$($VMName)]:: Waiting for PowerShell Direct (using $($cred.username))"
   while ((Invoke-Command -VMName $VMName -Credential $cred {"Test"} -ea SilentlyContinue) -ne "Test") {Sleep -Seconds 1}}
```

**Résultat**  
Imprime un message convivial et applique un verrouillage tant que la connexion à la machine virtuelle n’est pas établie.  
Réussite en mode silencieux.

### Verrouillage des scripts tant que l’invité n’a pas de réseau
Avec PowerShell Direct, vous pouvez vous connecter à une session PowerShell à partir d’une machine virtuelle avant que celle-ci ait reçu une adresse IP.

``` PowerShell
# Wait for DHCP
while ((Get-NetIPAddress | ? AddressFamily -eq IPv4 | ? IPAddress -ne 127.0.0.1).SuffixOrigin -ne "Dhcp") {sleep -Milliseconds 10}
```

** Résultat ** Applique un verrouillage tant qu’un bail DHCP n’a pas été reçu.  Étant donné que ce script ne recherche pas de sous-réseau ou d’adresse IP spécifique, il fonctionne indépendamment de la configuration réseau que vous utilisez.  
Réussite en mode silencieux.

## Gestion des informations d’identification avec PowerShell
Les scripts Hyper-V doivent souvent traiter des informations d’identification pour un ou plusieurs hôtes Hyper-V, machines virtuelles ou les deux.

Il existe plusieurs méthodes pour y parvenir avec PowerShell Direct ou la communication à distance PowerShell standard :

1. La première méthode, qui est aussi la plus simple, suppose que les mêmes informations d’identification d’utilisateur soient valides dans l’hôte et l’invité, ou dans l’hôte local et l’hôte distant.  
  C’est une opération assez facile si vous ouvrez une session avec votre compte Microsoft, ou si vous êtes dans un environnement de domaine.  
  Dans ce scénario, vous pouvez simplement exécuter la commande `Invoke-Command -VMName "test" {get-process}`.

2. Laissez PowerShell vous demander vos informations d’identification  
  Si vos informations d’identification ne correspondent pas, vous obtenez automatiquement une invite vous demandant vos informations d’identification. Vous pouvez alors indiquer les informations d’identification appropriées pour la machine virtuelle.

3. Stockez les informations d’identification dans une variable pour pouvoir les réutiliser.
  Exécutez une commande simple comme la suivante :  
  ``` PowerShell
  $localCred = Get-Credential
   ```
  Puis exécuter une commande de ce type :
  ``` PowerShell
  Invoke-Command -VMName "test" -Credential $localCred  {get-process} 
  ```
  Signifie que vous êtes invité à indiquer vos informations d’identification qu’une seule fois par script/session PowerShell.

4. Codez vos informations d’identification dans vos scripts.  **Ne le faites pas pour une charge de travail ou un système réels**
 > Avertissement : _N’utilisez pas cette méthode dans un système de production.  Ne l’utilisez pas avec les mots de passe réels._
  
  Vous pouvez créer un objet PSCredential avec du code comme suit :  
  ``` PowerShell
  $localCred = New-Object -typename System.Management.Automation.PSCredential -argumentlist "Administrator", (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) 
  ```
  Manifestement non sécurisé, mais utile pour les tests.  De cette façon, vous n’êtes pas invité à indiquer vos informations d’identification pendant la session. 




<!--HONumber=Jun16_HO4-->



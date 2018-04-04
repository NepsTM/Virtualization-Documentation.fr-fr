---
title: Gérer des machines virtuelles Windows avec PowerShell Direct
description: Gérer des machines virtuelles Windows avec PowerShell Direct
keywords: windows 10, hyper-v, powershell, services d’intégration, composants d’intégration, automation, powershell direct
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: fb228e06-e284-45c0-b6e6-e7b0217c3a49
ms.openlocfilehash: 779dcf51d4903c9467cc52dbadb865beb9929bd2
ms.sourcegitcommit: e7fa38bcb7744a34e7a58978b55af1fbf6353247
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/20/2018
---
# <a name="virtual-machine-automation-and-management-using-powershell"></a>Gestion et automatisation de machines virtuelles avec PowerShell
 
PowerShell Direct vous permet d’exécuter du code PowerShell arbitraire dans une machine virtuelle Windows10 ou Windows Server2016 à partir de votre hôte Hyper-V, indépendamment de la configuration réseau ou des paramètres de gestion à distance.

**Façons d’exécuter PowerShell Direct:**  
* En tant que session interactive: [cliquez ici](#create-and-exit-an-interactive-powershell-session) pour créer et quitter une session PowerShell interactive à l’aide d’Enter-PSSession.
* En tant que session à usage unique pour exécuter une seule commande ou un seul script: [cliquez ici](#run-a-script-or-command-with-invoke-command) pour exécuter un script ou une commande à l’aide d’Invoke-Command.
* En tant que session persistante (build14280 et ultérieures): [cliquez ici](#copy-files-with-new-pssession-and-copy-item) pour créer une session persistante à l’aide de New-PSSession.  
Poursuivez en copiant un fichier vers et à partir de la machine virtuelle à l’aide de la commande Copy-Item, puis déconnectez-vous à l’aide de Remove-PSSession.

## <a name="requirements"></a>Configuration requise
**Configuration requise pour le système d’exploitation:**
* Hôte: Windows10, Windows Server2016 ou version ultérieure exécutant Hyper-V.
* Invité/Machine virtuelle: Windows10, Windows Server2016 ou version ultérieure.

Si vous gérez des machines virtuelles plus anciennes, utilisez Connexion à une machine virtuelle (VMConnect) ou [configurez un réseau virtuel pour la machine virtuelle](http://technet.microsoft.com/library/cc816585.aspx). 

**Configuration requise:**    
* La machine virtuelle doit être en cours d’exécution localement sur l’hôte.
* La machine virtuelle doit être allumée et s’exécuter avec au moins un profil utilisateur configuré.
* Vous devez être connecté à l’ordinateur hôte en tant qu’administrateur Hyper-V.
* Vous devez fournir des informations d’identification utilisateur valides pour la machine virtuelle.

-------------

## <a name="create-and-exit-an-interactive-powershell-session"></a>Créer et quitter une session PowerShell interactive

Pour exécuter des commandes PowerShell dans une machine virtuelle, le plus simple consiste à démarrer une session interactive.

Quand la session démarre, les commandes que vous tapez s’exécutent sur la machine virtuelle, comme si vous les aviez tapées directement dans une session PowerShell sur la machine virtuelle elle-même.

**Pour démarrer une session interactive:**

1. Sur l’hôte Hyper-V, ouvrez PowerShell en tant qu’administrateur.

2. Exécutez l’une des commandes suivantes pour créer une session interactive en utilisant le nom ou le GUID de la machine virtuelle:  
  
  ``` PowerShell
  Enter-PSSession -VMName <VMName>
  Enter-PSSession -VMId <VMId>
  ```
  
  Quand vous y êtes invité, fournissez les informations d’identification de la machine virtuelle.

3. Exécutez les commandes sur votre machine virtuelle.
  
  La valeur VMName doit s’afficher comme préfixe de votre invite de commandes PowerShell, comme indiqué ici:
  
  ``` 
  [VMName]: PS C:\>
  ```
  
  Toute exécution de commande s’effectue sur votre machine virtuelle. Pour effectuer un test, vous pouvez exécuter `ipconfig` ou `hostname` pour vérifier que ces commandes sont exécutées dans la machine virtuelle.
  
4. Une fois terminé, exécutez la commande suivante pour fermer la session:  
  
   ``` PowerShell
   Exit-PSSession 
   ``` 

> Remarque: Si votre session ne se connecte pas, voir la section de [dépannage](#troubleshooting) pour déterminer les causes potentielles. 

Pour en savoir plus sur ces applets de commande, consultez [Enter-PSSession](http://technet.microsoft.com/library/hh849707.aspx) et [Exit-PSSession](http://technet.microsoft.com/library/hh849743.aspx). 

-------------

## <a name="run-a-script-or-command-with-invoke-command"></a>Exécuter un script ou une commande avec Invoke-Command

PowerShell Direct avec Invoke-Command est parfait pour les situations où vous devez exécuter une commande ou un script sur une machine virtuelle, mais où vous n’avez pas besoin de continuer d’interagir avec la machine virtuelle au-delà de ce point.

**Pour exécuter une commande unique:**

1. Sur l’hôte Hyper-V, ouvrez PowerShell en tant qu’administrateur.

2. Exécutez l’une des commandes suivantes pour créer une session en utilisant le nom ou le GUID de la machine virtuelle:  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -ScriptBlock { command } 
   Invoke-Command -VMId <VMId> -ScriptBlock { command }
   ```
   
   Quand vous y êtes invité, fournissez les informations d’identification de la machine virtuelle.
   
   La commande s’exécute sur la machine virtuelle. S’il y a une sortie sur la console, elle est imprimée sur votre console.  La connexion est automatiquement fermée dès que la commande s’exécute.
   
   
**Pour exécuter un script:**

1. Sur l’hôte Hyper-V, ouvrez PowerShell en tant qu’administrateur.

2. Exécutez l’une des commandes suivantes pour créer une session en utilisant le nom ou le GUID de la machine virtuelle:  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -FilePath C:\host\script_path\script.ps1 
   Invoke-Command -VMId <VMId> -FilePath C:\host\script_path\script.ps1 
   ```
   
   Quand vous y êtes invité, fournissez les informations d’identification de la machine virtuelle.
   
   Le script s’exécute sur la machine virtuelle.  La connexion est automatiquement fermée dès que la commande s’exécute.

Pour en savoir plus sur cette applet de commande, voir [Invoke-Command](http://technet.microsoft.com/library/hh849719.aspx). 

-------------

## <a name="copy-files-with-new-pssession-and-copy-item"></a>Copier des fichiers avec New-PSSession et Copy-Item

> **Remarque:** PowerShell Direct ne prend en charge les sessions persistantes que dans Windows build14280 et ultérieures.

Les sessions persistantes PowerShell sont extrêmement utiles lors de l’écriture de scripts qui coordonnent des actions sur un ou plusieurs ordinateurs distants.  Une fois créées, les sessions persistantes existent en arrière-plan jusqu’à ce que vous décidiez de les supprimer.  Cela signifie que vous pouvez référencer la même session indéfiniment avec `Invoke-Command` ou `Enter-PSSession` sans passer d’informations d’identification.

De la même manière, les sessions conservent l’état.  Étant donné que les sessions persistantes sont maintenues, toutes les variables créées dans une session ou passées à une session sont conservées entre plusieurs appels. Un certain nombre d’outils sont disponibles pour travailler avec les sessions persistantes.  Pour cet exemple, nous allons utiliser [New-PSSession](https://technet.microsoft.com/en-us/library/hh849717.aspx) et [Copy-Item](https://technet.microsoft.com/en-us/library/hh849793.aspx) pour déplacer des données de l’hôte vers une machine virtuelle ou d’une machine virtuelle vers l’hôte.

**Pour créer une session, copiez des fichiers:**  

1. Sur l’hôte Hyper-V, ouvrez PowerShell en tant qu’administrateur.

2. Exécutez l’une des commandes suivantes pour créer une session PowerShell persistante sur la machine virtuelle à l’aide de `New-PSSession`.
  
  ``` PowerShell
  $s = New-PSSession -VMName <VMName> -Credential (Get-Credential)
  $s = New-PSSession -VMId <VMId> -Credential (Get-Credential)
  ```
  
  Quand vous y êtes invité, fournissez les informations d’identification de la machine virtuelle.
  
  > **Avertissement:**  
   Il y a un bogue dans les builds antérieures à 14500.  Si les informations d’identification ne sont pas explicitement spécifiées avec l’indicateur `-Credential`, le service dans l’invité se bloque et doit être redémarré.  Si vous rencontrez ce problème, des instructions de contournement sont disponibles [ici](#error-a-remote-session-might-have-ended).
  
3. Copiez un fichier sur la machine virtuelle.
  
  Pour copier `C:\host_path\data.txt` sur la machine virtuelle à partir de l’ordinateur hôte, exécutez:
  
  ``` PowerShell
  Copy-Item -ToSession $s -Path C:\host_path\data.txt -Destination C:\guest_path\
  ```
  
4.  Copiez un fichier à partir de la machine virtuelle (sur l’hôte). 
   
   Pour copier `C:\guest_path\data.txt` sur l’hôte à partir de la machine virtuelle, exécutez:
  
  ``` PowerShell
  Copy-Item -FromSession $s -Path C:\guest_path\data.txt -Destination C:\host_path\
  ```

5. Arrêtez la session persistante à l’aide de `Remove-PSSession`.
  
  ``` PowerShell 
  Remove-PSSession $s
  ```
  
-------------

## <a name="troubleshooting"></a>Résolution des problèmes

Les messages d’erreur liés à l’utilisation de PowerShell Direct sont peu nombreux.  Voici les plus courants, les causes possibles et les outils à utiliser pour diagnostiquer les problèmes.

### <a name="-vmname-or--vmid-parameters-dont-exist"></a>Les paramètres -VMName ou -VMID n’existent pas
**Problème:**  
`Enter-PSSession`, `Invoke-Command` ou `New-PSSession` n’ont pas de paramètre `-VMName` ou `-VMId`.

**Causes possibles:**  
Le problème le plus probable est que PowerShell Direct n’est pas pris en charge par votre système d’exploitation hôte.

Vous pouvez vérifier votre build Windows en exécutant la commande suivante:

``` PowerShell
[System.Environment]::OSVersion.Version
```

Si vous exécutez une build prise en charge, il est également possible que votre version de PowerShell n’exécute pas PowerShell Direct.  Pour PowerShell Direct et JEA, la version majeure doit être la version5 ou ultérieures.

Vous pouvez vérifier votre numéro de build PowerShell en exécutant la commande suivante:

``` PowerShell
$PSVersionTable.PSVersion
```


### <a name="error-a-remote-session-might-have-ended"></a>Erreur: une session à distance a peut-être pris fin.
> **Remarque:**  
Pour Enter-PSSession entre les builds d’hôte 10240 et 12400, toutes les erreurs ci-dessous sont signalées sous la forme «Une session à distance a peut-être pris fin».

**Message d’erreur:**
```
Enter-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**Causes possibles:**
* La machine virtuelle existe mais n’est pas en cours d’exécution.
* Le système d’exploitation invité ne prend pas en charge PowerShell Direct (voir la [configuration requise](#Requirements)).
* PowerShell n’est pas encore disponible dans l’invité.
  * Le système d’exploitation n’est pas entièrement démarré.
  * Le système d’exploitation ne peut pas démarrer correctement.
  * Un événement de démarrage exige une entrée de l’utilisateur.

Vous pouvez utiliser l’applet de commande [Get-VM](http://technet.microsoft.com/library/hh848479.aspx) pour vérifier quelles machines virtuelles sont en cours d’exécution sur l’hôte.

**Message d’erreur:**  
```
New-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**Causes possibles:**
* Une des raisons répertoriées ci-dessus: elles sont également applicables à `New-PSSession`  
* Un bogue dans les builds actuelles dans lesquelles des informations d’identification doivent être passées explicitement avec `-Credential`.  Quand cela se produit, l’ensemble du service se bloque dans le système d’exploitation invité et doit être redémarré.  Vous pouvez vérifier si la session est toujours disponible avec Enter-PSSession.

Pour contourner le problème des informations d’identification, connectez-vous à la machine virtuelle avec VMConnect, ouvrez PowerShell, puis redémarrez le service vmicvmsession à l’aide de la commande PowerShell suivante:

``` PowerShell
Restart-Service -Name vmicvmsession
```

### <a name="error-parameter-set-cannot-be-resolved"></a>Erreur: le jeu de paramètres ne peut pas être résolu
**Message d’erreur:**  
``` 
Enter-PSSession : Parameter set cannot be resolved using the specified named parameters.
```

**Causes possibles:**  
* `-RunAsAdministrator` n’est pas pris en charge lors de la connexion à des machines virtuelles.
     
  Lors de la connexion à un conteneur Windows, l’indicateur `-RunAsAdministrator` autorise les connexions administrateur sans informations d’identification explicites.  Étant donné que les machines virtuelles ne donnent pas à l’hôte un accès administrateur implicite, vous devez entrer explicitement les informations d’identification.

Les informations d’identification d’administrateur peuvent être passées à la machine virtuelle avec le paramètre `-Credential` ou en les entrant manuellement quand vous y êtes invité.


### <a name="error-the-credential-is-invalid"></a>Erreur: Les informations d’identification ne sont pas valides.

**Message d’erreur:**  
```
Enter-PSSession : The credential is invalid.
```

**Causes possibles:** 
* Impossible de valider les informations d’identification de l’invité.
  * Les informations d’identification fournies sont incorrectes.
  * Aucun compte d’utilisateur n’existe dans l’invité (le système d’exploitation n’a pas encore démarré)
  * Si vous vous connectez en tant qu’administrateur: l’administrateur n’a pas été défini comme un utilisateur actif.  Pour en savoir plus, cliquez [ici](https://technet.microsoft.com/en-us/library/hh825104.aspx).
  
### <a name="error-the-input-vmname-parameter-does-not-resolve-to-any-virtual-machine"></a>Erreur: Impossible de résoudre le paramètre d’entrée VMName en une machine virtuelle quelconque.

**Message d’erreur:**  
```
Enter-PSSession : The input VMName parameter does not resolve to any virtual machine.
```

**Causes possibles:**  
* Vous n’êtes pas administrateur Hyper-V.  
* La machine virtuelle n’existe pas.

Vous pouvez utiliser l’applet de commande [Get-VM](http://technet.microsoft.com/library/hh848479.aspx) pour vérifier que les informations d’identification que vous utilisez ont le rôle d’administrateur Hyper-V et pour identifier les machines virtuelles démarrées et exécutées localement sur l’hôte.


-------------

## <a name="samples-and-user-guides"></a>Exemples et guides de l’utilisateur

PowerShell Direct prend en charge JEA (Just Enough Administration).  Consultez ce guide de l’utilisateur pour l’essayer.

Pour obtenir des exemples, consultez [GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93).

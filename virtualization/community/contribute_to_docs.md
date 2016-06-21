---
title: &508030939 Ressources de la communauté
description: Ressources de la communauté
keywords: windows 10, hyper-v, container, docker
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &1331152366 virtualization
ms.service: virtualization
ms.assetid: 731ed95a-ce13-4c6e-a450-49563bdc498c
---

# Contribution à la documentation

> <g id="1" ctype="x-strong">Remarque :</g> Pour pouvoir contribuer, vous devez avoir un compte <g id="3CapsExtId1" ctype="x-link"><g id="3CapsExtId2" ctype="x-linkText">GitHub</g><g id="3CapsExtId3" ctype="x-title"></g></g>.

## Modification d’un document existant

1. Recherchez le document à modifier.

2. Sélectionnez <g id="2" ctype="x-strong">Contribute to this topic</g>  
  <g id="1" ctype="x-linkText"></g>

  Cela vous redirige automatiquement vers le fichier markdown associé à ce fichier dans GitHub.

  Assurez-vous que vous êtes connecté à GitHub. Si ce n’est pas le cas, connectez-vous ou créez un compte GitHub.

  <g id="1" ctype="x-linkText"></g>

3. Sélectionnez l’icône d’édition pour modifier dans l’éditeur du navigateur.

  <g id="1" ctype="x-linkText"></g>

4. Apportez les modifications en ligne.

  Actions possibles :
  1. Modifier le fichier
  2. Prévisualiser les modifications
  3. Renommer le fichier (il est peu probable que vous souhaitiez le faire)

  <g id="1" ctype="x-linkText"></g>

5. Proposer vos modifications en tant que requête de tirage

  <g id="1" ctype="x-linkText"></g>

6. Réviser vos modifications

  <g id="1" ctype="x-strong">Ce que nous recherchons dans une requête de tirage</g>
  * La modification est correcte : elle représente réellement la technologie
  * L’orthographe et la grammaire sont correctes
  * Emplacement logique dans la documentation

  <g id="1" ctype="x-linkText"></g>

7. Créer une <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">requête de tirage</g><g id="2CapsExtId3" ctype="x-title"></g></g>

## Requêtes de tirage

La plupart des modifications sont validées par une requête de tirage. Une requête de tirage permet de réviser un ensemble de modifications avec plusieurs réviseurs, ainsi que de modifier et commenter le contenu actuel.


## Répliquer le dépôt et modifier localement

Pour le travail sur la documentation à long terme, clonez le dépôt localement et travaillez sur votre ordinateur.

Le guide suivant vous montre comment émuler ma configuration (celle de Sarah Cooley). Il existe de nombreuses autres configurations qui fonctionnent tout aussi bien.

> <g id="1" ctype="x-strong">Remarque :</g> Tous ces outils de documentation fonctionnent aussi bien sur Linux/OSX. Si vous souhaitez d’autres guides, demandez.

Ce guide est constitué de trois sections :
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Configurer Git</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * installation de Git
  * configuration initiale
  * réplication du dépôt de documentation
  * clonage de votre copie sur l’ordinateur local
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Gestion des informations d’identification de départ</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * informations sur la dissimulation des informations d’identification et l’assistant d’identification.
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Configurer l’environnement de documentation</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * installer VSCode
  * description de quelques fonctionnalités pratiques avec VSCode pour Git
  * effectuer une première validation.

### Configurer Git

1. Installer Git (sur Windows) à partir d’<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">ici</g><g id="2CapsExtId3" ctype="x-title"></g></g>

  Vous ne devez modifier qu’une seule valeur dans l’installation :

  <g id="1" ctype="x-strong">Ajustement de votre environnement PATH</g>
  Utiliser Git à partir de l’invite de commandes Windows

  <g id="1" ctype="x-linkText"></g>

  Vous pouvez ainsi utiliser des commandes Git dans la console PowerShell/n’importe quelle console Windows.

2. Configurer votre identité Git

  Ouvrez une fenêtre PowerShell et exécutez :

  ``` PowerShell
  git config --global user.name "User Name"
  git config --global user.email username@microsoft.com
  ```

  Git utilise ces valeurs pour étiqueter vos validations.

> Si l’erreur suivante s’affiche, Git n’est probablement pas installé correctement ou vous devez relancer PowerShell.
>    ``` PowerShell
>    Git : le terme « Git » n’est pas reconnu comme nom d’applet de commande, fonction, fichier de script ou programme exécutable. Vérifiez l’orthographe du nom, ou si un chemin existe, vérifiez qu’il est correct et réessayez.
>    ```

3. Configurer votre environnement Git

   Installez un assistant d’identification pour n’avoir à entrer le nom d’utilisateur et le mot de passe qu’une seule fois (au moins sur cet ordinateur).
   J’utilise cet <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">assistant d’identification Windows</g><g id="2CapsExtId3" ctype="x-title"></g></g> de base

   Une fois l’installation terminée, exécutez les commandes suivantes pour activer l’assistant d’identification et définir votre comportement de type Push :
   ```
   git config --global credential.helper manager
   git config --global push.default simple
   ```

   La première fois que vous devez vous authentifier auprès de GitHub, le système vous demande votre nom d’utilisateur et le code d’authentification à deux facteurs si vous l’avez activé.
   Exemple :
   ```
   C:\Users\plang\Source\Repos\Virtualization-Documentation [master]> git pull
   Please enter your GitHub credentials for https://github.com/
   username: plang@microsoft.com
   password:
   authcode (app): 562689
   ```
   Vous recevez automatiquement un <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">jeton d’accès personnel</g><g id="2CapsExtId3" ctype="x-title"></g></g> avec les autorisations appropriées sur GitHub,
   que vous stockez en lieu sûr sur l’ordinateur local. Aucune demande ne devrait vous être faite par la suite.

4. Répliquer le dépôt

5. Cloner le dépôt

  Git Clone effectue une copie locale du dépôt Git avec les raccordements appropriés pour la synchronisation avec d’autres clones du même dépôt.

  Par défaut, Git Clone crée un dossier avec le même nom que le dépôt dans le répertoire actif. Je conserve tous mes dépôts Git dans mon répertoire utilisateur. En savoir plus sur Git Clone <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">ici</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

  ``` PowerShell
  cd ~
  git clone https://github.com/Microsoft/Virtualization-Documentation.git
  ```

  Si l’opération a réussi, vous avez maintenant un dossier <g id="2" ctype="x-code">Virtualization-Documentation</g>.

  ``` PowerShell
  cd Virtualization-Documentation
  ```

5. [Facultatif] Configurer Posh-Git

  Posh-Git est un module PowerShell créé par la communauté qui facilite l’utilisation de Git dans PowerShell. Il ajoute la saisie semi-automatique via la touche Tab pour Git dans PowerShell et peut également faire en sorte que l’invite affiche des informations utiles sur la création de branches et l’état des fichiers. En savoir plus <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">ici</g><g id="2CapsExtId3" ctype="x-title"></g></g>. Vous pouvez installer Posh-Git en exécutant la commande suivante dans une console Administrateur PowerShell.

  ``` PowerShell
  Install-Module -Name posh-git
  ```

  Pour que Posh-Git fonctionne automatiquement chaque fois que vous démarrez PowerShell, ajoutez le code suivant à votre profil PowerShell (par exemple, <g id="2" ctype="x-code">%UserProfile%\My Documents\WindowsPowerShell\profile.ps1 </g>)

  ``` PowerShell
  Import-Module posh-git

  function global:prompt {
    $realLASTEXITCODE = $LASTEXITCODE

    Write-Host($pwd.ProviderPath) -nonewline

    Write-VcsStatus

    $global:LASTEXITCODE = $realLASTEXITCODE
    return "> "
  }
  ```

### Valider et dissimuler les informations d’identification

  Pour valider que le dépôt est correctement configuré, essayez d’extraire un nouveau contenu.

  ``` PowerShell
  git pull
  ```


### Configurer l’environnement d’édition Markdown

1. Téléchargez VSCode

6. Effectuez un test de validation. Si vos informations d’identification sont correctement dissimulées, tout devrait parfaitement fonctionner.









<!--HONumber=May16_HO1-->



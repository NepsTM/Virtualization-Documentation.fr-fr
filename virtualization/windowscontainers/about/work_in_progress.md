---
title: "Conteneurs Windows - Travail en cours"
description: "Conteneurs Windows - Travail en cours"
keywords: docker, containers
author: scooley
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5d9f1cb4-ffb3-433d-8096-b085113a9d7b
redirect_url: ../containers_welcome
translationtype: Human Translation
ms.sourcegitcommit: e3f5535594123f6b4f8931e41a91d92f3b837814
ms.openlocfilehash: 085bb8c0158aedf4270cf2423114ec1901af1ebd

---

# Travail en cours

Si votre problème n’est pas traité ici ou si vous avez des questions, faites-nous part de vos commentaires via le [forum](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).

-----------------------

## Fonctionnalité générale

### Correspondance du numéro de build entre le conteneur et l’hôte
Un conteneur Windows nécessite une image de système d’exploitation qui correspond à l’hôte de conteneur en matière de niveau de build et de correctif. Toute discordance peut aboutir à une instabilité et/ou un comportement imprévisible pour le conteneur et/ou l’hôte.

Si vous installez des mises à jour sur le système d’exploitation de l’hôte de conteneur Windows, vous devez mettre à jour l’image du système d’exploitation de base du conteneur pour disposer des mises à jour correspondantes.
<!-- Can we give examples of behavior or errors?  Makes it more searchable -->

**Solution de contournement :**   
Téléchargez et installez une image de base de conteneur correspondant à la version et au niveau de correctif du système d’exploitation de l’hôte de conteneur.

### Comportement par défaut du pare-feu
Dans un environnement comprenant un hôte de conteneur et des conteneurs, il n’y a que le pare-feu de l’hôte de conteneur. Toutes les règles de pare-feu configurées dans l’hôte de conteneur sont propagées à tous ses conteneurs.

### Les conteneurs Windows démarrent lentement
Si le démarrage de votre conteneur prend plus de 30 secondes, il est possible qu’il effectue plusieurs analyses antivirus en double.

De nombreuses solutions anti-programme malveillant, comme Windows Defender, peuvent inutilement analyser les fichiers figurant dans les images de conteneur, en particulier tous les fichiers binaires du système d’exploitation et les fichiers de l’image de système d’exploitation du conteneur.  Cela se produit chaque fois qu’un conteneur est créé. En effet, pour le logiciel anti-programme malveillant, tous les « fichiers du conteneur » ressemblent à des nouveaux fichiers qui n’ont pas encore été analysés.  Par conséquent, quand le processus dans le conteneur tente de lire ces fichiers, les composants du logiciel anti-programme malveillant les analysent avant d’autoriser l’accès aux fichiers.  En réalité, ces fichiers ont été déjà analysés quand l’image de conteneur a été importée ou extraite sur le serveur. À partir de Windows Server Technical Preview 5, l’infrastructure disponible permet aux solutions anti-programme malveillant, comme Windows Defender, de tenir compte des situations et d’agir en conséquence pour éviter plusieurs analyses. Les solutions anti-programme malveillant peuvent mettre à jour leur solution à l’aide des instructions fournies [ici](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers) et éviter plusieurs analyses. 

### La commande start/stop se solde parfois par un échec si la mémoire est limitée à moins de 48 Mo
Les conteneurs Windows se heurtent à des erreurs aléatoires et incohérentes quand la mémoire est limitée à moins de 48 Mo.

Si vous exécutez plusieurs fois les commandes PowerShell start et stop, des échecs se produisent lors du démarrage ou de l’arrêt.

```PowerShell
new-container "Test" -containerimagename "WindowsServerCore" -MaximumBytes 32MB
start-container test
stop-container test
```

**Solution de contournement :**  
Remplacez la valeur de la mémoire par 48 Mo. 


### Échec de start-container quand le nombre de processeurs est de 1 ou 2 sur une machine virtuelle à 4 cœurs

Les conteneurs Windows ne démarrent pas et génèrent l’erreur suivante :  
`failed to start: This operation returned because the timeout period expired. (0x800705B4).`

Cela se produit quand le nombre de processeurs est de 1 ou 2 sur une machine virtuelle à 4 cœurs.

``` PowerShell
new-container "Test2" -containerimagename "WindowsServerCore"
Set-ContainerProcessor -ContainerName test2 -Maximum 2
Start-Container test2

Start-Container : 'test2' failed to start.
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4).
'test2' failed to start. (Container ID 133E9DBB-CA23-4473-B49C-441C60ADCE44)
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4). (Container ID
133E9DBB-CA23-4473-B49C-441C60ADCE44)
At line:1 char:1
+ Start-Container test2
+ ~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationTimeout: (:) [Start-Container], VirtualizationException
    + FullyQualifiedErrorId : OperationTimeout,Microsoft.Containers.PowerShell.Cmdlets.StartContainer
PS C:\> Set-ContainerProcessor -ContainerName test2 -Maximum 3
PS C:\> Start-Container test2
```

**Solution de contournement :**  
Augmentez les processeurs accessibles au conteneur, ne spécifiez pas explicitement les processeurs accessibles au conteneur ou réduisez les processeurs accessibles à la machine virtuelle.

--------------------------

## Mise en réseau

### Isolation des compartiments réseau et implications
Chaque conteneur utilise un compartiment réseau pour assurer une isolation. Toutes les cartes réseau de conteneur (points de terminaison) attachés à un conteneur donné résident dans le même compartiment réseau. En fonction du mode de mise en réseau (pilote) utilisé, il peut arriver que vous ne puissiez pas à accéder à deux points de terminaison de conteneur différents à l’aide de la même adresse IP ou du même port. De plus, les règles de pare-feu Windows ne prennent pas en charge les compartiments ou les conteneurs. Par conséquent, toutes les règles de pare-feu ajoutées s’appliquent à tous les conteneurs situés sur l’hôte du conteneur, quel que soit le point de terminaison.

*** Mise en réseau transparente ***


*** Mise en réseau NAT *** Vous pouvez exposer plusieurs points de terminaison attachés à un seul conteneur à l’aide de règles de réacheminement de port NAT qui sont appliquées pour chaque point de terminaison. Ces règles de réacheminement doivent utiliser des ports externes différents (sur l’hôte du conteneur) en cas de mappage au même port interne (dans le conteneur).  Toutefois, comme indiqué ci-dessus, les règles de pare-feu ajoutées ont une portée globale sur l’hôte du conteneur.



### Les mappages NAT statiques peuvent entrer en conflit avec les mappages de ports via Docker
À compter de Windows Server Technical Preview 5, les règles de création de NAT et de mappage de port sont intégrées aux applets de commande *ContainerNetwork* et aux commandes Docker. Le service de réseau hôte (HNS, Host Network Service) Windows gère la NAT (traduction d’adresses réseau) pour votre compte. Toutefois, il est possible qu’un client externe essaie de créer une règle de mappage de port en double en utilisant la même NAT que celle créée par HNS.


Voici un exemple de conflit avec un mappage statique sur le port 80 et l’erreur signalée par Docker quand cela se produit.
```
C:\Users\Administrator>docker run -it -p 80:80 windowsservercore cmd
docker: Error response from daemon: failed to create endpoint berserk_bassi on network nat: hnsCall failed in Win32: The remote procedure call failed. (0x6be).
```


***Prévention*** En général, il est très improbable que cela se produise car HNS gère la NAT (traduction d’adresses réseau). Toutes les règles de mappage/de réacheminement de port doivent être créées à l’aide de `docker run -p <external>:<internal>` ou d’Add-ContainerNetworkAdapterStaticMapping. Toutefois, si les mappages ne sont pas automatiquement nettoyés par HNS, cette erreur peut être résolue en supprimant le mappage de port à l’aide de PowerShell. En effet, il n’y aura plus de conflit avec le port 80 provoqué dans l’exemple ci-dessus.
```powershell
Get-NetNatStaticMapping | ? ExternalPort -eq 80 | Remove-NetNatStaticMapping
```


### Les conteneurs Windows n’obtiennent pas d’adresses IP
Si vous vous connectez aux conteneurs Windows avec des commutateurs de machine virtuelle DHCP, il est possible que l’hôte de conteneur reçoive une adresse IP, mais pas les conteneurs.

Les conteneurs obtiennent une adresse IP APIPA 169.254.***.*** Adresse IP APIPA.

**Solution de contournement :** il s’agit d’un effet secondaire du partage du noyau.  Tous les conteneurs possèdent effectivement la même adresse MAC.

Activez l’usurpation des adresses MAC sur l’hôte physique qui héberge la machine virtuelle d’hôte de conteneur.

Pour cela, vous pouvez utiliser PowerShell.
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```
--------------------------

## Compatibilité des applications

Face aux nombreuses questions qui se posent sur le fonctionnement, ou non, des applications dans les conteneurs Windows, nous avons décidé de consacrer [un article](../reference/app_compat.md) aux informations de compatibilité des applications.

Quelques-uns des problèmes les plus courants sont également répertoriés ici.

### Les journaux des événements ne sont pas disponibles dans le conteneur

Les commandes PowerShell comme `get-eventlog Application` et les API qui interrogent le journal des événements retournent une erreur semblable à ceci :
```
get-eventlog : Cannot open log Application on machine .. Windows has not provided an error code.
At line:1 char:1
```

Pour résoudre ce problème, vous pouvez ajouter l’étape suivante à un fichier Dockerfile. La journalisation des événements sera activée pour les images créées quand cette étape est incluse.
```
RUN powershell.exe -command Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\WMI\Autologger\EventLog-Application Start 1
```


### Une erreur inattendue s’est produite dans un appel de méthode d’API à une instance localdb.
Une erreur inattendue s’est produite dans un appel de méthode d’API à une instance localdb.

### RTerm ne fonctionne pas
RTerm est installé, mais il ne démarre pas dans un conteneur Windows Server.

Erreur :  
```
'C:\Program' is not recognized as an internal or external command,
operable program or batch file.
```


### Conteneur : Visual C++ Runtime x64/x86 2015 n’est pas installé

Comportement observé : dans un conteneur :
```
C:\build\vcredist_2015_x64.exe /q /norestart
C:\build>echo %errorlevel%
0
C:\build>wmic product get
No Instance(s) Available.
```

Il s’agit d’un problème d’interopérabilité avec le filtre de déduplication. La déduplication vérifie la cible de renommage pour déterminer si le fichier est dédupliqué. La création échoue avec le message `STATUS_IO_REPARSE_TAG_NOT_HANDLED`, car le filtre de conteneur Windows Server se situe au-dessus de la déduplication.


Pour plus d’informations sur les applications pouvant être placées dans des conteneurs, voir l’[article sur la compatibilité des applications](../reference/app_compat.md).

--------------------------


## Gestion de Docker

### Les commandes Docker ne fonctionnent pas toutes
* Docker Exec échoue dans les conteneurs Hyper-V.

Si vous êtes confronté à un échec qui ne figure pas dans cette liste (ou si une commande échoue de manière imprévue), faites-le nous savoir via les [forums](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).

### Le collage de commandes dans une session Docker interactive est limité à 50 caractères
Le collage de commandes dans une session Docker interactive est limité à 50 caractères.  
Si vous copiez une ligne de commande dans une session Docker interactive, elle est actuellement limitée à 50 caractères. La chaîne collée est simplement tronquée.

Ce n’est pas délibéré, et nous travaillons à lever cette restriction.

### Erreurs net use
Net use retourne l’erreur système 1223 au lieu de demander le nom de l’utilisateur ou le mot de passe.

**Solution de contournement :**  
Spécifiez le nom de l’utilisateur et le mot de passe lors de l’exécution de net use.

``` PowerShell
net use S: \\your\sources\here /User:shareuser [yourpassword]
``` 


--------------------------



## Bureau à distance 

Il n’est pas possible de gérer les conteneurs Windows ni d’interagir avec eux par le biais d’une session RDP dans TP5.

--------------------------

### La sortie d’un conteneur dans un hôte de conteneur Nano Server n’est pas possible avec « exit »
Si vous essayez de quitter un conteneur qui se trouve dans un hôte de conteneur Nano Server avec « exit », vous êtes déconnecté de l’hôte de conteneur Nano Server, mais ne quittez pas le conteneur.

**Solution de contournement :** utilisez Exit-PSSession à la place pour quitter le conteneur.

N’hésitez pas à nous faire part de vos demandes de fonctionnalités via les [forums](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers). 


--------------------------


## Utilisateurs et domaines

### Utilisateurs locaux
Des comptes d’utilisateurs locaux peuvent être créés et utilisés pour l’exécution des services et applications Windows dans des conteneurs.


### Appartenance au domaine
Les conteneurs ne peuvent pas joindre des domaines Active Directory ni exécuter des services ou applications en tant qu’utilisateurs du domaine, comptes de service ni comptes d’ordinateurs. 

Les conteneurs sont conçus pour démarrer rapidement dans un état cohérent connu qui est en grande partie indépendant de l’environnement. La jonction à un domaine et l’application de paramètres de stratégie de groupe du domaine augmentent le temps nécessaire au démarrage d’un conteneur, modifient son mode de fonctionnement au fil du temps et limitent la possibilité de déplacer ou de partager des images entre les développeurs et les déploiements.

Nous examinons soigneusement les commentaires sur la façon dont les applications et services utilisent Active Directory et sur leur déploiement dans les conteneurs. Si vous avez plus d’informations sur ce qui vous convient le mieux, partagez-les avec nous dans les [forums](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers). 

Nous étudions activement des solutions pour prendre en charge ces types de scénarios.



<!--HONumber=Jul16_HO1-->



# Travail en cours

Si votre problème n’est pas traité ici ou si vous avez des questions, faites-nous part de vos commentaires via le [forum](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).

-----------------------


## Fonctionnalité générale

### Correspondance du numéro de build entre le conteneur et l’hôte

Un conteneur Windows nécessite une image de système d’exploitation qui correspond à l’hôte de conteneur en matière de niveau de build et de correctif. Toute discordance peut aboutir à une instabilité et/ou un comportement imprévisible pour le conteneur et/ou l’hôte.

Si vous installez des mises à jour sur le système d’exploitation de l’hôte de conteneur Windows, vous devez mettre à jour l’image du système d’exploitation de base du conteneur pour disposer des mises à jour correspondantes.


**Solution de contournement :**   
Téléchargez et installez une image de base de conteneur correspondant à la version et au niveau de correctif du système d’exploitation de l’hôte de conteneur.

### Tous les lecteurs autres que C:/ sont visibles dans les conteneurs

Tous les lecteurs autres que C:/ accessibles à l’hôte de conteneur sont automatiquement mappés à des nouveaux conteneurs Windows en cours d’exécution.

Pour l’instant, il n’existe aucun moyen de mapper des dossiers de manière sélective à un conteneur. En guise de solution de contournement provisoire, les lecteurs sont mappés automatiquement.

**Solution de contournement : **  
Nous y travaillons. À l’avenir, vous pourrez partager des dossiers.

### Comportement par défaut du pare-feu

Dans un environnement comprenant un hôte de conteneur et des conteneurs, il n’y a que le pare-feu de l’hôte de conteneur. Toutes les règles de pare-feu configurées dans l’hôte de conteneur sont propagées à tous ses conteneurs.

### Les conteneurs Windows démarrent lentement

Si le démarrage de votre conteneur prend plus de 30 secondes, il est possible qu’il effectue plusieurs analyses antivirus en double.

De nombreuses solutions anti-programme malveillant, comme Windows Defender, peuvent inutilement analyser les fichiers figurant dans les images de conteneur, en particulier les fichiers binaires du système d’exploitation et les fichiers de l’image du système d’exploitation du conteneur. Cela se produit chaque fois qu’un conteneur est créé. En effet, pour le logiciel anti-programme malveillant, tous les « fichiers du conteneur » ressemblent à des nouveaux fichiers qui n’ont pas encore été analysés. Par conséquent, quand le processus dans le conteneur tente de lire ces fichiers, les composants du logiciel anti-programme malveillant les analysent avant d’autoriser l’accès aux fichiers. En réalité, ces fichiers ont été déjà analysés quand l’image de conteneur a été importée ou extraite sur le serveur. Les versions préliminaires à venir bénéficieront d’une nouvelle infrastructure pour que les solutions anti-programme malveillant, comme Windows Defender, tiennent compte de ces situations et agissent en conséquence pour éviter plusieurs analyses.

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
« Échec du démarrage : cette opération s’est terminée car le délai d’attente a expiré (0x800705B4). »

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

### Compartiments réseau limités

Dans cette version, nous prenons en charge un compartiment réseau par conteneur. Cela signifie que si vous avez un conteneur avec plusieurs cartes réseau, vous ne pouvez pas accéder au même port réseau sur chaque carte (par exemple, 192.168.0.1:80 et 192.168.0.2:80 appartenant au même conteneur).

**Solution de contournement : **  
Si plusieurs points de terminaison doivent être exposés par un conteneur, utilisez le mappage de port NAT.

### Les conteneurs Windows n’obtiennent pas d’adresses IP

Si vous vous connectez aux conteneurs Windows avec des commutateurs de machine virtuelle DHCP, il est possible que l’hôte de conteneur reçoive une adresse IP, mais pas les conteneurs.

Les conteneurs obtiennent une adresse IP APIPA 169.254.***.***.

**Solution de contournement :**
Il s’agit d’un effet secondaire du partage du noyau. Tous les conteneurs possèdent effectivement la même adresse MAC.

Activez l’usurpation des adresses MAC sur l’hôte de conteneur.

Pour cela, vous pouvez utiliser PowerShell.
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```

--------------------------


## Compatibilité des applications

Face aux nombreuses questions qui se posent sur le fonctionnement ou non des applications dans les conteneurs Windows, nous avons décidé de consacrer [un article](../reference/app_compat.md) aux informations de compatibilité des applications.

Quelques-uns des problèmes les plus courants sont également répertoriés ici.

### Une erreur inattendue s’est produite dans un appel de méthode d’API à une instance localdb

Une erreur inattendue s’est produite dans un appel de méthode d’API à une instance localdb.

### RTerm ne fonctionne pas

RTerm est installé, mais il ne démarre pas dans un conteneur Windows Server.

Erreur :
```
'C:\Program' is not recognized as an internal or external command,
operable program or batch file.
```


### Conteneur : Visual C++ Runtime x64/x86 2015 n’est pas installé

Comportement observé :
Dans un conteneur :
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

### Clients docker non sécurisés

Dans cette version préliminaire, les communications Docker sont publiques si vous savez où regarder.

### Les commandes Docker ne fonctionnent pas toutes

Docker Exec échoue dans les conteneurs Hyper-V.

Les commandes liées à DockerHub ne sont pas encore prises en charge.

Si vous êtes confronté à un échec qui ne figure pas dans cette liste (ou si une commande échoue de manière imprévue), faites-le nous savoir via les [forums](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).

### Le collage de commandes dans une session Docker interactive est limitée à 50 caractères

Le collage de commandes dans une session Docker interactive est limitée à 50 caractères.  
Si vous copiez une ligne de commande dans une session Docker interactive, elle est actuellement limitée à 50 caractères. La chaîne collée est simplement tronquée.

Ce n’est pas délibéré, et nous travaillons à lever cette restriction.

### Erreurs net use

Net use retourne l’erreur système 1223 au lieu de demander le nom de l’utilisateur ou le mot de passe.

**Solution de contournement :**  
Spécifiez le nom de l’utilisateur et le mot de passe lors de l’exécution de net use.

``` PowerShell
net use S: \\your\sources\here /User:shareuser [yourpassword]
```


## Bureau à distance

Il n’est pas possible de gérer les conteneurs Windows ni d’interagir avec eux par le biais d’une session RDP dans TP4.

--------------------------


## Gestion de PowerShell

### Les sessions *-PSSession n’ont pas toutes un argument containerid

C’est exact. Nous prévoyons de prendre entièrement en charge cimsession à l’avenir.

N’hésitez pas à nous faire part de vos demandes de fonctionnalités via [les forums](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).





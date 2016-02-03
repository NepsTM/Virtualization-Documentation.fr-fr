# Docker et Windows

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.**

Docker est une plateforme de déploiement et de gestion de conteneurs, qui fonctionne à la fois avec les conteneurs Linux et Windows. Docker est utilisé pour créer, gérer et supprimer des conteneurs et des images de conteneur. Docker permet le stockage des images de conteneur dans un registre public (Docker Hub) et des registres privés (Docker Trusted Registries). Docker propose en outre des fonctionnalités de clustering de l’hôte de conteneur avec Docker Swarm et l’automatisation du déploiement avec Docker Compose. Pour plus d’informations sur Docker et l’ensemble d’outils Docker, visitez [Docker.com](https://www.docker.com/).

> La fonctionnalité de conteneur Windows doit être activée avant que Docker ne permette de créer et gérer des conteneurs Windows Server et Hyper-V. Pour obtenir des instructions sur l’activation de cette fonctionnalité, voir le [Guide de déploiement de l’hôte de conteneur](./docker_windows.md).

## Windows Server

### Installer Docker

Le démon Docker et l’interface de ligne de commande ne sont pas fournis avec Windows Server ni Windows Server Core, et ne sont pas installés avec la fonctionnalité de conteneur Windows. Docker doit être installé séparément. Ce document vous guide lors de l’installation manuelle du démon Docker et du client Docker. Des méthodes automatisées pour effectuer ces tâches sont également indiquées.

Le démon et l’interface de ligne de commande Docker ont été développés en langage Go. À ce stade, docker.exe n’est pas installé comme un service Windows. Plusieurs méthodes permettent de créer un service Windows : un exemple illustré ici utilise `nssm.exe`.

Téléchargez docker.exe à partir de `https://aka.ms/tp4/docker`, puis placez-le dans le répertoire System32 sur l’hôte du conteneur.

```powershell
PS C:\> wget https://aka.ms/tp4/docker -OutFile $env:SystemRoot\system32\docker.exe
```

Créez un répertoire nommé `c:\programdata\docker`. Dans ce répertoire, créez un fichier nommé `runDockerDaemon.cmd`.

```powershell
PS C:\> New-Item -ItemType File -Path C:\ProgramData\Docker\runDockerDaemon.cmd -Force
```

Copiez le texte suivant dans le fichier `runDockerDaemon.cmd`. Ce fichier de commandes démarre le démon Docker avec la commande `docker daemon –D –b  « commutateur virtuel »`. Remarque : le nom du commutateur virtuel dans ce fichier doit correspondre au nom de celui que les conteneurs utilisent pour la connectivité réseau.

```powershell
@echo off
set certs=%ProgramData%\docker\certs.d

if exist %ProgramData%\docker (goto :run)
mkdir %ProgramData%\docker

:run
if exist %certs%\server-cert.pem (goto :secure)

docker daemon -D -b "Virtual Switch"
goto :eof

:secure
docker daemon -D -b "Virtual Switch" -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
```
Téléchargez nssm.exe à partir de [https://nssm.cc/release/nssm-2.24.zip](https://nssm.cc/release/nssm-2.24.zip).

```powershell
PS C:\> wget https://nssm.cc/release/nssm-2.24.zip -OutFile $env:ALLUSERSPROFILE\nssm.zip
```

Extrayez les fichiers, puis copiez `nssm-2.24\win64\nssm.exe` dans le répertoire `c:\windows\system32`.

```powershell
PS C:\> Expand-Archive -Path $env:ALLUSERSPROFILE\nssm.zip $env:ALLUSERSPROFILE
PS C:\> Copy-Item $env:ALLUSERSPROFILE\nssm-2.24\win64\nssm.exe $env:SystemRoot\system32
```
Exécutez `nssm install` pour configurer le service Docker.

```powershell
PS C:\> start-process nssm install
```

Entrez les données suivantes dans les champs correspondants dans le programme d’installation du service NSSM.

Onglet Application :

- **Chemin d’accès :** C:\Windows\System32\cmd.exe

- **Répertoire de démarrage :** C:\Windows\System32

- **Arguments :** /s /c C:\ProgramData\docker\runDockerDaemon.cmd

- **Nom du service :** Docker

![](media/nssm1.png)

Onglet Détails :

- **Nom complet :** Docker

- **Description :** le démon Docker fournit des fonctionnalités de gestion de conteneurs pour les clients Docker.


![](media/nssm2.png)

Onglet E/S :

- **Sortie (stdout) :** C:\ProgramData\docker\daemon.log

- **Erreur (stderr) :** C:\ProgramData\docker\daemon.log


![](media/nssm3.png)

Quand vous avez terminé, cliquez sur le bouton `Installer le service`.

Une fois cette opération terminée, quand Windows démarre, le démon (service) Docker démarre également.

### Suppression de Docker

Si vous avez suivi ce guide pour la création d’un service Windows à partir de docke.exe, la commande suivante supprime le service.

```powershell
PS C:\> sc.exe delete Docker

[SC] DeleteService SUCESS
```

## Nano Server

### Installer Docker

Téléchargez docker.exe à partir de `https://aka.ms/tp4/docker` et copiez-le dans le dossier `windows\system32` de l’hôte du conteneur Nano Server.

Exécutez la commande ci-dessous pour démarrer le démon Docker. Cette commande doit être exécutée lors de chaque démarrage de l’hôte de conteneur. Cette commande démarre le démon Docker, spécifie un commutateur virtuel pour la connectivité du conteneur et configure le démon pour qu’il écoute les demandes Docker entrantes sur le port 2375. Dans cette configuration, Docker peut être géré à partir d’un ordinateur distant.

```powershell
PS C:\> start-process cmd "/k docker daemon -D -b <Switch Name> -H 0.0.0.0:2375”
```

### Suppression de Docker

Pour supprimer le démon et l’interface de ligne de commande Docker de Nano Server, supprimez `docker.exe` du répertoire Windows\system32.

```powershell
PS C:\> Remove-Item $env:SystemRoot\system32\docker.exe
```

### Session interactive Nano

> Pour plus d’informations sur la gestion à distance de Nano Server, voir [Prise en main de Nano Server](https://technet.microsoft.com/en-us/library/mt126167.aspx#bkmk_ManageRemote).

Vous pouvez recevoir cette erreur lors de la gestion interactive d’un conteneur sur un hôte Nano Server.

```powershell
docker : cannot enable tty mode on non tty input
+ CategoryInfo          : NotSpecified: (cannot enable tty mode on non tty input:String) [], RemoteException
+ FullyQualifiedErrorId : NativeCommandError 
```

Cela peut se produire quand vous tentez d’exécuter un conteneur avec une session interactive, en utilisant -it :

```powershell
Docker run -it <image> <command>
```
ou quand vous tentez l’attachement à un conteneur en cours d’exécution :

```powershell
Docker attach <container name>
```

Pour créer une session interactive avec un conteneur créé avec Docker sur un hôte Nano Server, le démon Docker doit être géré à distance. Pour ce faire, téléchargez docker.exe à partir de [cet emplacement](https://aka.ms/ContainerTools) et copiez-le sur un système distant.

Tout d’abord, vous devez configurer le démon Docker dans Nano Server pour écouter les commandes à distance. Pour cela, exécutez cette commande dans Nano Server :

```powershell
docker daemon -D -H <ip address of Nano Server>:2375
```

À présent, sur votre machine, ouvrez une session PowerShell ou CMD et exécutez les commandes Docker en spécifiant l’hôte distant avec `-H`.

```powershell
.\docker.exe -H tcp://<ip address of Nano Server>:2375
```

Par exemple, si vous souhaitez afficher les images disponibles :

```powershell
.\docker.exe -H tcp://<ip address of Nano Server>:2375 images
```




<!--HONumber=Jan16_HO3-->

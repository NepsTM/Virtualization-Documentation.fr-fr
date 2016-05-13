



# Comparaison entre Docker et PowerShell pour la gestion des conteneurs Windows

Il existe plusieurs méthodes pour gérer des conteneurs Windows à l’aide d’outils Windows intégrés (PowerShell, dans cette version préliminaire) et d’outils de gestion Open Source tels que Docker.  
Pour en savoir plus sur chaque type d’outils, voir :
* [Gérer les conteneurs Windows avec Docker](../quick_start/manage_docker.md)
* [Gérer les conteneurs Windows avec PowerShell](../quick_start/manage_powershell.md)

Cette page offre des informations de référence plus détaillées sur la comparaison entre les outils Docker et les outils de gestion PowerShell.

## Comparaison entre PowerShell pour les conteneurs et les machines virtuelles Hyper-V

Vous pouvez créer et exécuter des conteneurs Windows, et interagir avec eux, à l’aide d’applets de commande PowerShell. Tout ce dont vous avez besoin est fourni avec Windows.

Si vous avez utilisé Hyper-V PowerShell, vous devez déjà être à l’aise avec la conception des applets de commande. Une grande partie du flux de travail est le même que pour gérer une machine virtuelle avec le module Hyper-V. Les commandes `New-VM`, `Get-VM`, `Start-VM` et `Stop-VM` sont remplacées par les commandes `New-Container`, `Get-Container`, `Start-Container` et `Stop-Container`. Il existe quelques applets de commande et paramètres spécifiques des conteneurs, mais le cycle de vie général et la gestion d’un conteneur Windows ressemblent à peu près à ceux d’une machine virtuelle Hyper-V.

## Quelles sont les différences de gestion entre PowerShell et Docker ?

Les applets de commande PowerShell des conteneurs exposent une API qui n’est pas tout à fait identique à celle de Docker. En règle générale, les applets de commande fonctionnent de manière plus précise. Certaines commandes Docker ont des homologues assez directs dans PowerShell :

| Commande docker| Applet de commande PowerShell|
|----|----|
| `docker ps -a`| `Get-Container`|
| `docker images`| `Get-ContainerImage`|
| `docker rm`| `Remove-Container`|
| `docker rmi`| `Remove-ContainerImage`|
| `docker create`| `New-Container`|
| `docker commit <ID_conteneur>`| `New-ContainerImage -Container &lt;conteneur&gt;`|
| `docker load &lt;tarball&gt;`| `Import-ContainerImage <package_AppX>`|
| `docker save`| `Export-ContainerImage`|
| `docker start`| `Start-Container`|
| `docker stop`| `Stop-Container`|

Toutes les commandes Docker n’ont pas leur équivalent dans les applets de commande PowerShell, et il existe un certain nombre de commandes pour lesquelles nous ne fournissons pas de remplacements PowerShell* (notamment `docker build` et `docker cp`). Mais ce qui peut sauter aux yeux est qu’il n’existe aucun remplacement en une seule ligne pour `docker run`.

\* Susceptible d’être modifié.

### Mais nous ne pouvons pas nous passer de docker run ! Comment faire ?

Nous allons vous présenter quelques astuces qui permettent d’obtenir un modèle d’interaction plus familier pour les utilisateurs qui sont déjà à l’aise avec PowerShell. Bien entendu, si vous êtes habitué à la façon dont docker fonctionne, vous allez devoir changer un peu votre vision des choses.

1.  Le cycle de vie d’un conteneur dans le modèle PowerShell est légèrement différent. Dans le module PowerShell des conteneurs, nous présentons les opérations les plus granulaires de `New-Container` (qui crée un nouveau conteneur arrêté) et `Start-Container`.

  Entre la création et le démarrage du conteneur, vous pouvez également configurer les paramètres du conteneur. Pour TP3, la seule autre configuration que nous comptons exposer est la possibilité de définir la connexion réseau pour le conteneur à l’aide des applets de commande (Add/Remove/Connect/Disconnect/Get/Set)-ContainerNetworkAdapter.

2.  Vous ne pouvez actuellement pas transmettre une commande à exécuter à l’intérieur du conteneur au démarrage. Toutefois, vous pouvez toujours obtenir une session PowerShell interactive vers un conteneur en cours d’exécution à l’aide de la commande `Enter-PSSession -ContainerId <ID d’un conteneur en cours d’exécution>`, et vous pouvez exécuter une commande à l’intérieur d’un conteneur en cours d’exécution à l’aide de la commande `Invoke-Command -ContainerId <id de conteneur> -ScriptBlock { code à exécuter à l’intérieur du conteneur }` ou `Invoke-Command -ContainerId <id de conteneur> -FilePath <chemin du script>`.  
Ces deux commandes autorisent l’indicateur `-RunAsAdministrator` facultatif pour les actions qui nécessitent des privilèges élevés.


## Problèmes connus et avertissements

1.  Pour l’instant, il n’y a aucune communication entre les applets de commande de conteneurs et les conteneurs ou images créés avec Docker, et inversement, Docker n’a aucun lien avec les conteneurs et images créés avec PowerShell. Si vous avez créé un conteneur dans Docker, vous devez le gérer avec Docker. Si vous l’avez créé avec PowerShell, vous devez le gérer avec PowerShell.

2.  Il nous reste un peu de travail pour améliorer l’expérience de l’utilisateur final : les messages d’erreur, le signalement de la progression, les chaînes d’événements non valides, etc. Si vous vous trouvez dans une situation où vous aimeriez avoir plus d’informations, n’hésitez pas à envoyer vos suggestions sur les forums.

## Révision rapide

Voici une présentation de certains flux de travail courants.

Nous supposons que vous avez installé une image du système d’exploitation de conteneur nommée « ServerDatacenterCore » et créé un commutateur virtuel nommé « Virtual Switch » (à l’aide de New-VMSwitch).

``` PowerShell
### 1. Enumerating images
# At this point, you can enumerate the images on the system:
Get-ContainerImage

# Get-ContainerImage also accepts filters.
# For example, this will return all container images whose Name starts with S (case-insensitive):
Get-ContainerImage -Name S*

# You can save the results of this to a variable.
# (If you're not familiar with PowerShell, the "$" denotes a variable.)
$baseImage = Get-ContainerImage -Name ServerDatacenterCore
$baseImage

### 2. Creating and enumerating containers
# Now, we can create a new container using this image:
New-Container -Name "MyContainer" -ContainerImage $baseImage -SwitchName "Virtual Switch"

# Now we can enumerate all containers.
Get-Container

# Similarly, we can save this container to a variable:
$container1 = Get-Container -Name "MyContainer"

### 3. Starting containers, interacting with running containers, and stopping containers
# Now let's go ahead and start the container.
Start-Container -Name "MyContainer"

# (We could've also started this container using "Start-Container -Container $container1".)

# With the container now running, let's go ahead and enter an interactive PowerShell session:
Enter-PSSession -ContainerId $container1.Id

# This should eventually bring up a PowerShell prompt from inside the container.
# You can try all the things that you did in the interactive cmd prompt given by "docker run -it".
# For now, just to prove we've been here, we can create a new file:
cd \
mkdir Test
cd Test
echo "hello world" > hello.txt
exit

# Now we should be back in the outside world. Even though we've exited the PowerShell session,
# the container itself is still running, as you can see by printing out the container again:
$container1

# Before we can commit this container to a new image, we need to stop the container.
# Let's do that now.
Stop-Container -Container $container1

### 4. Creating a new container image
# And now let's commit it to a new image.
$image1 = New-ContainerImage -Container $container1 -Publisher Test -Name Image1 -Version 1.0

# Enumerate all the images again, for sanity's sake:
Get-ContainerImage

# Rinse and repeat! Make another container based on the new image.
$container2 = New-Container -Name "MySecondContainer" -ContainerImage $image1 -SwitchName "Virtual Switch"

# (If you like, you can start the second container and verify that the new file
# "\Test\hello.txt" is there as expected.)

### 5. Removing a container
# The first container we created is now stopped. Let's get rid of it:
Remove-Container -Container $container1

# And confirm that it's gone:
Get-Container

### 6. Exporting, removing, and importing images
# For images that aren't the base OS image, we can export them into an .appx package file.
Export-ContainerImage -Image $image1 -Path "C:\exports"
# This should create a .appx file in the C:\exports folder.
# If you've given your image the same publisher, name, and version we used earlier,
# you'd expect the resulting .appx to be named "CN=Test_Image1_1.0.0.0.appx".

# Before we can try importing the image again, we need to remove the image.
# (If you have any running containers that depend on this image, you'll want to stop them first.)
Remove-ContainerImage -Image $image1

# Now let's import the image again:
Import-ContainerImage -Path C:\exports\CN=Test_Image1_1.0.0.0.appx

# We'd previously created a container dependent on this image. You should be able to start it:
Start-Container -Container $container2 
```

### Créer votre propre exemple

Vous pouvez afficher toutes les applets de commande de conteneurs à l’aide de la commande `Get-Command -Module Containers`. Il existe plusieurs autres applets de commande qui ne sont pas décrites ici et que vous pouvez découvrir par vous-même.    
**Remarque** Cette commande ne retourne pas les applets de commande `Enter-PSSession` et `Invoke-Command`, qui font partie du noyau PowerShell.

Vous pouvez également obtenir de l’aide sur toutes les applets de commande en utilisant la commande `Get-Help [nom de l’applet de commande]` ou `[nom de l’applet de commande] -?`. Aujourd’hui, la sortie de l’aide est générée automatiquement et vous indique uniquement la syntaxe des commandes. Nous ajouterons plus de documentation à mesure que nous approcherons de la finalisation de la conception des applets de commande.

Vous pouvez aussi vous référer à PowerShell ISE, un autre moyen agréable de découvrir la syntaxe que vous ne connaissez peut-être pas si vous n’avez pas encore souvent utilisé PowerShell. Si vous exécutez une référence qui l’autorise, essayez de démarrer ISE : ouvrez le volet Commandes, choisissez le module « Conteneurs », qui affiche une représentation graphique des applets de commande et leurs jeux de paramètres.

PS : juste pour prouver que c’est faisable, voici une fonction PowerShell qui adapte certaines des applets de commande que nous avons déjà vues en un ersatz de `docker run`. (En fait, c’est une preuve de concept qui n’est pas encore développée activement).

``` PowerShell
function Run-Container ([string]$ContainerImageName, [string]$Name="fancy_name", [switch]$Remove, [switch]$Interactive, [scriptblock]$Command) {
    $image = Get-ContainerImage -Name $ContainerImageName
    $container = New-Container -Name $Name -ContainerImage $image
    Start-Container $container

    if ($Interactive) {
         Start-Process powershell ("-NoExit", "-c", "Enter-PSSession -ContainerId $($container.Id)") -Wait
    } else {
        Invoke-Command -ContainerId $container.Id -ScriptBlock $Command
    }

    Stop-Container $container

    if ($Remove) {
        Remove-Container $container -Force
    }
} 
```

## Docker

Les conteneurs Windows peuvent être gérés avec des commandes Docker. Bien que les conteneurs Windows devraient pouvoir être comparés à leurs homologues Linux et avoir la même expérience de gestion avec Docker, il existe certaines commandes Docker qui n’ont tout simplement aucun sens pour un conteneur Windows. D’autres commandes n’ont tout simplement pas été testées (nous allons y parvenir).

Pour ne pas dupliquer la documentation sur les API disponible dans Docker, voici un lien vers les API de gestion correspondantes. Les procédures pas à pas sont formidables.

Nous consignons ce qui fonctionne et ce qui ne fonctionne pas dans les API Docker dans notre document Travail en cours.





<!--HONumber=Feb16_HO3-->



# <a name="build-and-run-an-application-with-or-without-net-core-20-or-powershell-core-6"></a>Créer et exécuter une application avec ou sans .NETCore2.0 ou PowerShellCore6

Dans cette version, l’image de base du système d’exploitation du conteneur NanoServer a supprimé .NETCore et PowerShell, bien que .NET Core et PowerShell soient pris en charge en tant que conteneur en couche additionnel sur le conteneur NanoServer de base.  

Si votre conteneur doit exécuter du code natif ou des infrastructures ouvertes telles que Node.js, Python, Ruby, etc., le conteneur NanoServer de base est suffisant.  Il convient toutefois de noter que certains codes natifs peuvent ne pas fonctionner en raison des [économies d’encombrement](https://docs.microsoft.com/en-us/windows-server/get-started/nano-in-semi-annual-channel) introduites dans cette version par rapport à WindowsServer2016. Si vous remarquez des problèmes de régression, faites-le nous savoir dans les [forums](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers). 

Pour créer votre conteneur à partir d’un fichier Dockerfile, utilisez la commande docker build, puis pour l’exécuter, utilisez la commande docker run.  La commande suivante télécharge l’image de base du système d’exploitation du conteneur NanoServer, ce qui peut prendre quelques minutes, et imprime un message «Hello World!» sur la console de l’hôte.

```
docker run microsoft/nanoserver-insider cmd /c echo Hello World!
```

Vous pouvez créer des applications plus complexes à l’aide de [fichiers Dockerfile sur Windows](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile), avec la syntaxe de fichiers Dockerfile comme FROM, RUN, COPY, ADD, CMD, etc. Vous ne serez pas en mesure d’exécuter certaines commandes immédiatement à partir de cette image de base, mais vous pourrez désormais créer des images de conteneur incluant uniquement les éléments dont vous avez besoin pour que votre application fonctionne.

Étant donné que .NETCore et PowerShell ne sont pas disponibles dans l’image de base du système d’exploitation du conteneur NanoServer, le défi consiste à créer un conteneur dont le contenu utilise un format zip compressé. Grâce à la fonctionnalité de [création échelonnée](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) disponible dans Docker17.05, vous pouvez exploiter PowerShell dans un autre conteneur pour décompresser le contenu et le copier dans le conteneur Nano. Vous pouvez utiliser cette approche pour créer un conteneur .NETCore et un conteneur PowerShell. 

Vous pouvez extraire l’image du conteneur PowerShell à l’aide de cette commande:

```
docker pull microsoft/nanoserver-insider-powershell
```

Vous pouvez extraire l’image du conteneur .NETCore à l’aide de cette commande:

```
docker pull microsoft/nanoserver-insider-dotnet
```

Voici quelques exemples d’utilisation de créations échelonnées pour la création de ces images de conteneur.

## <a name="deploy-apps-based-on-net-core-20"></a>Déployer des applications basées sur .NETCore2.0
Vous pouvez exploiter l’image de conteneur .NETCore2.0 de la version Insider pour exécuter vos applications .NETCore si votre application .NETCore est créée à un autre emplacement et que vous souhaitez l’exécuter dans le conteneur.  Des informations supplémentaires sur l’exécution d’une application .NETCore avec les images de conteneur .NETCore sont disponibles à partir de [.NET Core GitHub](https://github.com/dotnet/dotnet-docker-nightly).  Si vous développez une application dans le conteneur, utilisez le Kit de développement logiciel (SDK) .NETCore à la place.  Les utilisateurs avancés peuvent créer leur propre conteneur .NETCore2.0 avec la version .NETCore2.0, un fichier Dockerfile et l’URL spécifiée dans [dotnet-docker-nightly](https://github.com/dotnet/dotnet-docker-nightly/tree/master/2.0). Pour ce faire, ils peuvent utiliser un conteneur WindowsServerCore pour télécharger et décompresser les fichiers.  L’exemple de fichier Dockerfile est identique au [fichier Dockerfile .NETCoreRuntime](https://github.com/dotnet/dotnet-docker-nightly/blob/master/2.0/runtime/nanoserver-insider/amd64/Dockerfile).


Avec ce fichier Dockerfile, un conteneur .NETCore2.0 peut être créé à l’aide de la commande suivante.

```
docker build -t nanoserverdnc -f Dockerfile-dotnetRuntime .
```

## <a name="run-powershell-core-6-in-a-container"></a>Exécuter PowerShellCore 6dans un conteneur
À l’aide de la même méthode de [création échelonnée](https://docs.docker.com/engine/userguide/eng-image/multistage-build/), vous pouvez créer un conteneur PowerShellCore6 avec [ce fichier Dockerfile PowerShell](https://github.com/PowerShell/PowerShell-Docker/blob/master/release/stable/nanoserver/docker/Dockerfile).


Ensuite, exécutez la commande docker build pour créer l’image de conteneur PowerShell.

``` 
docker build -t nanoserverPowerShell6 -f Dockerfile-PowerShell6 .
```

Pour plus d’informations, voir [PowerShell GitHub](https://github.com/PowerShell/PowerShell-Docker/tree/master/release).  Il est important de mentionner que le fichier zip PowerShell contient un sous-ensemble de .NETCore2.0 qui est nécessaire pour créer PowerShellCore6.  Si vos modules PowerShell dépendent de .NETCore2.0, il est possible de créer le conteneur PowerShell sur le conteneur Nano .NETCore, au lieu du conteneur Nano de base, autrement dit à l’aide de FROM microsoft/nanoserver-insider-dotnet dans le fichier Dockerfile. 

## <a name="next-steps"></a>Étapes suivantes
- Utilisez une des nouvelles images de conteneur basées sur NanoServer, disponibles dans le hub Docker, c’est-à-dire une image NanoServer de base, Nano avec .NETCore2.0 et Nano avec PowerShellCore6
- Créez votre propre image de conteneur basée sur la nouvelle image de base du système d’exploitation du conteneur NanoServer en utilisant l’exemple de contenu de fichier Dockerfile fourni dans ce guide 

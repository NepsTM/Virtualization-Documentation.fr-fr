# Dossiers partagés de conteneur

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.**

Les dossiers partagés permettent aux données d’être partagées entre un hôte de conteneur et un conteneur. Une fois que le dossier partagé est créé, il est disponible dans le conteneur. Toutes les données qui se trouvent dans le dossier partagé de l’hôte sont disponibles dans le conteneur. Toutes les données qui se trouvent dans le dossier partagé du conteneur sont disponibles sur l’hôte. Un dossier unique sur l’hôte peut être partagé avec plusieurs conteneurs. Dans cette configuration, les données peuvent être partagées entre les conteneurs en cours d’exécution.

## Gérer les données - PowerShell

### Créer un dossier partagé

Pour créer un dossier partagé, utilisez la commande `Add-ContainerSharedFolder`. L’exemple ci-dessous crée un répertoire dans le conteneur `c:\shared_data`, qui est mappé à un répertoire sur l’hôte `c:\data_source`.

> Un conteneur doit être dans un état arrêté pour pouvoir ajouter un dossier partagé.

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\data_source -DestinationPath c:\shared_data

ContainerName SourcePath       DestinationPath AccessMode
------------- ----------       --------------- ----------
DEMO          c:\data_source   c:\shared_data  ReadWrite
```

### Dossier partagé en lecture seule

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\sf1 -DestinationPath c:\sf2 -AccessMode ReadOnly

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\sf1     c:\sf2          ReadOnly
```

### Liste des dossiers partagés

Pour afficher la liste des dossiers partagés d’un conteneur particulier, utilisez la commande `Get-ContainerSharedFolder`.

```powershell
PS C:\> Get-ContainerSharedFolder -ContainerName DEMO2

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\source  c:\source       ReadWrite
```

### Modifier un dossier partagé

Pour modifier la configuration d’un dossier partagé existant, utilisez la commande `Set-ContainerSharedFolder`.

```powershell
PS C:\> Set-ContainerSharedFolder -ContainerName SFRO -SourcePath c:\sf1 -DestinationPath c:\sf1
```

### Supprimer un dossier partagé

Pour supprimer un dossier partagé, utilisez la commande `Remove-ContainerSharedFolder`.

> Un conteneur doit être dans un état arrêté pour pouvoir supprimer un dossier partagé

```powershell
PS C:\> Remove-ContainerSharedFolder -ContainerName DEMO2 -SourcePath c:\source -DestinationPath c:\source
```
## Gérer les données - Docker

### Montage de volumes

Quand vous gérez des conteneurs Windows avec Docker, vous pouvez monter des volumes à l’aide de l’option `-v`.

Dans l’exemple ci-dessous, le dossier source est c:\source et le dossier destination c:\destination.

```powershell
PS C:\> docker run -it -v c:\source:c:\destination 1f62aaf73140 cmd
```

Pour plus d’informations sur la gestion des données dans des conteneurs avec Docker, voir [Volumes Docker sur Docker.com](https://docs.docker.com/userguide/dockervolumes/).

## Vidéo de la procédure pas à pas

<iframe src="https://channel9.msdn.com/blogs/Containers/Container-Fundamentals--part-3-Shared-Folders/Player#ccLang=fr" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>



<!--HONumber=Feb16_HO1-->

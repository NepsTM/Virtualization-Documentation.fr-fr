



# Gestion des ressources de conteneur

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.**

Les conteneurs Windows offrent la possibilité de gérer la quantité de ressources de processeur, d’E/S disque, de réseau et de mémoire que les conteneurs peuvent consommer. En limitant la consommation de ressources du conteneur, vous permettez que les ressources de l’hôte soient utilisées efficacement et vous évitez la surconsommation. Ce document décrit en détail la gestion des ressources de conteneur avec PowerShell et Docker.

## Gérer les ressources avec PowerShell

### Mémoire

Vous pouvez définir des limites de mémoire de conteneur au moment de la création du conteneur à l’aide du paramètre `-MaximumMemoryBytes` de la commande `New-Container`. Cet exemple définit la quantité maximale de mémoire avec la valeur 256 Mo.

```powershell
PS C:\> New-Container -Name TestContainer -MaximumMemoryBytes 256MB -ContainerimageName WindowsServerCore
```
Vous pouvez également définir la limite de mémoire d’un conteneur existant à l’aide de l’applet de commande `Set-ContainerMemory`.

```powershell
PS C:\> Set-ContainerMemory -ContainerName TestContainer -MaximumBytes 256mb
```

### Bande passante réseau

Vous pouvez définir des limites de bande passante réseau sur un conteneur existant. Pour ce faire, vérifiez que le conteneur possède une carte réseau à l’aide de la commande `Get-ContainerNetworkAdapter`. S’il n’existe pas de carte réseau, utilisez la commande `Add-ContainerNetworkAdapter` pour en créer une. Enfin, utilisez la commande `Set-ContainerNetworkAdapter` pour limiter la bande passante maximale de sortie du conteneur.

Dans l’exemple ci-dessous, la bande passante maximale est de 100 Mbits/s.

```powershell
PS C:\> Set-ContainerNetworkAdapter -ContainerName TestContainer -MaximumBandwidth 100000000
```

### Processeur

Pour limiter la quantité de calcul qu’un conteneur peut utiliser, définissez un pourcentage maximal de processeur ou une pondération relative pour le conteneur. Bien que vous ayez la possibilité d’utiliser ces deux schémas de gestion de processeur, cela n’est pas recommandé. Par défaut, tous les conteneurs peuvent utiliser entièrement le processeur (un maximum de 100 %) et ont une pondération relative de 100.

L’exemple ci-dessous définit la pondération relative du conteneur avec la valeur 1 000. La pondération par défaut d’un conteneur étant 100, ce conteneur a une priorité 10 fois supérieure à celle d’un conteneur ayant la valeur par défaut. La valeur maximale est 10 000.

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 -RelativeWeight 10000
```

Vous pouvez également définir une limite matérielle sur la quantité de processeur qu’un conteneur peut utiliser en termes de pourcentage de temps processeur. Par défaut, un conteneur peut utiliser 100 % du processeur. L’exemple ci-dessous définit le pourcentage maximal de processeur qu’un conteneur peut utiliser avec une valeur de 30 %. L’utilisation de l’indicateur -Maximum définit automatiquement la propriété RelativeWeight avec la valeur 100.

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 -Maximum 30
```

### E/S de stockage

Vous pouvez limiter la quantité d’E/S qu’un conteneur peut utiliser en termes de bande passante (octets par seconde) ou 8 k d’E/S par seconde normalisées. Ces deux paramètres peuvent être définis conjointement. La limitation se produit dès que la première limite est atteinte.

```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumBandwidth 1000000
```
```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumIOPS 32
```

## Gérer les ressources avec Docker

Nous offrons la possibilité de gérer un sous-ensemble des ressources de conteneur avec Docker. Plus précisément, nous permettons aux utilisateurs de spécifier la façon dont le processeur est partagé entre les conteneurs.

### Processeur

Le pourcentage de processeur réparti entre les conteneurs peut être géré au moment de l’exécution avec l’indicateur --cpu-shares. Par défaut, tous les conteneurs bénéficient d’une même proportion de temps processeur. Pour modifier le pourcentage relatif de processeur que les conteneurs utilisent, exécutez l’indicateur --cpu-shares avec une valeur comprise entre 1 et 10 000. Par défaut, tous les conteneurs reçoivent une pondération de 5 000. Pour plus d’informations sur la contrainte de partage d’UC, consultez les [informations de référence sur Docker Run](https://docs.docker.com/engine/reference/run/#cpu-share-constraint).

```powershell 
C:\> docker run -it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## Problèmes connus

- Les contrôles de ressources de processeur et d’E/S ne sont actuellement pas pris en charge par les conteneurs Hyper-V.
- Les contrôles de ressources d’E/S ne sont actuellement pas pris en charge par les dossiers partagés de conteneur.

## Vidéo de la procédure pas à pas

<iframe src="https://channel9.msdn.com/Blogs/containers/Container-Fundamentals--Part-4-Resource-Management/player#ccLang=fr" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>







<!--HONumber=Feb16_HO4-->



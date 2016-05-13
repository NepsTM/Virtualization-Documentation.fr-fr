



# Gestion des conteneurs Windows Server

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.**

Le cycle de vie de conteneur inclut des actions, telles que le démarrage, l’arrêt et la suppression de conteneurs. Quand vous effectuez ces actions, il est possible que vous deviez également récupérer une liste d’images de conteneur, gérer la mise en réseau des conteneurs et limiter les ressources des conteneurs. Ce document décrit en détail les tâches de gestion des conteneurs de base à l’aide de PowerShell.

Pour obtenir une documentation sur la gestion des conteneurs Windows avec Docker, consultez le document Docker [Utilisation de conteneurs](https://docs.docker.com/userguide/usingdocker/).

## PowerShell

### Créer un conteneur

Quand vous créez un conteneur, vous avez besoin du nom d’une image de conteneur qui servira de base de conteneur. Vous pouvez rechercher ce nom à l’aide de la commande `Get-ContainerImage`.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version         IsOSImage
----              ---------    -------         ---------
NanoServer        CN=Microsoft 10.0.10584.1000 True
WindowsServerCore CN=Microsoft 10.0.10584.1000 True
```

Utilisez la commande `New-Container` pour créer un conteneur. Le conteneur peut également recevoir un nom NetBIOS à l’aide du paramètre `-ContainerComputerName`.

```powershell
PS C:\> New-Container -ContainerImageName WindowsServerCore -Name demo -ContainerComputerName demo

Name State Uptime   ParentImageName
---- ----- ------   ---------------
demo  Off   00:00:00 WindowsServerCore
```

Une fois le conteneur créé, ajoutez-lui une carte réseau.

```powershell
PS C:\> Add-ContainerNetworkAdapter -ContainerName demo
```

Pour pouvoir connecter la carte réseau du conteneur à un commutateur virtuel, vous avez besoin du nom du commutateur. Pour obtenir la liste des commutateurs virtuels, utilisez la commande `Get-VMSwitch`.

```powershell
PS C:\> Get-VMSwitch

Name SwitchType NetAdapterInterfaceDescription
---- ---------- ------------------------------
DHCP External   Microsoft Hyper-V Network Adapter
NAT  NAT
```

Connectez la carte réseau au commutateur virtuel à l’aide de la commande `Connect-ContainerNetworkAdapter`. **REMARQUE** : Cette opération peut également être effectuée quand le conteneur est créé à l’aide du paramètre -SwitchName.

```powershell
PS C:\> Connect-ContainerNetworkAdapter -ContainerName demo -SwitchName NAT
```

### Démarrer un conteneur

Pour démarrer le conteneur, un objet PowerShell représentant ce conteneur est énuméré. Pour cela, placez la sortie de la commande `Get-Container` dans une variable PowerShell.

```powershell
PS C:\> $container = Get-Container -Name demo
```

Vous pouvez ensuite utiliser ces données avec la commande `Start-Container` pour démarrer le conteneur.

```powershell
PS C:\> Start-Container $container
```

Le script suivant démarre tous les conteneurs sur l’hôte.

```powershell
PS C:\> Get-Container | Start-Container
```

### Se connecter à un conteneur

PowerShell Direct peut être utilisé pour se connecter à un conteneur. Cela peut être utile si vous devez effectuer manuellement une tâche, telle que l’installation d’un logiciel, le démarrage d’un processus ou la résolution des problèmes d’un conteneur. PowerShell Direct étant utilisé, une session PowerShell peut être créée avec le conteneur, quelle que soit la configuration du réseau. Pour plus d’informations sur PowerShell Direct, consultez le [blog PowerShell Direct](http://blogs.technet.com/b/virtualization/archive/2015/05/14/powershell-direct-running-powershell-inside-a-virtual-machine-from-the-hyper-v-host.aspx).

Pour créer une session interactive avec le conteneur, utilisez la commande `Enter-PSSession`.

 ```powershell
PS C:\> Enter-PSSession -ContainerName demo -RunAsAdministrator
 ```

Notez qu’une fois la session PowerShell à distance créée, l’invite PowerShell change pour refléter le nom du conteneur.

```powershell
[demo]: PS C:\>
```

Les commandes peuvent également être exécutées sur un conteneur sans créer de session PowerShell persistante. Pour ce faire, utilisez la commande `Invoke-Command`.

L’exemple suivant crée un dossier nommé « Application » dans le conteneur.

```powershell

PS C:\> Invoke-Command -ContainerName demo -ScriptBlock {New-Item -ItemType Directory -Path c:\application }

Directory: C:\
Mode                LastWriteTime         Length Name                                                 PSComputerName
----                -------------         ------ ----                                                 --------------
d-----       10/28/2015   3:31 PM                application                                          demo
```

### Arrêter un conteneur

Pour arrêter le conteneur, un objet PowerShell représentant ce conteneur est nécessaire. Pour cela, placez la sortie de la commande `Get-Container` dans une variable PowerShell.

```powershell
PS C:\> $container = Get-Container -Name demo
```

Vous pouvez ensuite utiliser ces données avec la commande `Stop-Container` pour arrêter le conteneur.

```powershell
PS C:\> Stop-Container $container
```

Le script suivant arrête tous les conteneurs sur l’hôte.

```powershell
PS C:\> Get-Container | Stop-Container
```

### Supprimer un conteneur

Quand un conteneur est devenu inutile, il peut être supprimé. Pour supprimer un conteneur, il doit être arrêté, et un objet PowerShell doit être créé pour le représenter.

```powershell
PS C:\> $container = Get-Container -Name demo
```

Pour supprimer le conteneur, utilisez la commande `Remove-Container`.

```powershell
PS C:\> Remove-Container $container -Force
```

Le script suivant supprime tous les conteneurs sur l’hôte.

```powershell
PS C:\> Get-Container | Remove-Container -Force
```

## Docker

### Créer un conteneur

Pour créer un conteneur avec Docker, utilisez la commande `docker run`.

```powershell
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Pour plus d’informations sur la commande docker run, voir la [référence sur docker run}( https://docs.docker.com/engine/reference/run/).

### Arrêter un conteneur

Pour arrêter un conteneur avec Docker, utilisez la commande `docker stop`.

```powershell
PS C:\> docker stop tender_panini

tender_panini
```

Cet exemple arrête tous les conteneurs en cours d’exécution avec Docker.

```powershell
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### Supprimer un conteneur

Pour supprimer un conteneur avec Docker, utilisez la commande `docker rm`.

```powershell
PS C:\> docker rm prickly_pike

prickly_pike
```

Pour supprimer tous les conteneurs avec Docker

```powershell
PS C:\> docker rm $(docker ps -a -q)

dc3e282c064d
2230b0433370
```

Pour plus d’informations sur la commande docker rm, consultez les [informations de référence sur docker rm](https://docs.docker.com/engine/reference/commandline/rm/).






<!--HONumber=Feb16_HO4-->



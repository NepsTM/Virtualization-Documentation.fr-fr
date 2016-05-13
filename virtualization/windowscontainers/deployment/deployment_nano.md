---
author: neilpeterson
---


# Déploiement d’un hôte de conteneur – Nano Server

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.**

Les étapes du déploiement d’un hôte de conteneur Windows sont différentes selon le système d’exploitation et le type de système hôte (physique ou virtuel). Les étapes décrites dans ce document sont utilisées pour déployer un hôte de conteneurs Windows sur Nano Server, sur un système physique ou virtuel. Pour installer un hôte de conteneurs Windows sur Windows Server, consultez [Déploiement d’un hôte de conteneurs – Windows Server](./deployment.md).

Pour plus d’informations sur la configuration requise, consultez [Configuration requise pour l’hôte de conteneurs Windows](./system_requirements.md).

Les scripts PowerShell sont disponibles pour automatiser le déploiement d’un hôte de conteneur Windows.
- [Déployer un hôte de conteneurs dans une nouvelle machine virtuelle Hyper-V](../quick_start/container_setup.md).
- [Déployer un hôte de conteneurs sur un système existant](../quick_start/inplace_setup.md).


# Hôte Nano Server

Les étapes répertoriées dans ce tableau peuvent servir à déployer un hôte de conteneur vers Nano Server. Voici les configurations requises pour les conteneurs Windows Server et Hyper-V.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>Action de déploiement</strong></td>
<td width="70%"><strong>Détails</strong></td>
</tr>
<tr>
<td>[Préparer Nano Server pour les conteneurs](#nano)</td>
<td>Préparez un disque dur virtuel Nano Server avec les fonctionnalités de conteneur et Hyper-V.</td>
</tr>
<tr>
<td>[Créer un commutateur virtuel](#vswitch)</td>
<td>Les conteneurs se connectent à un commutateur virtuel pour assurer la connectivité réseau.</td>
</tr>
<tr>
<td>[Configurer NAT](#nat)</td>
<td>Si un commutateur virtuel est configuré avec la traduction d’adresses réseau, NAT doit aussi être configuré.</td>
</tr>
<tr>
<td>[Installer des images de système d’exploitation du conteneur](#img)</td>
<td>Les images de système d’exploitation fournissent la base pour les déploiements de conteneurs.</td>
</tr>
<tr>
<td>[Installer Docker](#docker)</td>
<td>Facultatif, mais nécessaire pour créer et gérer des conteneurs Windows avec Docker. </td>
</tr>
</table>

Ces étapes doivent être effectuées si des conteneurs Hyper-V seront utilisés. Remarque : Les opérations marquées d’un astérisque (*) sont nécessaires seulement si l’hôte de conteneurs est lui-même une machine virtuelle Hyper-V.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>Action de déploiement</strong></td>
<td width="70%"><strong>Détails</strong></td>
</tr>
<tr>
<td>[Activer le rôle Hyper-V](#hypv) </td>
<td>Hyper-V est requis uniquement si les conteneurs Hyper-V sont utilisés.</td>
</tr>
<tr>
<td>[Activer la virtualisation imbriquée *](#nest)</td>
<td>Si l’hôte de conteneur est lui-même une machine virtuelle Hyper-V, la virtualisation imbriquée doit être activée.</td>
</tr>
<tr>
<td>[Configurer des processeurs virtuels *](#proc)</td>
<td>Si l’hôte de conteneur est lui-même une machine virtuelle Hyper-V, au moins deux processeurs virtuels doivent être configurés.</td>
</tr>
<tr>
<td>[Désactiver la mémoire dynamique *](#dyn)</td>
<td>Si l’hôte de conteneur est lui-même une machine virtuelle Hyper-V, la mémoire dynamique doit être désactivée.</td>
</tr>
<tr>
<td>[Configurer l’usurpation des adresses MAC *](#mac)</td>
<td>Si l’hôte de conteneur est virtualisé, l’usurpation MAC doit être activée.</td>
</tr>
</table>

## Étapes du déploiement

### <a name=nano></a> Préparer Nano Server

Le déploiement de Nano Server implique la création d’un disque dur virtuel préparé, qui inclut le système d’exploitation Nano Server et d’autres packages de fonctionnalités. Ce guide détaille rapidement la préparation d’un disque dur virtuel Nano Server, qui peut être utilisé pour les conteneurs Windows. Pour plus d’informations sur Nano Server et pour explorer différentes options de déploiement de Nano Server, consultez la [documentation de Nano Server](https://technet.microsoft.com/en-us/library/mt126167.aspx).

Créez un dossier nommé `nano`.

```powershell
PS C:\> New-Item -ItemType Directory c:\nano
```

Recherchez les fichiers `NanoServerImageGenerator.psm1` et `Convert-WindowsImage.ps1` dans le dossier Nano Server, sur le support Windows Server. Copiez-les dans `c:\nano`.

```powershell
#Set path to Windows Server 2016 Media
PS C:\> $WindowsMedia = "C:\Users\Administrator\Desktop\TP4 Release Media"

PS C:\> Copy-Item $WindowsMedia\NanoServer\Convert-WindowsImage.ps1 c:\nano

PS C:\> Copy-Item $WindowsMedia\NanoServer\NanoServerImageGenerator.psm1 c:\nano
```
Exécutez la commande suivante pour créer un disque dur virtuel Nano Server. Le paramètre `-Containers` indique que le package du conteneur est installé, tandis que le paramètre `-Compute` traite le package Hyper-V. Hyper-V est obligatoire seulement si des conteneurs Hyper-V sont utilisés.

```powershell
PS C:\> Import-Module C:\nano\NanoServerImageGenerator.psm1

PS C:\> New-NanoServerImage -MediaPath $WindowsMedia -BasePath c:\nano -TargetPath C:\nano\NanoContainer.vhdx -MaxSize 10GB -GuestDrivers -ReverseForwarders -Compute -Containers
```
Quand vous avez terminé, créez une machine virtuelle à partir du fichier `NanoContainer.vhdx`. Cette machine virtuelle exécute le système d’exploitation Nano Server et des packages facultatifs.

### <a name=vswitch></a>Créer un commutateur virtuel

Chaque conteneur doit être relié à un commutateur virtuel afin de communiquer via un réseau. Un commutateur virtuel est créé avec la commande `New-VMSwitch`. Les conteneurs prennent en charge un commutateur virtuel de type `Externe` ou `NAT`. Pour plus d’informations sur la mise en réseau des conteneurs Windows, consultez [Mise en réseau d’un conteneur](../management/container_networking.md).

Cet exemple crée un commutateur virtuel nommé « Commutateur virtuel », un type NAT et le sous-réseau NAT 172.16.0.0/12.

```powershell
PS C:\> New-VMSwitch -Name "Virtual Switch" -SwitchType NAT -NATSubnetAddress "172.16.0.0/12"
```

### <a name=nat></a>Configurer NAT

Outre la création d’un commutateur virtuel, si le type de commutateur est NAT, un objet NAT doit être créé. Pour ce faire, utilisez la commande `New-NetNat`. Cet exemple crée un objet NAT, nommé `ContainerNat`, et un préfixe d’adresse qui correspond au sous-réseau NAT assigné au commutateur du conteneur.

```powershell
PS C:\> New-NetNat -Name ContainerNat -InternalIPInterfaceAddressPrefix "172.16.0.0/12"

Name                             : ContainerNat
ExternalIPInterfaceAddressPrefix :
InternalIPInterfaceAddressPrefix : 172.16.0.0/12
IcmpQueryTimeout                 : 30
TcpEstablishedConnectionTimeout  : 1800
TcpTransientConnectionTimeout    : 120
TcpFilteringBehavior             : AddressDependentFiltering
UdpFilteringBehavior             : AddressDependentFiltering
UdpIdleSessionTimeout            : 120
UdpInboundRefresh                : False
Store                            : Local
Active                           : True
```

### <a name=img></a>Installer les images de système d’exploitation

Une image de système d’exploitation est utilisée comme base pour un conteneur Windows Server ou Hyper-V. L’image est utilisée pour déployer un conteneur, et peut ensuite être modifiée et capturée dans une nouvelle image de conteneur. Les images de système d’exploitation ont été créées avec Windows Server Core et Nano Server comme système d’exploitation sous-jacent.

Les images de système d’exploitation du conteneur sont accessibles et installées à l’aide du module PowerShell ContainerProvider. Vous devez installer ce module avant de l’utiliser. Les commandes suivantes peuvent être utilisées pour installer le module.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

Utilisez `Find-ContainerImage` pour retourner une liste d’images depuis le gestionnaire de package PowerShell OneGet.

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```
**Remarque** : Pour l’instant, seule l’image du système d’exploitation Nano Server est compatible avec un hôte de conteneurs Nano Server. Pour télécharger et installer l’image du système d’exploitation de base Nano Server, exécutez la commande suivante.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

Vérifiez que l’image est installée en utilisant la commande `Get-ContainerImage`.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
```
Pour plus d’informations sur la gestion des images de conteneur, consultez [Images de conteneur Windows](../management/manage_images.md).


### <a name=docker></a>Installer Docker

Le démon Docker et l’interface de ligne de commande ne sont pas fournis avec Windows, et ne sont pas installés avec la fonctionnalité de conteneur Windows. Docker n’est pas obligatoire pour utiliser des conteneurs Windows. Si vous voulez installer Docker, suivez les instructions de l’article [Docker et Windows](./docker_windows.md).

Vous pouvez utiliser la commande `Enter-PSSession` dans l’hôte de gestion Hyper-V pour vous connecter à l’hôte du conteneur.

```powershell
PS C:\> Enter-PSSession -VMName <VM Name>
```

## Hôte de conteneurs Hyper-V

### <a name=hypv></a>Activer le rôle Hyper-V

Sur Nano Server, cette opération peut être effectuée lors de la création de l’image de Nano Server. Pour ces instructions, consultez [Préparation de Nano Server pour les conteneurs](#nano).

### <a name=nest></a>Virtualisation imbriquée

Si l’hôte de conteneur est exécuté sur une machine virtuelle Hyper-V et héberge également des conteneurs Hyper-V, la virtualisation imbriquée doit être activée. Vous pouvez pour cela exécuter la commande PowerShell suivante.

**Remarque** : Les machines virtuelles doivent être arrêtées lors de l’exécution de cette commande.

```powershell
PS C:\> Set-VMProcessor -VMName <VM Name> -ExposeVirtualizationExtensions $true
```

### <a name=proc></a>Configurer les processeurs virtuels

Si l’hôte de conteneur est exécuté sur une machine virtuelle Hyper-V et héberge également des conteneurs Hyper-V, la machine virtuelle requiert au moins deux processeurs. Cette configuration peut être effectuée via les paramètres de la machine virtuelle ou avec la commande suivante.

**Remarque** : Les machines virtuelles doivent être arrêtées lors de l’exécution de cette commande.

```poweshell
PS C:\> Set-VMProcessor -VMName <VM Name> -Count 2
```

### <a name=dyn></a>Désactiver la mémoire dynamique

Si l’hôte de conteneur est lui-même une machine virtuelle Hyper-V, la mémoire dynamique doit être désactivée sur la machine virtuelle de l’hôte de conteneur. Cette configuration peut être effectuée via les paramètres de la machine virtuelle ou avec la commande suivante.

**Remarque** : Les machines virtuelles doivent être arrêtées lors de l’exécution de cette commande.

```poweshell
PS C:\> Set-VMMemory <VM Name> -DynamicMemoryEnabled $false
```

### <a name=mac></a>Usurpation des adresses MAC

Enfin, si l’hôte du conteneur s’exécute à l’intérieur d’une machine virtuelle Hyper-V, l’usurpation MAC doit être activée. Chaque conteneur reçoit ainsi une adresse IP. Pour activer l’usurpation des adresses MAC, exécutez la commande suivante sur l’hôte Hyper-V. La propriété VMName sera le nom de l’hôte de conteneur.

```powershell
PS C:\> Get-VMNetworkAdapter -VMName <VM Name> | Set-VMNetworkAdapter -MacAddressSpoofing On
```






<!--HONumber=Mar16_HO3-->



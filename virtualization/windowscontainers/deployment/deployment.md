# Déploiement d’un hôte de conteneur – Windows Server

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.**

Les étapes du déploiement d’un hôte de conteneur Windows sont différentes selon le système d’exploitation et le type de système hôte (physique ou virtuel). Les étapes décrites dans ce document permettent de déployer un hôte de conteneur Windows pour Windows Server 2016 ou Windows Server Core 2016 sur un système physique ou virtuel. Pour installer un hôte de conteneur Windows sur Nano Server, voir [Déploiement d’un hôte de conteneur – Nano Server](./deployment_nano.md).

Pour plus d’informations sur la configuration requise, voir [Configuration requise pour l’hôte de conteneur Windows](./system_requirements.md).

Les scripts PowerShell permettent également d’automatiser le déploiement d’un hôte de conteneur Windows.
- [Déployer un hôte de conteneur dans une nouvelle machine virtuelle Hyper-V](../quick_start/container_setup.md).
- [Déployer un hôte de conteneur sur un système existant](../quick_start/inplace_setup.md).

# Hôte Windows Server

Les étapes répertoriées dans ce tableau peuvent servir à déployer un hôte de conteneur vers Windows Server 2016 et Windows Server 2016 Core. Voici les configurations requises pour les conteneurs Windows Server et Hyper-V.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>Action de déploiement</strong></td>
<td width="70%"><strong>Détails</strong></td>
</tr>
<tr>
<td>[Installer la fonctionnalité de conteneur](#role)</td>
<td>La fonctionnalité de conteneur permet l’utilisation du conteneur Windows Server et Hyper-V.</td>
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
<td>[Installer les images de système d’exploitation du conteneur](#img)</td>
<td>Les images de système d’exploitation fournissent la base pour les déploiements de conteneurs.</td>
</tr>
<tr>
<td>[Installer Docker](#docker)</td>
<td>Facultatif, mais nécessaire pour créer et gérer des conteneurs Windows avec Docker.</td>
</tr>
</table>

Si vous utilisez des conteneurs Hyper-V, vous devrez effectuer ces étapes. Remarque : les opérations marquées d’un astérisque (*) ne sont nécessaires que si l’hôte du conteneur est une machine virtuelle Hyper-V.

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
<td>[Activer la virtualisation imbriquée*](#nest)</td>
<td>Si l’hôte de conteneur est lui-même une machine virtuelle Hyper-V, la virtualisation imbriquée doit être activée.</td>
</tr>
<tr>
<td>[Configurer les processeurs virtuels*](#proc)</td>
<td>Si l’hôte de conteneur est lui-même une machine virtuelle Hyper-V, au moins deux processeurs virtuels doivent être configurés.</td>
</tr>
<tr>
<td>[Désactiver la mémoire dynamique*](#dyn)</td>
<td>Si l’hôte de conteneur est lui-même une machine virtuelle Hyper-V, la mémoire dynamique doit être désactivée.</td>
</tr>
<tr>
<td>[Configurer l’usurpation des adresses MAC*](#mac)</td>
<td>Si l’hôte de conteneur est virtualisé, l’usurpation MAC doit être activée.</td>
</tr>
</table>

## Étapes de déploiement

### <a name=role></a>Installer la fonctionnalité de conteneur

La fonctionnalité de conteneur peut être installée sur Windows Server 2016, ou Windows Server 2016 Core, à l’aide du Gestionnaire de serveur Windows ou de PowerShell.

Pour installer le rôle à l’aide de PowerShell, exécutez la commande suivante dans une session PowerShell avec élévation de privilèges.

```powershell
PS C:\> Install-WindowsFeature containers
```
Le système doit être redémarré après l’installation du rôle de conteneur.

```powershell
PS C:\> shutdown /r 
```
Une fois que le système a redémarré, utilisez la commande `Get-ContainerHost` pour vérifier que le rôle de conteneur est correctement installé :

```powershell
PS C:\> Get-ContainerHost

Name            ContainerImageRepositoryLocation
----            --------------------------------
WIN-LJGU7HD7TEP C:\ProgramData\Microsoft\Windows\Hyper-V\Container Image Store
```

### <a name=vswitch></a>Créer un commutateur virtuel

Chaque conteneur doit être relié à un commutateur virtuel afin de communiquer via un réseau. Un commutateur virtuel est créé avec la commande `New-VMSwitch`. Les conteneurs prennent en charge un commutateur virtuel de type `Externe` ou `NAT`. Pour plus d’informations sur la mise en réseau de conteneurs Windows, voir [Mise en réseau de conteneurs](../management/container_networking.md).

Cet exemple crée un commutateur virtuel nommé « Commutateur virtuel », un type NAT et le sous-réseau NAT 172.16.0.0/12.

```powershell
PS C:\> New-VMSwitch -Name "Virtual Switch" -SwitchType NAT -NATSubnetAddress 172.16.0.0/12
```

### <a name=nat></a>Configurer NAT

Outre la création d’un commutateur virtuel, si le type de commutateur est NAT, un objet NAT doit être créé. Pour ce faire, utilisez la commande `New-NetNat`. Cet exemple crée un objet NAT, nommé `ContainerNat`, et un préfixe d’adresse qui correspond au sous-réseau NAT assigné au commutateur du conteneur.

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

Utilisez `Find-ContainerImage` pour retourner une liste d’images à partir du gestionnaire de package PowerShell OneGet :
```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```
Pour télécharger et installer l’image du système d’exploitation de base Nano Server, exécutez la commande suivante.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

De même, cette commande télécharge et installe l’image de système d’exploitation de base Windows Server Core.

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0

Downloaded in 0 hours, 2 minutes, 28 seconds.
```

**Problème :** les applets de commande Save-ContainerImage et Install-ContainerImage ne fonctionnent pas avec une image de conteneur WindowsServerCore à partir d’une session PowerShell à distance.<br /> **Solution de contournement :** ouvrez une session sur l’ordinateur à l’aide du Bureau à distance et utilisez directement l’applet de commande Save-ContainerImage.

Utilisez la commande `Get-ContainerImage` pour vérifier que les images ont été installées.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```
Pour plus d’informations sur la gestion des images de conteneur, voir [Images de conteneur Windows](../management/manage_images.md).


### <a name=docker></a>Installer Docker

Le démon Docker et l’interface de ligne de commande ne sont pas fournis avec Windows, et ne sont pas installés avec la fonctionnalité de conteneur Windows. Docker n’est pas obligatoire pour utiliser des conteneurs Windows. Si vous voulez installer Docker, suivez les instructions de l’article [Docker et Windows](./docker_windows.md).


## Hôte de conteneur Hyper-V

### <a name=hypv></a>Activer le rôle Hyper-V

Si des conteneurs de Hyper-V doivent être déployés, le rôle Hyper-V doit être activé sur l’hôte du conteneur. Le rôle Hyper-V peut être installé sur Windows Server 2016 ou Windows Server 2016 Core à l’aide de la commande `Install-WindowsFeature`. Si l’hôte de conteneur est une machine virtuelle Hyper-V, la virtualisation imbriquée doit être activée en premier. Pour cela, voir [Configurer la virtualisation imbriquée](#nest).

```powershell
PS C:\> Install-WindowsFeature hyper-v
```

### <a name=nest></a>Configurer la virtualisation imbriquée

Si l’hôte de conteneur est exécuté sur une machine virtuelle Hyper-V et héberge également des conteneurs Hyper-V, la virtualisation imbriquée doit être activée. Vous pouvez pour cela exécuter la commande PowerShell suivante.

**Remarque** Les machines virtuelles doivent être désactivées lors de l’exécution de cette commande.

```powershell
PS C:\> Set-VMProcessor -VMName <VM Name> -ExposeVirtualizationExtensions $true
```

### <a name=proc></a>Configurer les processeurs virtuels

Si l’hôte de conteneur est exécuté sur une machine virtuelle Hyper-V et héberge également des conteneurs Hyper-V, la machine virtuelle requiert au moins deux processeurs. Cette configuration peut être effectuée via les paramètres de la machine virtuelle ou avec la commande suivante.

**Remarque** Les machines virtuelles doivent être désactivées lors de l’exécution de cette commande.

```poweshell
PS C:\> Set-VMProcessor –VMName <VM Name> -Count 2
```

### <a name=dyn></a>Désactiver la mémoire dynamique

Si l’hôte de conteneur est lui-même une machine virtuelle Hyper-V, la mémoire dynamique doit être désactivée sur la machine virtuelle de l’hôte de conteneur. Cette configuration peut être effectuée via les paramètres de la machine virtuelle ou avec la commande suivante.

**Remarque** Les machines virtuelles doivent être désactivées lors de l’exécution de cette commande.

```poweshell
PS C:\> Set-VMMemory <VM Name> -DynamicMemoryEnabled $false
```

### <a name=mac></a>Configurer l’usurpation des adresses MAC

Enfin, si l’hôte du conteneur s’exécute sur une machine virtuelle Hyper-V, l’usurpation MAC doit être activée. Chaque conteneur reçoit ainsi une adresse IP. Pour activer l’usurpation des adresses MAC, exécutez la commande suivante sur l’hôte Hyper-V. La propriété VMName sera le nom de l’hôte de conteneur.

```powershell
PS C:\> Get-VMNetworkAdapter -VMName <VM Name> | Set-VMNetworkAdapter -MacAddressSpoofing On
```





<!--HONumber=Feb16_HO1-->

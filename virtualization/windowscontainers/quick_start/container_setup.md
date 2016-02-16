# Déployer un hôte de conteneur Windows dans une nouvelle machine virtuelle Hyper-V

Ce document décrit l’utilisation d’un script PowerShell pour déployer une nouvelle machine virtuelle Hyper-V, qui est ensuite configurée comme un hôte de conteneur Windows.

Pour effectuer un pas à pas détaillé dans un déploiement avec script d’un hôte de conteneur Windows sur un système virtuel ou physique existant, voir [Déploiement d’un hôte de conteneur Windows sur place](./inplace_setup.md).

**À LIRE AVANT L’INSTALLATION DE L’IMAGE DE SYSTÈME D’EXPLOITATION DU CONTENEUR :** les termes du contrat de licence de la version préliminaire du logiciel Microsoft Windows Server (« termes du contrat de licence ») s’appliquent à votre utilisation du supplément de l’image de système d’exploitation du conteneur Microsoft Windows (« logiciel supplémentaire »). En téléchargeant et en utilisant le logiciel supplémentaire, vous acceptez les termes du contrat de licence, et vous ne pouvez pas l’utiliser si vous n’avez pas accepté les termes du contrat de licence. La version préliminaire du logiciel Windows Server et le logiciel supplémentaire sont tous deux concédés sous licence par Microsoft Corporation.

Les éléments suivants sont requis pour effectuer les exercices relatifs aux conteneurs **Windows Server** et **Hyper-V** dans ce démarrage rapide.

* Système exécutant Windows 10 build 10586 ou ultérieure / Windows Server Technical Preview 4 ou ultérieure.
* Rôle Hyper-V activé ([voir les instructions](https://msdn.microsoft.com/virtualization/hyperv_on_windows/quick_start/walkthrough_install#UsingPowerShell)).
* 20 Go de stockage disponible pour l’image hôte de conteneur, l’image de base du système d’exploitation et les scripts d’installation.
* Autorisations d’administrateur sur l’hôte Hyper-V.

> Un hôte de conteneur virtualisé exécutant des conteneurs Hyper-V nécessite une virtualisation imbriquée. L’hôte physique et l’hôte virtuel doivent tous deux exécuter un système d’exploitation qui prend en charge la virtualisation imbriquée. Pour plus d’informations, voir Nouveautés d’Hyper-V dans [Windows Server 2016 Technical Preview](https://technet.microsoft.com/library/dn765471.aspx#BKMK_nested).

## Configurer un nouvel hôte de conteneur dans une nouvelle machine virtuelle

Les conteneurs Windows sont constitués de plusieurs composants, tels que l’hôte de conteneur Windows et les images de base du système d’exploitation du conteneur. Nous avons mis au point un script qui effectue le téléchargement et la configuration de ces éléments pour vous. Suivez ces étapes pour déployer une nouvelle machine virtuelle Hyper-V et configurer ce système comme hôte de conteneur Windows.

Démarrez une session PowerShell en tant qu’administrateur. Pour cela, cliquez avec le bouton droit sur l’icône PowerShell et sélectionnez Exécuter en tant qu’administrateur, ou exécutez la commande suivante à partir de n’importe quelle session PowerShell.

``` powershell
PS C:\> start-process powershell -Verb runAs
```

Avant de télécharger et d’exécuter le script, vérifiez qu’un commutateur virtuel Hyper-V externe a été créé. Dans le cas contraire, ce script échoue.

Exécutez la commande suivante pour retourner une liste de commutateurs virtuels externes. Si rien n’est retourné, créez un commutateur virtuel externe, puis passez à l’étape suivante de ce guide.

```powershell
PS C:\> Get-VMSwitch | where {$_.SwitchType –eq “External”}
```

Utilisez la commande suivante pour télécharger le script de configuration. Le script peut également être téléchargé manuellement à partir de cet emplacement : [Script de configuration](https://aka.ms/tp4/New-ContainerHost).

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/New-ContainerHost -OutFile c:\New-ContainerHost.ps1
```

Exécutez la commande suivante pour créer et configurer l’hôte de conteneur, où `<hôte_conteneur>` est le nom de la machine virtuelle.

``` powershell
PS C:\> powershell.exe -NoProfile c:\New-ContainerHost.ps1 –VmName testcont -WindowsImage ServerDatacenterCore -Hyperv
```

Quand le script commence, vous êtes invité à entrer un mot de passe. Il s’agit du mot de passe affecté au compte d’administrateur.

Ensuite, vous êtes invité à lire et accepter les termes du contrat de licence.

```
Before installing and using the Windows Server Technical Preview 4 with Containers virtual machine you must:
    1. Review the license terms by navigating to this link: http://aka.ms/tp4/containerseula
    2. Print and retain a copy of the license terms for your records.
By downloading and using the Windows Server Technical Preview 4 with Containers virtual machine you agree to such
license terms. Please confirm you have accepted and agree to the license terms.
[N] No  [Y] Yes  [?] Help (default is "N"):
```

Le script commence alors à télécharger et configurer les composants de conteneur Windows. Ce processus peut prendre un certain temps en raison du téléchargement volumineux. Quand l’opération est terminée, la machine virtuelle est configurée et prête pour que vous puissiez créer et gérer des conteneurs Windows et des images de conteneur Windows avec à la fois PowerShell et Docker.

Quand le script de configuration est terminé, connectez-vous à la machine virtuelle via le mot de passe spécifié lors du processus de configuration et vérifiez que la machine virtuelle a une adresse IP valide. Une fois le traitement de ces éléments terminé, votre système doit être prêt pour les conteneurs Windows.

## Étapes suivantes : commencer à utiliser des conteneurs

Maintenant que vous disposez d’un système Windows Server 2016 exécutant la fonctionnalité de conteneur Windows, accédez aux guides suivants pour commencer à utiliser les conteneurs Windows Server et Hyper-V.

[Démarrage rapide : conteneurs Windows et PowerShell](./manage_powershell.md)  
[Démarrage rapide : conteneurs Windows et Docker](./manage_docker.md)




<!--HONumber=Feb16_HO2-->

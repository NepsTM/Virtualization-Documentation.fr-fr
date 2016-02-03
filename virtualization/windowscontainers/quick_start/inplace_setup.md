# Déployer un hôte de conteneur Windows sur un système virtuel ou physique existant

Ce document décrit l’utilisation d’un script PowerShell pour déployer et configurer le rôle de conteneur Windows sur un système virtuel ou physique existant.

Pour effectuer un pas à pas détaillé dans un déploiement avec script d’une nouvelle machine virtuelle Hyper-V configurée comme un hôte de conteneur Windows, voir [Nouvel hôte de conteneur Windows Hyper-V](./container_setup.md).

**À LIRE AVANT L’INSTALLATION DE L’IMAGE DE SYSTÈME D’EXPLOITATION DU CONTENEUR :** les termes du contrat de licence de la version préliminaire du logiciel Microsoft Windows Server (« termes du contrat de licence ») s’appliquent à votre utilisation du supplément de l’image de système d’exploitation du conteneur Microsoft Windows (« logiciel supplémentaire »). En téléchargeant et en utilisant le logiciel supplémentaire, vous acceptez les termes du contrat de licence, et vous ne pouvez pas l’utiliser si vous n’avez pas accepté les termes du contrat de licence. La version préliminaire du logiciel Windows Server et le logiciel supplémentaire sont tous deux concédés sous licence par Microsoft Corporation.

Les éléments suivants sont requis pour effectuer les exercices relatifs aux conteneurs Windows Server et conteneurs Hyper-V dans ce démarrage rapide.

* Système exécutant Windows Server Technical Preview 4 ou ultérieure.
* 10 Go de stockage disponible pour l’image hôte de conteneur, l’image de base du système d’exploitation et les scripts d’installation.
* Autorisations d’administrateur sur le système.

## Configurer un hôte de système nu ou de machine virtuelle existant pour les conteneurs

Les conteneurs Windows nécessitent les images de base du système d’exploitation du conteneur. Nous avons mis au point un script qui effectue le téléchargement et l’installation pour vous. Suivez ces étapes pour configurer votre système comme hôte de conteneur Windows. Pour plus d’informations, voir Nouveautés d’Hyper-V dans [Windows Server 2016 Technical Preview](https://tnstage.redmond.corp.microsoft.com/en-US/library/dn765471.aspx#BKMK_nested).

Démarrez une session PowerShell en tant qu’administrateur. Pour cela, exécutez la commande suivante à partir de la ligne de commande.

``` powershell
PS C:\> powershell.exe
```

Vérifiez que le titre de la fenêtre est « Administrateur : Windows PowerShell ». Si Administrateur n’est pas indiqué, lancez cette commande pour une exécution avec des privilèges d’administrateur :

``` powershell
PS C:\> start-process powershell -Verb runas
```

Utilisez la commande suivante pour télécharger le script d’installation. Le script peut également être téléchargé manuellement à partir de cet emplacement : [Script de configuration](https://aka.ms/tp4/Install-ContainerHost).

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/Install-ContainerHost -OutFile C:\Install-ContainerHost.ps1
```

Une fois le téléchargement terminé, exécutez le script.
``` PowerShell
PS C:\> C:\Install-ContainerHost.ps1 -HyperV
```

Le script commence alors à télécharger et configurer les composants de conteneur Windows. Ce processus peut prendre un certain temps en raison du téléchargement volumineux. L’ordinateur peut redémarrer au cours du processus. Quand l’opération est terminée, l’ordinateur est configuré et prêt pour que vous puissiez créer et gérer des conteneurs Windows et des images de conteneur Windows avec à la fois PowerShell et Docker.

Une fois le traitement de ces éléments terminé, votre système doit être prêt pour les conteneurs Windows.

## Étapes suivantes : commencer à utiliser des conteneurs

Maintenant que vous disposez d’un système Windows Server 2016 exécutant la fonctionnalité de conteneur Windows, accédez aux guides suivants pour commencer à utiliser les conteneurs Windows Server et Hyper-V.

[Démarrage rapide : conteneurs Windows et Docker](./manage_docker.md)

[Démarrage rapide : conteneurs Windows et PowerShell](./manage_powershell.md)




<!--HONumber=Jan16_HO1-->

---
title: Créer gMSAs pour les conteneurs Windows
description: Comment créer un compte de service géré de groupe (gMSAs) pour les conteneurs Windows.
keywords: dockeur, conteneurs, Active Directory, GMSA, compte de service géré de groupe, comptes de service géré par groupe
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 9ed9029e534d56bfe1830281d0bfd3ddde0cee9e
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079663"
---
# <a name="create-gmsas-for-windows-containers"></a>Créer gMSAs pour les conteneurs Windows

Les réseaux Windows utilisent fréquemment Active Directory (AD) pour faciliter l’authentification et l’autorisation entre les utilisateurs, les ordinateurs et les autres ressources du réseau. Les développeurs d’applications d’entreprise créent souvent leurs applications pour une intégration publicitaire et s’exécutent sur des serveurs appartenant à un domaine pour tirer parti de l’authentification intégrée de Windows, ce qui permet aux utilisateurs et autres services de se connecter automatiquement et de manière transparente à application avec leur identité.

Bien que les conteneurs Windows ne puissent pas être joints au domaine, ils peuvent toujours utiliser les identités de domaine Active Directory pour prendre en charge différents scénarios d’authentification.

Pour ce faire, vous pouvez configurer un conteneur Windows pour qu’il s’exécute avec un [compte de service géré de groupe](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA), qui est un type spécial de compte de service introduit dans Windows Server 2012 conçu pour permettre à plusieurs ordinateurs de partager une identité sans avoir besoin de pour connaître son mot de passe.

Lorsque vous exécutez un conteneur avec un gMSA, l’hôte de conteneur récupère le mot de passe gMSA à partir d’un contrôleur de domaine Active Directory et le donne à l’instance de conteneur. Le conteneur utilisera les informations d’identification gMSA chaque fois que son compte d’ordinateur doit accéder aux ressources du réseau.

Cet article explique comment commencer à utiliser les comptes de service géré du groupe Active Directory avec des conteneurs Windows.

## <a name="prerequisites"></a>Prérequis

Pour exécuter un conteneur Windows avec un compte de service géré de groupe, vous avez besoin des éléments suivants:

- Domaine Active Directory avec au moins un contrôleur de domaine exécutant Windows Server 2012 ou une version ultérieure. Il n’existe aucune exigence de niveau fonctionnel de forêt ou de domaine pour utiliser gMSAs, mais les mots de passe gMSA peuvent être distribués uniquement par les contrôleurs de domaine exécutant Windows Server 2012 ou une version ultérieure. Pour plus d’informations, voir [Configuration requise pour Active Directory pour gMSAs](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req).
- Autorisation de création d’un compte gMSA. Pour créer un compte gMSA, vous devez être un administrateur de domaine ou utiliser un compte qui a été délégué l’autorisation *création d’objets MSDS-GroupManagedServiceAccount* .
- Accès à Internet pour télécharger le module CredentialSpec PowerShell. Si vous travaillez dans un environnement déconnecté, vous pouvez [enregistrer le module](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1) sur un ordinateur équipé d’un accès à Internet et le copier sur l’hôte de votre ordinateur de développement ou conteneur.

## <a name="one-time-preparation-of-active-directory"></a>Préparation ponctuelle d’Active Directory

Si vous n’avez pas encore créé de gMSA dans votre domaine, vous devrez générer la clé racine du service de distribution de clés (KDS). Le KDS est responsable de la création, de la rotation et du lancement du mot de passe gMSA pour les hôtes autorisés. Lorsqu’un hôte de conteneur doit utiliser gMSA pour exécuter un conteneur, il contacte l’adresse KDS pour récupérer le mot de passe actuel.

Pour vérifier si la clé de racine KDS a déjà été créée, exécutez l’applet de commande PowerShell suivante en tant qu’administrateur de domaine sur un contrôleur de domaine ou un membre de domaine sur lequel les outils AD PowerShell sont installés:

```powershell
Get-KdsRootKey
```

Si la commande renvoie un ID clé, c’est que vous l’avez définie et que vous pouvez passer directement à la section [créer un compte de service géré du groupe](#create-a-group-managed-service-account) . Dans le cas contraire, continuez sur pour créer la clé racine de KDS.

Dans un environnement de production ou un environnement de test doté de plusieurs contrôleurs de domaine, exécutez l’applet de commande suivante dans PowerShell en tant qu’administrateur de domaine pour créer la clé racine de KDS.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

Même si la commande implique que la clé sera effective immédiatement, vous devrez patienter 10 heures avant la réplication de la clé racine de KDS et la rendre disponible pour une utilisation sur tous les contrôleurs de domaine.

Si vous n’avez qu’un seul contrôleur de domaine dans votre domaine, vous pouvez accélérer le processus en définissant la clé pour qu’elle soit effective 10 heures.

>[!IMPORTANT]
>N’utilisez pas cette technique dans un environnement de production.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>Créer un compte de service géré de groupe

Chaque conteneur qui utilise l’authentification Windows intégrée doit avoir au moins un gMSA. Le gMSA principal est utilisé chaque fois qu’une application s’exécute sur un système ou un service réseau pour accéder aux ressources du réseau. Le nom du gMSA deviendra le nom du conteneur sur le réseau, quel que soit le nom d’hôte attribué au conteneur. Les conteneurs peuvent également être configurés avec des gMSAs supplémentaires, au cas où vous souhaiteriez exécuter un service ou une application dans le conteneur sous forme d’identité différente de celle du compte d’ordinateur conteneur.

Lorsque vous créez un gMSA, vous créez également une identité partagée qui peut être utilisée simultanément sur différentes machines. L’accès au mot de passe gMSA est protégé par une liste de contrôle d’accès Active Directory. Nous vous recommandons de créer un groupe de sécurité pour chaque compte gMSA et d’ajouter les hôtes de conteneur pertinents au groupe de sécurité afin de limiter l’accès au mot de passe.

Enfin, dans la mesure où les conteneurs n’inscrivent pas automatiquement les noms de principal de service (SPN), vous devez créer manuellement au moins un SPN d’hôte pour votre compte gMSA.

En règle générale, le SPN hôte ou http est enregistré en utilisant le même nom que le compte gMSA, mais vous devrez peut-être utiliser un nom de service différent si les clients accèdent à l’application conteneur à partir d’un équilibreur de charge ou d’un nom DNS différent du nom gMSA.

Par exemple, si le compte gMSA est nommé «WebApp01», mais que les utilisateurs accèdent au site à `mysite.contoso.com`, `http/mysite.contoso.com` vous devez inscrire un nom de fichier SPN sur le compte gMSA.

Certaines applications nécessitent des SPN supplémentaires pour leurs protocoles uniques. Par exemple, SQL Server nécessite le `MSSQLSvc/hostname` SPN.

Le tableau suivant répertorie les attributs requis pour la création d’un gMSA.

|propriété gMSA | Valeur obligatoire | Exemple |
|--------------|----------------|--------|
|Nom | Tout nom de compte valide. | `WebApp01` |
|DnsHostName | Nom de domaine ajouté au nom du compte. | `WebApp01.contoso.com` |
|ServicePrincipalNames | Définissez au moins le SPN hôte, puis ajoutez d’autres protocoles selon vos besoins. | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | Le groupe de sécurité contenant les hôtes de votre conteneur. | `WebApp01Hosts` |

Une fois que vous avez décidé du nom de votre gMSA, exécutez les applets de commande suivantes dans PowerShell pour créer le groupe de sécurité et gMSA.

> [!TIP]
> Vous devez utiliser un compte qui appartient au groupe de sécurité des **administrateurs de domaine** ou qui a été délégué l’autorisation **créer des objets MSDS-GroupManagedServiceAccount** pour exécuter les commandes suivantes.
> L’applet de commande [New-ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) fait partie des outils ad PowerShell des [Outils d’administration de serveur distant](https://aka.ms/rsat).

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -GroupScope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

Nous vous recommandons de créer différents comptes gMSA pour vos environnements de développement, de test et de production.

## <a name="prepare-your-container-host"></a>Préparer votre hôte de conteneur

Chaque hôte de conteneur exécutant un conteneur Windows avec un gMSA doit être joint au domaine et avoir accès à la récupération du mot de passe gMSA.

1. Connectez-vous à votre ordinateur à votre domaine Active Directory.
2. Assurez-vous que votre hôte appartient au groupe de sécurité contrôlant l’accès au mot de passe gMSA.
3. Redémarrez l’ordinateur pour qu’il tire son nouvel appartenance au groupe.
4. Configurez la version [de bureau de l’amarrage pour Windows 10](https://docs.docker.com/docker-for-windows/install/) ou [docker pour Windows Server](https://docs.docker.com/install/windows/docker-ee/).
5. Recommande Vérifier que l’hôte peut utiliser le compte gMSA en exécutant [test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount). Si la commande retourne **false**, suivez les [instructions de dépannage](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa).

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>Créer une spécification d’informations d’identification

Il s’agit d’un document JSON contenant des métadonnées sur le ou les comptes gMSA que vous voulez qu’un conteneur utilise. En conservent la configuration d’identité indépendamment de l’image du conteneur, vous pouvez modifier la gMSA utilisée par le conteneur en échangeant simplement le fichier de spécifications d’informations d’identification, sans modification de code.

Le fichier de spécifications des informations d’identification est créé à l’aide du [module CredentialSpec PowerShell](https://aka.ms/credspec) d’un hôte de conteneur joint au domaine.
Une fois que vous avez créé le fichier, vous pouvez le copier dans d’autres hôtes de conteneur ou dans votre conteneur Orchestrator.
Le fichier de spécifications d’informations d’identification ne contient pas de secrets, tels que le mot de passe gMSA, dans la mesure où l’hôte de conteneur récupère le gMSA de la part du conteneur.

L’administrateur attend la recherche du fichier de spécifications d’informations d’identification sous le répertoire **CredentialSpecs** dans le répertoire de données de l’ancrage. Dans une installation par défaut, ce dossier se trouve à `C:\ProgramData\Docker\CredentialSpecs`l’adresse.

Pour créer un fichier de spécifications d’information d’identification sur votre hôte de conteneur:

1. Installer les outils de services d’annuaire publicitaire du RSAT
    - Pour Windows Server, exécutez **install-WINDOWSFEATURE RSAT-ad-PowerShell**.
    - Pour Windows 10, version 1809 ou ultérieure, exécutez **Add-WindowsCapability-Online-Name’RSAT. ActiveDirectory. DS-LDS. Tools ~ ~ ~ ~ 0.0.1.0 '**.
    - Pour les versions antérieures de Windows 10, <https://aka.ms/rsat>voir.
2. Exécutez l’applet de commande suivante pour installer la dernière version du [module CredentialSpec PowerShell](https://aka.ms/credspec):

    ```powershell
    Install-Module CredentialSpec
    ```

    Si vous ne disposez pas d’un accès à Internet sur l' `Save-Module CredentialSpec` hôte de votre conteneur, exécutez-le sur un ordinateur connecté `C:\Program Files\WindowsPowerShell\Modules` à Internet, puis `$env:PSModulePath` copiez le dossier du module dans ou dans un autre emplacement dans l’hôte du conteneur.

3. Exécutez l’applet de commande suivante pour créer le nouveau fichier de spécifications d’informations d’identification:

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    Par défaut, l’applet de passe crée une spécification cred en utilisant le nom gMSA fourni en tant que compte d’ordinateur pour le conteneur. Le fichier est enregistré dans le répertoire de l’CredentialSpecs de l’amarrage en utilisant le domaine gMSA et le nom du compte pour le nom de fichier.

    Vous pouvez créer une spécification d’informations d’identification incluant des comptes gMSA supplémentaires si vous exécutez un service ou un processus en tant que gMSA secondaire dans le conteneur. Pour ce faire, utilisez le `-AdditionalAccounts` paramètre:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    Pour obtenir la liste complète des paramètres pris en `Get-Help New-CredentialSpec`charge, exécutez.

4. Vous pouvez afficher une liste de toutes les spécifications d’informations d’identification et leur chemin d’accès complet avec l’applet de commande suivante:

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez configuré votre compte gMSA, vous pouvez l’utiliser pour:

- [Configurer des applications](gmsa-configure-app.md)
- [Conteneurs d’exécution](gmsa-run-container.md)
- [Orchestrer des conteneurs](gmsa-orchestrate-containers.md)

Si vous rencontrez des problèmes lors de l’installation, consultez notre [Guide de résolution des problèmes](gmsa-troubleshooting.md) pour consulter les solutions possibles.

## <a name="additional-resources"></a>Ressources supplémentaires

- Pour en savoir plus sur gMSAs, voir [vue d’ensemble des comptes de service géré par groupe](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).
- Pour obtenir une démonstration vidéo, regardez notre [démonstration enregistrée](https://youtu.be/cZHPz80I-3s?t=2672) sur enflammer 2016.

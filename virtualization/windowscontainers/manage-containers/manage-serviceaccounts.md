---
title: Comptes de Service administrés de groupe pour les conteneurs Windows
description: Comptes de Service administrés de groupe pour les conteneurs Windows
keywords: docker, conteneurs, active directory, le compte gmsa
author: rpsqrd
ms.date: 03/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 17c4089c98a74ea5937bac5d0eb4d4f1749aecf7
ms.sourcegitcommit: b8afbfb63c33a491d7bad44d8d5962e6a60cb566
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/22/2019
ms.locfileid: "9257445"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>Comptes de Service administrés de groupe pour les conteneurs Windows

Dans un réseau basé sur Windows, il est courant d’utiliser Active Directory (AD) pour faciliter l’authentification et autorisation entre les utilisateurs, les ordinateurs et autres ressources réseau. Les développeurs d’applications entreprise conçoivent souvent leurs applications doivent être intégrées à Active Directory et s’exécute sur des serveurs pour tirer parti de l’authentification Windows intégrée, ce qui facilite la création d’utilisateurs et d’autres services connecter automatiquement et en toute transparence joints au domaine l’application avec leur identité.

Bien que les conteneurs Windows ne peut pas être joints à un domaine, ils peuvent toujours utiliser les identités de domaine Active Directory pour prendre en charge différents scénarios de l’authentification.
Pour ce faire, vous pouvez configurer un conteneur Windows s’exécute avec un [groupe géré compte de service](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA): un type spécial d’introduite dans Windows Server 2012 est conçu pour permettre à plusieurs ordinateurs de partager une identité sans avoir besoin d’un compte de service Pour connaître son mot de passe.
Lorsque vous exécutez un conteneur avec un compte gMSA, l’hôte de conteneur récupère le mot de passe du compte gMSA à partir d’un contrôleur de domaine Active Directory et lui donne à l’instance de conteneur.
Le conteneur utilisera les informations d’identification de compte gMSA chaque fois que son compte d’ordinateur (SYSTEM) a besoin d’accéder aux ressources réseau.

Cet article explique comment commencer à utiliser des comptes de service géré de groupe Active Directory avec des conteneurs Windows.

## <a name="prerequisites"></a>Prérequis

Pour exécuter un conteneur Windows avec un compte de service géré de groupe, vous devez les éléments suivants:

**Un domaine Active Directory au moins un contrôleur de domaine exécutant Windows Server 2012 ou version ultérieure.**
Aucune forêt ou un domaine fonctionnel niveau est requise pour utiliser des comptes Gmsa, mais les mots de passe du compte gMSA ne peuvent être distribuée par des contrôleurs de domaine exécutant Windows Server 2012 ou version ultérieure.
Pour plus d’informations, voir [Configuration requise pour Active Directory pour les comptes Gmsa](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req).



**Autorisation de créer un compte gMSA.**
Pour créer un compte gMSA, vous devez être un administrateur de domaine ou utiliser un compte qui a été délégué l’autorisation de *créer des objets msDS-GroupManagedServiceAccount* .



**Accès à internet pour télécharger le module CredentialSpec PowerShell.**
Si vous travaillez dans un environnement déconnecté, vous pouvez [Enregistrer le module](https://docs.microsoft.com/en-us/powershell/module/powershellget/save-module?view=powershell-5.1) sur un ordinateur avec internet accéder et copiez-le sur votre hôte d’ordinateur ou un conteneur de développement.

## <a name="one-time-preparation-of-active-directory"></a>Préparation à usage unique d’Active Directory

Si vous n’avez pas encore créé un compte gMSA dans votre domaine, il est probable que devez générer la clé racine de Service de Distribution de clés.
Le clés (KDS) est responsable de la création, la rotation et le mot de passe du compte gMSA aux hôtes autorisés en libérant.
Quand un hôte de conteneur doit utiliser le compte gMSA pour exécuter un conteneur, il contacte le KDS pour récupérer le mot de passe en cours.

Pour vérifier si les KDS racine clé a déjà été créé, exécutez la commande PowerShell suivante en tant qu' *Administrateur de domaine* sur un contrôleur de domaine ou un membre de domaine et les outils AD PowerShell installés:

```powershell
Get-KdsRootKey
```

Si la commande retourne une clé d’ID, que vous êtes ensemble tous les et vous pouvez passer à la section [Création d’un compte de service géré de groupe](#create-a-group-managed-service-account) .
Sinon, passez à créer la clé racine de clés (KDS).

Dans un environnement de test avec plusieurs contrôleurs de domaine ou un environnement de production, exécutez la commande suivante dans PowerShell en tant qu' *Administrateur de domaine* pour créer la clé racine de clés (KDS).
Bien que la commande implique la clé sont effective immédiatement, vous devez attendre 10 heures avant que la clé racine de clés (KDS) est répliquées et disponibles pour une utilisation sur tous les contrôleurs de domaine.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

Si vous avez uniquement un contrôleur de domaine de votre domaine, vous pouvez accélérer le processus en définissant la clé pour être efficaces 10 heures auparavant.
N’utilisez pas cette technique dans un environnement de production.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>Créer un compte de service géré de groupe

Chaque conteneur qui utilise l’authentification Windows intégrée doit au moins un compte gMSA.
Le compte gMSA principal est utilisé chaque fois que les applications s’exécutant en tant que *SERVICE réseau* ou *système* accéder aux ressources sur le réseau.
Le nom du compte gMSA sera le nom du conteneur sur le réseau, quel que soit le nom d’hôte attribué au conteneur.
Les conteneurs peuvent également être configurés avec des comptes Gmsa supplémentaires, au cas où vous souhaitez exécuter un service ou une application dans le conteneur en tant qu’une identité différente à partir du compte d’ordinateur de conteneur.

Lorsque vous créez un compte gMSA, vous créez une identité partagée pouvant être utilisées simultanément sur différents ordinateurs.
Accès au mot de passe du compte gMSA est protégé par une liste de contrôle d’accès annuaire Active.
Nous vous recommandons de créer un groupe de sécurité pour chaque compte gMSA et en ajoutant les hôtes de conteneur approprié au groupe de sécurité pour limiter l’accès au mot de passe.

Enfin, dans la mesure où les conteneurs n’inscrivent pas automatiquement n’importe quel nom Principal de Service (SPN), vous devrez créer manuellement au moins un SPN «hôte» pour votre compte gMSA.
En règle générale, l’hôte ou http SPN est inscrite à l’aide du même nom en tant que compte gMSA, mais vous devrez peut-être utiliser un nom de service différents si les clients à l’application en conteneur situé derrière un équilibreur de charge ou le nom DNS qui est différent du nom du compte gMSA.
Par exemple, si le compte gMSA est *WebApp01* , mais vos utilisateurs accèdent au site à *mysite.contoso.com* , vous devez inscrire un `http/mysite.contoso.com` SPN sur le compte gMSA.
Certaines applications peuvent nécessiter des SPN supplémentaires pour leurs protocoles uniques, par exemple, SQL Server nécessite le SPN «MSSQLSvc/nom d’hôte».

Le tableau suivant répertorie les attributs requis lors de la création d’un compte gMSA.

Propriété de compte gMSA | Valeur requise | Exemple
--------------|----------------|--------
Nom | N’importe quel nom de compte valide. | `WebApp01`
DnsHostName | Le nom de domaine ajouté au nom du compte. | `WebApp01.contoso.com`
ServicePrincipalNames | Définir au moins l’hôte SPN, ajouter d’autres protocoles si nécessaire. | `'host/WebApp01', 'host/WebApp01.contoso.com'`
PrincipalsAllowedToRetrieveManagedPassword | Le groupe de sécurité contenant vos hôtes de conteneur. | `WebApp01Hosts`

Une fois que vous savez ce que vous voulez appeler votre compte gMSA, exécutez les commandes suivantes dans PowerShell pour créer le groupe de sécurité et d’un compte gMSA.

> [!TIP]
> Vous devez utiliser un compte auquel appartient le groupe de sécurité **Admins du domaine** ou a été délégué l’autorisation de **créer des objets msDS-GroupManagedServiceAccount** pour exécuter les commandes suivantes.
> L’applet de commande [New-ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) fait partie des outils AD PowerShell à partir des [Outils d’Administration de serveur distant](https://aka.ms/rsat).

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -Scope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

Il est recommandé de créer des comptes gMSA distinct pour vos environnements de développement, test et de production.

## <a name="prepare-your-container-host"></a>Préparer votre hôte de conteneur

Chaque hôte de conteneur qui s’exécute un conteneur Windows avec un compte gMSA doit être joint au domaine et accès aux récupérer le mot de passe du compte gMSA.

1.  Joindre votre ordinateur à votre domaine Active Directory.
2.  Assurez-vous que votre hôte appartient au groupe de sécurité contrôler l’accès au mot de passe du compte gMSA.
3.  Redémarrez l’ordinateur afin qu’il obtient son appartenance à un nouveau groupe.
4.  Configurer [Docker bureau pour Windows 10](https://docs.docker.com/docker-for-windows/install/) ou [Docker pour Windows Server](https://docs.docker.com/install/windows/docker-ee/).
5.  (Recommandé) Vérifiez que l’hôte peut utiliser le compte gMSA en exécutant le [Test-ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/activedirectory/test-adserviceaccount). Si la commande retourne la **valeur False**, consultez la section [résolution des problèmes](#troubleshooting) de diagnostic comme suit.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>Créer une spécification d’informations d’identification

Un fichier de spécification d’informations d’identification est un document JSON contenant des métadonnées sur les comptes gMSA que vous voulez un conteneur à utiliser.
En maintenant la configuration de l’identité distinct à partir de l’image de conteneur, vous pouvez facilement modifier le compte gMSA utilisé par le conteneur en remplaçant simplement le fichier spécification d’informations d’identification: aucune modification du code nécessaire.

Le fichier de spécification d’informations d’identification est créé à l’aide du [module PowerShell CredentialSpec](https://aka.ms/credspec) sur un hôte de conteneur joint au domaine.
Une fois que vous avez créé le fichier, vous pouvez le copier d’autres hôtes de conteneur ou votre orchestrateur de conteneur.
Le fichier de spécification d’informations d’identification ne contient pas les secrets, par exemple, le mot de passe du compte gMSA, dans la mesure où l’hôte de conteneur récupère le compte gMSA pour le compte du conteneur.

Docker s’attend à trouver le fichier de spécification d’informations d’identification sous le répertoire **CredentialSpecs** dans le répertoire de données de Docker.
Dans une installation par défaut, vous trouverez ce dossier à `C:\ProgramData\Docker\CredentialSpecs`.

Procédez comme suit pour créer un fichier de spécification d’informations d’identification sur votre hôte de conteneur:
1.  Installer les outils RSAT AD PowerShell
    -   Pour Windows Server, exécutez `Install-WindowsFeature RSAT-AD-PowerShell`
    -   Pour Windows 10, version 1809 ou une version ultérieure, exécutez `Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'`
    -   Pour les versions antérieures de Windows 10, voirhttps://aka.ms/rsat
2.  Installer la dernière version du [module PowerShell CredentialSpec](https://aka.ms/credspec):

    ```powershell
    Install-Module CredentialSpec
    ```

    Si vous n’avez pas accès à internet sur votre hôte de conteneur, exécutez `Save-Module CredentialSpec` sur un ordinateur connecté à internet et copiez le dossier module à `C:\Program Files\WindowsPowerShell\Modules` ou un autre emplacement dans `$env:PSModulePath` sur l’hôte de conteneur.

3.  Créer le nouveau fichier de spécification d’informations d’identification:

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    Par défaut, l’applet de commande crée une spécification de l’Assistant en utilisant le nom de compte gMSA fourni en tant que le compte d’ordinateur pour le conteneur.
    Le fichier sera être enregistré dans le répertoire Docker CredentialSpecs en utilisant le nom de domaine et le compte gMSA pour le nom de fichier.

    Vous pouvez créer une spécification d’informations d’identification qui inclut des comptes gMSA supplémentaires si vous utilisez un service ou un processus en tant qu’un compte gMSA secondaire dans le conteneur.
    Pour ce faire, utilisez le `-AdditionalAccounts` paramètre:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    Pour obtenir une liste complète des paramètres pris en charge, exécutez `Get-Help New-CredentialSpec`.

4.  Vous pouvez afficher une liste de toutes les informations d’identification spécifications et leur chemin d’accès complet avec la commande suivante:

    ```powershell
    Get-CredentialSpec
    ```

## <a name="configuring-your-application-to-use-the-gmsa"></a>Configurer votre application pour utiliser le compte gMSA

Dans la configuration typique, un conteneur est fourni uniquement dans un seul compte gMSA qui est utilisé chaque fois que le compte d’ordinateur conteneur essaie de s’authentifier aux ressources réseau.
Cela signifie que votre application doit être exécuté en tant que **Système Local** ou **Service réseau** s’il a besoin d’utiliser l’identité du compte gMSA.

### <a name="run-an-iis-app-pool-as-network-service"></a>Exécuter un Pool d’application IIS en tant que Service réseau

Si vous hébergez un site Web IIS dans votre conteneur, il vous souhaitez pour tirer parti du compte gMSA est défini votre identité du pool d’application au **Service réseau**.
Vous pouvez effectuer qui dans votre fichier Dockerfile en ajoutant la commande suivante:

```dockerfile
RUN (Get-IISAppPool DefaultAppPool).ProcessModel.IdentityType = "NetworkService"
```

Si vous avez utilisé précédemment des informations d’identification utilisateur statique pour votre Pool d’applications IIS, envisagez le compte gMSA en tant que la suppression de certaines de ces informations d’identification.
Vous pouvez modifier le compte gMSA entre les environnements de développement, test et de production et IIS reprend automatiquement à l’identité actuelle sans avoir à modifier l’image de conteneur.

### <a name="run-a-windows-service-as-network-service"></a>Exécuter un Service Windows en tant que Service réseau

Si votre application en conteneur s’exécute comme un Service Windows, vous pouvez définir le service s’exécute en tant que **Service réseau** dans votre fichier Dockerfile:

```dockerfile
RUN cmd /c 'sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""'
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>Exécuter des applications de console arbitraire en tant que Service réseau

Pour les applications de console générique qui ne sont pas hébergées dans IIS ou dans le Gestionnaire des services, il est souvent plus facile d’exécuter le conteneur en tant que **Service réseau** afin que l’application hérite automatiquement le contexte du compte gMSA.
Cette fonctionnalité est disponible à compter de Windows Server version 1709.

Ajoutez la ligne suivante à votre fichier Dockerfile pour que celle-ci s’exécute en tant que Service réseau par défaut:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

Vous pouvez également vous connecter à un conteneur en tant que Service réseau sur une base ponctuelles avec `docker exec`.
Cela est particulièrement utile si vous êtes résoudre des problèmes de connectivité dans un conteneur en cours d’exécution lorsque le conteneur ne s’exécute pas normalement en tant que Service réseau.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="run-a-container-with-a-gmsa"></a>Exécuter un conteneur avec un compte gMSA

Pour exécuter un conteneur avec un compte gMSA, fournissez le fichier de spécification d’informations d’identification pour le `--security-opt` paramètre [d’exécution de docker](https://docs.docker.com/engine/reference/run):

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Sur Windows Server 2016, 1709 et 1803, le nom d’hôte du conteneur *doit correspondre à* la gMSA nom abrégé.
Dans l’exemple ci-dessus, le nom de compte SAM compte gMSA est «webapp01» afin que le nom d’hôte de conteneur est défini sur le même.
Sur Windows Server 2019 et versions ultérieures, le champ de nom d’hôte n’est pas obligatoire, mais le conteneur toujours identifie elle-même par le nom du compte gMSA plutôt que le nom d’hôte, même si vous fournissez explicitement une autre.

Pour vérifier si le compte gMSA fonctionne correctement, exécutez la commande suivante dans le conteneur:

```
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

Si *l’État de connexion de contrôleurs de domaine approuvé* et *Approuver le statut de vérification* ne sont pas *NERR_Success*, consultez la section [résolution des problèmes](#troubleshooting) pour obtenir des conseils sur la manière de déboguer le problème.

Vous pouvez vérifier l’identité de compte gMSA à partir du conteneur en exécutant la commande suivante et en vérifiant le nom du client:

```
PS C:\> klist get krbtgt

Current LogonId is 0:0xaa79ef8
A ticket to krbtgt has been retrieved successfully.

Cached Tickets: (2)

#0>     Client: webapp01$ @ CONTOSO.COM
        Server: krbtgt/webapp01 @ CONTOSO.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
        Start Time: 3/21/2019 4:17:53 (local)
        End Time:   3/21/2019 14:17:53 (local)
        Renew Time: 3/28/2019 4:17:42 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called: dc01.contoso.com

[...]
```

Pour ouvrir PowerShell ou une autre application de console en tant que compte gMSA, vous pouvez demander le conteneur s’exécute sous le compte au lieu de la normale ContainerAdministrator (ou ContainerUser pour NanoServer) compte de Service réseau:

```powershell
# NOTE: you can only run as SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Lorsque vous exécutez en tant que Service réseau, vous pouvez tester l’authentification réseau en tant que compte gMSA en essayant de se connecter à SYSVOL sur un contrôleur de domaine:

```
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="orchestrating-containers-with-gmsa"></a>Orchestration de conteneurs avec le compte gMSA

Dans les environnements de production, vous utiliserez souvent un orchestrateur de conteneur pour déployer et gérer vos applications et services.
Chaque orchestrator a ses propres paradigmes de gestion et est responsable de l’acceptation des spécifications des informations d’identification pour donner à la plateforme de conteneur Windows.

Lorsque vous êtes orchestration de conteneurs avec des comptes Gmsa, vérifiez que:
> [!div class="checklist"]
> * Tous les hôtes de conteneur qui peuvent être planifiées pour exécuter des conteneurs avec des comptes Gmsa sont joints au domaine
> * Les hôtes de conteneur ont accès à récupérer les mots de passe pour tous les comptes Gmsa utilisée par les conteneurs
> * Les fichiers de spécification d’informations d’identification sont créés et chargés dans l’orchestrateur ou copiés sur chaque hôte de conteneur, en fonction de l’orchestrateur préfère comment gérer de manière.
> * Réseaux de conteneur autorisent les conteneurs communiquer avec les contrôleurs de domaine Active Directory pour récupérer les tickets de compte gMSA

### <a name="using-gmsa-with-service-fabric"></a>À l’aide du compte gMSA avec Service Fabric

Service Fabric prend en charge des conteneurs Windows en cours d’exécution avec un compte gMSA lorsque vous spécifiez l’emplacement de spécification d’informations d’identification dans votre manifeste d’application.
Vous devez créer le fichier de spécification d’informations d’identification et le placer dans le sous-répertoire **CredentialSpecs** du répertoire de données Docker sur chaque hôte afin que le Service Fabric puisse la localiser.
Le `Get-CredentialSpec` applet de commande, partie du [module PowerShell CredentialSpec](https://aka.ms/credspec), peut être utilisé pour vérifier si votre spécification d’informations d’identification est à l’emplacement approprié.

Voir [démarrage rapide: déployer des conteneurs Windows sur le Service Fabric](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-quickstart-containers) et [configurer le compte gMSA pour les conteneurs Windows en cours d’exécution sur le Service Fabric](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) pour plus d’informations sur la configuration de votre application.

### <a name="using-gmsa-with-docker-swarm"></a>À l’aide du compte gMSA avec Docker Swarm

Pour utiliser un compte gMSA avec des conteneurs gérés par Docker Swarm, utilisez la commande [de création du service de docker](https://docs.docker.com/engine/reference/commandline/service_create/) avec la `--credential-spec` paramètre:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Consultez l' [exemple de Docker Swarm](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) pour plus d’informations sur l’utilisation des informations d’identification spécifications avec les services Docker.

### <a name="using-gmsa-with-kubernetes"></a>À l’aide du compte gMSA avec Kubernetes

Prise en charge pour la planification des conteneurs Windows avec des comptes Gmsa dans Kubernetes est prise en charge alpha à compter de Kubernetes 1.14.
Vérifiez les [Windows comptes pour conteneur identité Web en haut](https://github.com/kubernetes/enhancements/blob/master/keps/sig-windows/20181221-windows-group-managed-service-accounts-for-container-identity.md) pour les dernières informations sur cette fonctionnalité et des informations sur la façon de la tester dans votre distribution Kubernetes.

## <a name="example-uses"></a>Exemples d’utilisation

### <a name="sql-connection-strings"></a>Chaînes de connexion SQL
Lorsqu’un service est exécuté en tant que système local ou service réseau dans un conteneur, il peut utiliser l’authentification intégrée de Windows pour se connecter à un serveur MicrosoftSQLServer.

Voici un exemple de chaîne de connexion qui utilise l’identité de conteneur pour s’authentifier auprès de SQL Server:

```
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

Sur le serveur MicrosoftSQLServer, créez une connexion en utilisant le nom du domaine et du compte gMSA, suivis d’un symbole$.
Une fois la connexion créée, elle peut être ajoutée à un utilisateur dans une base de données et bénéficier des autorisations d’accès appropriées.

Exemple: 

```sql
CREATE LOGIN "DEMO\WebApplication1$"
    FROM WINDOWS
    WITH DEFAULT_DATABASE = "MusicStore"
GO

USE MusicStore
GO
CREATE USER WebApplication1 FOR LOGIN "DEMO\WebApplication1$"
GO

EXEC sp_addrolemember 'db_datareader', 'WebApplication1'
EXEC sp_addrolemember 'db_datawriter', 'WebApplication1'
```

Pour la voir en action, découvrez la [démonstration enregistrée](https://youtu.be/cZHPz80I-3s?t=2672) disponible dans Microsoft Ignite2016 dans la session «Sur la voie de la conteneurisation: transformation des charges de travail en conteneurs».

## <a name="troubleshooting"></a>Résolution des problèmes

### <a name="known-issues"></a>Problèmes connus

**Nom d’hôte de conteneur doit correspondre au nom de compte gMSA pour WS2016, 1709 et 1803.**

Si vous exécutez Windows Server 2016, 1709, ou 1803, le nom d’hôte du conteneur doit correspondre à votre nom de compte SAM compte gMSA.
Lorsque le nom d’hôte ne correspond pas le nom du compte gMSA, entrant des demandes d’authentification NTLM et la traduction de nom/SID (utilisé par nombreuses bibliothèques, telles que le fournisseur de rôle d’appartenance ASP.NET) échouera.
Kerberos continueront à fonctionner normalement, même si le nom d’hôte ne correspond pas.

Cette limitation a été résolue dans Windows Server 2019, où le conteneur désormais utilisent toujours son nom de compte gMSA sur le réseau, quel que soit le nom d’hôte affecté.

**À l’aide d’un compte gMSA avec plusieurs conteneurs simultanément entraîne des pannes intermittentes sur WS2016, 1709 et 1803.**

Suite à la limitation antérieure nécessiter de tous les conteneurs d’utiliser le même nom d’hôte, un second problème concerne les versions de Windows avant Windows Server 2019 et Windows 10 version 1809.
Lorsque plusieurs conteneurs sont affectés à la même identité et le nom d’hôte, une condition de concurrence peut se produire lorsque deux conteneurs de communiquer avec le contrôleur de domaine même simultanément.
Lorsqu’un autre conteneur communique avec le même contrôleur de domaine, il annule la communication avec des conteneurs préalables à l’aide de la même identité.
Cela peut entraîner des échecs d’authentification et peut parfois être observé en tant qu’une erreur d’approbation lorsque vous exécutez `nltest /sc_verify:contoso.com` à l’intérieur du conteneur.

Nous avons modifié le comportement dans Windows Server 2019 pour séparer l’identité du conteneur du nom de l’ordinateur, ce qui permet de plusieurs conteneurs d’utiliser le même compte gMSA simultanément.

**Vous ne pouvez pas utiliser des comptes Gmsa aux conteneurs isolé Hyper-V sur Windows versions 1703 et 1709, 1803.**

Initialisation du conteneur est se bloquer ou échoue lorsque vous essayez d’utiliser un compte gMSA avec un conteneur isolé Hyper-V sur Windows 10 et Windows Server versions 1703 et 1709 de 1803.

Ce problème a été résolu dans Windows Server 2019 et Windows 10, version 1809. Vous pouvez également exécuter des conteneurs Hyper-V isolé avec des comptes Gmsa sur Windows Server 2016 et Windows 10, version 1607.

### <a name="general-troubleshooting-guidance"></a>Conseils de dépannage générales

Si vous vous rencontrez des erreurs lors de l’exécution d’un conteneur avec un compte gMSA, les étapes suivantes peuvent vous aider à identifier la cause.

**Assurez-vous que l’hôte peut utiliser le compte gMSA**

1.  Vérifiez que l’hôte est joint au domaine et de peut contacter le contrôleur de domaine.
2.  Installer les outils AD PowerShell de serveur distant et exécutez [Test-ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/activedirectory/test-adserviceaccount) pour voir si l’ordinateur a accès à récupérer le compte gMSA. Si l’applet de commande renvoie **False**, l’ordinateur n’a pas accès au mot de passe du compte gMSA.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```
3.  Si le Test-ADServiceAccount retourne la **valeur False**, vérifiez que l’hôte appartient à un groupe de sécurité qui a accès à récupérer le mot de passe du compte gMSA.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4.  Si votre hôte appartient à un groupe de sécurité autorisé à récupérer le mot de passe du compte gMSA, mais toujours défaillant Test-ADServiceAccount, vous devrez peut-être redémarrer votre ordinateur pour qu’il puisse obtenir un nouveau ticket reflétant son appartenance aux groupes en cours.

**Vérifier le fichier de la spécification d’informations d’identification**

1.  Exécutez `Get-CredentialSpec` toutes les informations d’identification à partir du [PowerShell Module de CredentialSpec](https://aka.ms/credspec) pour localiser les spécifications sur l’ordinateur. Les spécifications des informations d’identification doivent être stockées dans le répertoire «CredentialSpecs» sous le répertoire racine de Docker. Vous pouvez trouver la Docker répertoire racine en exécutant `docker info -f "{{.DockerRootDir}}"`.
2.  Ouvrez le fichier CredentialSpec et assurez-vous que les champs suivants sont remplis correctement:
    -   **SID**: le SID de votre compte gMSA
    -   **MachineAccountName**: le nom de compte SAM compte gMSA (n’incluez pas nom de domaine complet ou une enseigne estimé en dollars)
    -   **DnsTreeName**: le nom de domaine complet de votre forêt AD
    -   **DnsName**: le nom de domaine complet du domaine auquel appartient le compte gMSA
    -   **Nom_netbios**: nom NETBIOS pour le domaine auquel appartient le compte gMSA
    -   **Nom/GroupManagedServiceAccounts**: le nom de compte SAM compte gMSA (n’incluez pas nom de domaine complet ou une enseigne estimé en dollars)
    -   **GroupManagedServiceAccounts/étendue**: une entrée pour le nom de domaine complet du domaine et l’autre pour le NETBIOS

    Voir l’exemple complet ci-dessous pour une spécification d’informations d’identification complet:

    ```json
    {
    "CmsPlugins":[
        "ActiveDirectory"
    ],
    "DomainJoinConfig":{
        "Sid":"S-1-5-21-702590844-1001920913-2680819671",
        "MachineAccountName":"webapp01",
        "Guid":"56d9b66c-d746-4f87-bd26-26760cfdca2e",
        "DnsTreeName":"contoso.com",
        "DnsName":"contoso.com",
        "NetBiosName":"CONTOSO"
    },
    "ActiveDirectoryConfig":{
        "GroupManagedServiceAccounts":[
        {
            "Name":"webapp01",
            "Scope":"contoso.com"
        },
        {
            "Name":"webapp01",
            "Scope":"CONTOSO"
        }
        ]
    }
    }
    ```

3.  Vérifiez que le chemin d’accès au fichier de spécification d’informations d’identification est approprié à votre solution d’orchestration. Si vous utilisez Docker, assurez-vous que le conteneur, exécutez la commande inclut `--security-opt="credentialspec=file://NAME.json"`, où «NAME.json» est remplacé par la sortie du nom par **Get-CredentialSpec**. Le nom est un nom de fichier plat, relatif au dossier CredentialSpecs sous le répertoire racine de Docker.

**Vérifier le conteneur**

1.  Si vous exécutez une version de Windows avant Windows Server 2019 ou Windows 10, version 1809, votre nom d’hôte de conteneur doit correspondre au nom de compte gMSA. Vérifiez le `--hostname` paramètre correspond au nom court gMSA (aucun composant de domaine, par exemple, «webapp01» pas «webapp01.contoso.com»).

2.  Vérifier la configuration de mise en réseau de conteneur pour vérifier le conteneur peut résoudre et accéder à un contrôleur de domaine du compte gMSA. Les serveurs DNS mal configurés dans le conteneur sont un coupable courantes des problèmes d’identité.

3.  Vérifiez si le conteneur dispose d’une connexion valide au domaine en exécutant la commande suivante dans le conteneur (à l’aide de `docker exec` ou un équivalent):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    La vérification d’approbation doit retourner NERR_SUCCESS si le compte gMSA est disponible et la connectivité réseau permet au conteneur communiquer avec le domaine.
    Si elle échoue, vérifiez la configuration du réseau de l’hôte et le conteneur: les deux devront être en mesure de communiquer avec le contrôleur de domaine.

4.  Assurez-vous que votre application est [configuré pour utiliser le compte gMSA](#configuring-your-application-to-use-the-gmsa). Le compte d’utilisateur à l’intérieur du conteneur ne change pas lorsque vous utilisez un compte gMSA: au lieu de cela, le compte système utilise le compte gMSA lorsqu’il communique avec d’autres ressources réseau. Par conséquent, votre application doit être exécuté en tant que Service réseau ou système Local pour tirer parti de l’identité du compte gMSA.

    > [!TIP]
    > Si vous exécutez `whoami` ou utilisez un autre outil pour essayer et identifier votre contexte utilisateur actuel dans le conteneur, vous ne verrez jamais le nom du compte gMSA lui-même.
    > Il s’agit dans la mesure où vous vous connectez toujours dans le conteneur en tant qu’utilisateur local--pas une identité de domaine.
    > Le compte gMSA est utilisé par le compte d’ordinateur chaque fois qu’il communique avec les ressources réseau, c’est pourquoi votre application doit être exécutée en tant que Service réseau ou système Local.

5.  Enfin, si votre conteneur semble être configurée correctement, mais les utilisateurs ou autres services ne parvenez pas à s’authentifier automatiquement à votre application en conteneur, vérifiez les SPN sur votre compte gMSA. Clients localisera le compte gMSA par le nom à laquelle il a atteint votre application. Cela peut signifier que vous ayez besoin supplémentaires `host` SPN pour votre compte gMSA si, par exemple, les clients se connectent à votre application via un équilibreur de charge ou autre nom DNS.

## <a name="additional-resources"></a>Ressources supplémentaires
-   [Vue d’ensemble des comptes de Service administrés de groupe](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)

---
title: Grouper les comptes de service gérés pour les conteneurs Windows
description: Grouper les comptes de service gérés pour les conteneurs Windows
keywords: docker, conteneurs, Active Directory, GMSA
author: rpsqrd
ms.date: 05/23/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 8f184e58743bd41ab208b530976772c5fcffd189
ms.sourcegitcommit: 8e7fba17c761bf8f80017ba7f9447f986a20a2a7
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/24/2019
ms.locfileid: "9677318"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>Grouper les comptes de service gérés pour les conteneurs Windows

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
# To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -Scope DomainLocal

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
5. Recommande Vérifier que l’hôte peut utiliser le compte gMSA en exécutant [test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount). Si la commande retourne **false**, consultez la section [résolution des problèmes](#troubleshooting) de diagnostic.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
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
    - Pour Windows 10, version 1809 ou ultérieure, exécutez **install-WindowsCapability-online’RSAT. ActiveDirectory. DS-LDS. Tools ~ ~ ~ ~ 0.0.1.0 '**.
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

## <a name="configure-your-application-to-use-the-gmsa"></a>Configurer votre application pour utiliser gMSA

Dans le cas d’une configuration classique, un conteneur n’est fourni qu’à un seul compte gMSA, qui est utilisé chaque fois que le compte d’ordinateur conteneur tente d’s’authentifier aux ressources réseau. Cela signifie que votre application doit s’exécuter en tant que service **système local** ou **service réseau** si elle doit utiliser l’identité gMSA.

### <a name="run-an-iis-app-pool-as-network-service"></a>Exécuter un pool d’applications IIS en tant que service réseau

Si vous hébergez un site Web IIS dans votre conteneur, il vous suffit de définir l’identité de votre pool d’applications sur **service réseau**pour pouvoir utiliser le gMSA. Pour cela, vous pouvez ajouter la commande suivante dans votre Dockerfile:

```dockerfile
RUN (Get-IISAppPool DefaultAppPool).ProcessModel.IdentityType = "NetworkService"
```

Si vous utilisiez précédemment des informations d’identification d’utilisateur statique pour votre pool d’applications IIS, considérez le gMSA comme remplacement de ces informations d’identification. Vous pouvez modifier l’gMSA entre environnements de développement, de test et de production, et les services Internet (IIS) sélectionnent automatiquement l’identité actuelle sans avoir à modifier l’image du conteneur.

### <a name="run-a-windows-service-as-network-service"></a>Exécuter un service Windows en tant que service réseau

Si votre application conteneur s’exécute en tant que service Windows, vous pouvez définir le service pour qu’il s’exécute en tant que service **réseau** dans votre Dockerfile:

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>Exécuter des applications consoles arbitraires en tant que service réseau

Pour les applications de console génériques qui ne sont pas hébergées dans les services Internet (IIS) et le Service Manager, il est souvent plus facile d’exécuter le conteneur en tant que **service réseau** , ce qui permet à l’application d’hériter automatiquement du contexte gMSA. Cette fonctionnalité est disponible sous Windows Server version 1709.

Ajoutez la ligne suivante à votre Dockerfile pour qu’elle s’exécute en tant que service réseau par défaut:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

Vous pouvez également vous connecter à un conteneur en tant que service réseau en fonction d’un `docker exec`seul arrêt. Cela est particulièrement utile si vous rencontrez des problèmes de connectivité dans un conteneur en cours d’exécution lorsque le conteneur ne s’exécute pas normalement en tant que service réseau.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="run-a-container-with-a-gmsa"></a>Exécuter un conteneur avec un gMSA

Pour exécuter un conteneur avec un gMSA, spécifiez le fichier de spécifications d' `--security-opt` informations d' [](https://docs.docker.com/engine/reference/run)identification sur le paramètre de l’administrateur.

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>Sur Windows Server 2016 versions 1709 et 1803, le nom d’hôte du conteneur doit correspondre au nom court gMSA.

Dans l’exemple précédent, le nom du compte SAM gMSA est «webapp01», donc le nom d’hôte du conteneur est également appelé «webapp01».

Sur Windows Server 2019 et les versions ultérieures, le champ hostname n’est pas obligatoire, mais le conteneur restera identifiant par le nom gMSA à la place du nom d’hôte, même si vous en spécifiez explicitement un autre.

Pour vérifier si le fonctionnement correct de gMSA fonctionne, exécutez l’applet de commande suivante dans le conteneur:

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

Si l’état de la connexion de contrôleur de confiance et l' `NERR_Success`état de vérification de confiance ne sont pas répertoriés, consultez la section [résolution des problèmes](#troubleshooting) pour savoir comment résoudre le problème.

Vous pouvez vérifier l’identité gMSA à partir du conteneur en exécutant la commande suivante et en vérifiant le nom du client:

```powershell
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

Pour ouvrir PowerShell ou une autre application de console en tant que compte gMSA, vous pouvez demander à l’application de s’exécuter sous le compte de service réseau au lieu du compte ContainerAdministrator normal (ou ContainerUser pour le serveur de serveur):

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Lorsque vous exécutez le service réseau, vous pouvez tester l’authentification réseau en tant que gMSA en tentant de se connecter à SYSVOL sur un contrôleur de domaine:

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="orchestrate-containers-with-gmsa"></a>Orchestrer des conteneurs avec gMSA

Dans les environnements de production, vous utilisez souvent un conteneur Orchestrator pour déployer et gérer vos applications et services. Chaque Orchestrator dispose de ses propres paradigmes de gestion et est tenu d’accepter les spécifications d’informations d’identification à transmettre à la plateforme de conteneur Windows.

Lorsque vous orchestrez des conteneurs avec gMSAs, assurez-vous que:

> [!div class="checklist"]
> * Tous les hôtes de conteneur qui peuvent être planifiés pour exécuter des conteneurs avec gMSAs sont joints au domaine
> * Les hôtes de conteneur ont accès pour récupérer les mots de passe de tous les gMSAs utilisés par les conteneurs.
> * Les fichiers de spécification d’informations d’identification sont créés et téléchargés vers l’Orchestrator ou copiés sur chaque hôte de conteneur, en fonction de la façon dont l’Orchestrator préfère les gérer.
> * Les réseaux de conteneurs permettent aux conteneurs de communiquer avec les contrôleurs de domaine Active Directory pour récupérer les tickets gMSA

### <a name="how-to-use-gmsa-with-service-fabric"></a>Utiliser gMSA avec service Fabric

Service Fabric prend en charge l’exécution de conteneurs Windows avec un gMSA lorsque vous spécifiez l’emplacement des spécifications d’informations d’identification dans le manifeste de votre application. Vous devez créer le fichier de spécifications d’information d’identification et le placer dans le sous-répertoire **CredentialSpecs** du répertoire de données de l’ancrage sur chaque hôte de manière à ce que le service Fabric puisse le Rechercher. Vous pouvez exécuter l’applet de connexion **Get-CredentialSpec** , qui fait partie du [module CredentialSpec PowerShell](https://aka.ms/credspec), pour vérifier si les spécifications d’informations d’identification se trouvent à l’emplacement approprié.

Pour plus d’informations sur la configuration de votre application, voir [démarrage rapide: déploiement de conteneurs Windows sur le service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) et configuration [de gMSA pour les conteneurs Windows en cours d’exécution sur le service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) .

### <a name="how-to-use-gmsa-with-docker-swarm"></a>Utiliser gMSA avec l’essaimeur d’amarrage

Pour utiliser une gMSA avec des conteneurs gérés par essaim d’amarrage, exécutez la commande de création du service `--credential-spec` d' [ancrage](https://docs.docker.com/engine/reference/commandline/service_create/) avec le paramètre:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Pour plus d’informations sur l’utilisation des spécifications d’information d’identification avec les services d’amarrage, voir l' [exemple d’ancrage de dock](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) .

### <a name="how-to-use-gmsa-with-kubernetes"></a>Utiliser gMSA avec Kubernetes

La prise en charge de la planification de conteneurs Windows avec gMSAs dans Kubernetes est disponible sous la forme d’une fonctionnalité alpha dans Kubernetes 1,14. Pour plus d’informations sur cette fonctionnalité, voir [configurer gMSA pour les gousses et conteneurs Windows](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) et savoir comment la tester dans votre distribution Kubernetes.

## <a name="example-uses"></a>Exemple utilise

### <a name="sql-connection-strings"></a>Chaînes de connexion SQL

Lorsqu’un service est exécuté en tant que système local ou service réseau dans un conteneur, il peut utiliser l’authentification intégrée de Windows pour se connecter à un serveur MicrosoftSQLServer.

Voici un exemple de chaîne de connexion qui utilise l’identité de conteneur pour s’authentifier auprès de SQL Server:

```sql
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

Sur le serveur MicrosoftSQLServer, créez une connexion en utilisant le nom du domaine et du compte gMSA, suivis d’un symbole$. Une fois que vous avez créé l’ouverture de session, vous pouvez l’ajouter à un utilisateur sur une base de données et lui accorder des autorisations d’accès appropriées.

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

Pour le voir en action, consultez la [démonstration enregistrée](https://youtu.be/cZHPz80I-3s?t=2672) proposée par Microsoft entourage 2016 dans la session "suivre le chemin d’accès du conteneur-transformer les charges de travail en conteneurs".

## <a name="troubleshooting"></a>Résolution des problèmes

### <a name="known-issues"></a>Problèmes connus

#### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Le nom d’hôte du conteneur doit correspondre au nom gMSA pour Windows Server 2016 et Windows 10, versions 1709 et 1803

Si vous exécutez Windows Server 2016, version 1709 ou 1803, le nom d’hôte de votre conteneur doit correspondre au nom de votre compte SAM gMSA.

Lorsque le nom d’hôte ne correspond pas au nom gMSA, les demandes d’authentification NTLM entrantes et la traduction nom/ID de sécurité (utilisée par de nombreuses bibliothèques, comme le fournisseur de rôles d’appartenance ASP.NET) échoueront. Kerberos continuera de fonctionner normalement, même si le nom d’hôte et le nom gMSA ne correspondent pas.

Cette limitation a été résolue dans Windows Server 2019, où le conteneur utilisera désormais son nom gMSA sur le réseau quel que soit le nom d’hôte attribué.

#### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>L’utilisation d’un gMSA avec plusieurs conteneurs entraîne simultanément des échecs intermittents sur Windows Server 2016 et Windows 10, versions 1709 et 1803

Dans la mesure où tous les conteneurs doivent utiliser le même nom d’hôte, un deuxième problème affecte les versions de Windows antérieures à Windows Server 2019 et Windows 10, version 1809. Lorsque plusieurs conteneurs possèdent la même identité et le même nom d’hôte, une condition de concurrence critique risque de se produire lorsque deux conteneurs parlent simultanément au même contrôleur de domaine. Lorsqu’un autre conteneur parle au même contrôleur de domaine, il annule la communication avec les conteneurs antérieurs utilisant la même identité. Cela peut entraîner des échecs d’authentification intermittente et peut parfois être observé en tant qu’échec d' `nltest /sc_verify:contoso.com` approbation lors de l’exécution à l’intérieur du conteneur.

Nous avons modifié le comportement dans Windows Server 2019 pour séparer l’identité du conteneur du nom de l’ordinateur, ce qui permet à plusieurs conteneurs d’utiliser le même gMSA simultanément.

#### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Vous ne pouvez pas utiliser gMSAs avec des conteneurs isolés Hyper-V sur Windows 10 versions 1703, 1709 et 1803

L’initialisation d’un conteneur peut se bloquer ou échouer lorsque vous essayez d’utiliser un gMSA avec un conteneur Hyper-V isolé sur Windows 10 et Windows Server versions 1703, 1709 et 1803.

Ce bogue a été résolu dans Windows Server 2019 et Windows 10, version 1809. Vous pouvez également exécuter des conteneurs isolés Hyper-V avec gMSAs sur Windows Server 2016 et Windows 10, version 1607.

### <a name="general-troubleshooting-guidance"></a>Conseils de dépannage généraux

Si vous rencontrez des erreurs lors de l’exécution d’un conteneur avec un gMSA, les instructions suivantes peuvent vous aider à identifier la cause initiale.

#### <a name="make-sure-the-host-can-use-the-gmsa"></a>Vérifier que l’hôte peut utiliser le gMSA

1. Vérifiez que l’hôte est joint au domaine et qu’il peut accéder au contrôleur de domaine.
2. Installez les outils AD PowerShell de RSAT et exécutez [test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) pour voir si l’ordinateur a accès à la récupération de gMSA. Si l’applet de **** passe renvoie false, cela signifie que l’ordinateur n’a pas accès au mot de passe gMSA.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. Si **test-ADServiceAccount** renvoie la **valeur false**, assurez-vous que l’hôte appartient à un groupe de sécurité qui peut accéder au mot de passe gMSA.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. Si votre hôte appartient à un groupe de sécurité autorisé à récupérer le mot de passe gMSA mais qu’il ne parvient toujours pas à **tester-ADServiceAccount**, vous devrez peut-être redémarrer l’ordinateur pour obtenir un nouveau ticket reflétant son appartenance au groupe actuel.

#### <a name="check-the-credential-spec-file"></a>Vérifier le fichier de spécifications d’informations d’identification

1. Exécutez **Get-CredentialSpec** à partir du [module CredentialSpec PowerShell](https://aka.ms/credspec) pour rechercher toutes les spécifications d’informations d’identification sur l’ordinateur. Les spécifications d’informations d’identification doivent être stockées dans l’annuaire «CredentialSpecs» sous le répertoire racine de l’ancrage. Vous pouvez trouver le répertoire racine de l’ancrage en exécutant les **informations de dock-f "{{. DockerRootDir}} "**.
2. Ouvrez le fichier CredentialSpec et assurez-vous que les champs suivants sont correctement remplis:
    - **Sid**: le SID de votre compte gMSA
    - **MachineAccountName**: nom du compte Sam gMSA (pas d’inclusion du nom de domaine complet ou du signe dollar)
    - **DnsTreeName**: nom de domaine complet (FQDN) de votre forêt Active Directory
    - **DNSName**: FQDN du domaine auquel appartient gMSA
    - **NetBIOSName**: nom NetBIOS pour le domaine auquel appartient l’gMSA
    - **GroupManagedServiceAccounts/nom**: nom du compte Sam gMSA (n’incluez pas de nom de domaine complet ou signe dollar)
    - **GroupManagedServiceAccounts/Scope**: une entrée pour le nom de domaine complet et un autre pour le NetBIOS

    Votre entrée doit ressembler à l’exemple suivant d’une spécification complète des informations d’identification:

    ```json
    {
        "CmsPlugins": [
            "ActiveDirectory"
        ],
        "DomainJoinConfig": {
            "Sid": "S-1-5-21-702590844-1001920913-2680819671",
            "MachineAccountName": "webapp01",
            "Guid": "56d9b66c-d746-4f87-bd26-26760cfdca2e",
            "DnsTreeName": "contoso.com",
            "DnsName": "contoso.com",
            "NetBiosName": "CONTOSO"
        },
        "ActiveDirectoryConfig": {
            "GroupManagedServiceAccounts": [
                {
                    "Name": "webapp01",
                    "Scope": "contoso.com"
                },
                {
                    "Name": "webapp01",
                    "Scope": "CONTOSO"
                }
            ]
        }
    }
    ```

3. Vérifiez que le chemin d’accès au fichier de spécifications d’informations d’identification est correct pour votre solution d’orchestration. Si vous utilisez l’outil de connexion, assurez-vous que la `--security-opt="credentialspec=file://NAME.json"`commande exécuter du conteneur inclut, où «nom. JSON» est remplacé par le nom output par **Get-CredentialSpec**. Le nom est un nom de fichier plat, par rapport au dossier CredentialSpecs dans le répertoire racine de l’ancrage.

#### <a name="check-the-firewall-configuration"></a>Vérification de la configuration du pare-feu

Si vous utilisez une stratégie de pare-feu stricte sur le conteneur ou le réseau hôte, il est possible que les connexions requises du contrôleur de domaine Active Directory ou du serveur DNS soient bloquées.

| Protocole et port | Objectif |
|-------------------|---------|
| TCP et UDP 53 | DNS |
| TCP et UDP 88 | Kerberos |
| TCP 139 | Netlogon |
| TCP et UDP 389 | LDAP |
| TCP 636 | SSL LDAP |

Il peut être nécessaire d’autoriser l’accès à des ports supplémentaires en fonction du type de trafic envoyé par votre conteneur à un contrôleur de domaine.
Pour obtenir la liste complète des ports utilisés par Active Directory [, voir Active Directory et la configuration requise](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers) pour les ports de services de domaine Active Directory.

#### <a name="check-the-container"></a>Vérifier le conteneur

1. Si vous utilisez une version de Windows antérieure à Windows Server 2019 ou Windows 10, version 1809, le nom d’hôte de votre conteneur doit correspondre au nom gMSA. Assurez `--hostname` -vous que le paramètre correspond au nom court gMSA (sans composant de domaine; par exemple, «webapp01» au lieu de «webapp01.contoso.com»).

2. Vérifiez la configuration du réseau de conteneurs pour vérifier que le conteneur peut résoudre et accéder à un contrôleur de domaine pour le domaine gMSA. Les serveurs DNS incorrectement configurés dans le conteneur sont un problème d’identité courant.

3. Vérifiez si le conteneur dispose d’une connexion valide au domaine en exécutant l’applet de commande suivante dans le conteneur `docker exec` (à l’aide de ou un équivalent):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    La vérification de confiance doit `NERR_SUCCESS` retourner si le gMSA est disponible et que la connectivité réseau permet au conteneur de communiquer avec le domaine. En cas d’échec, vérifiez la configuration réseau de l’hôte et du conteneur. Les deux doivent pouvoir communiquer avec le contrôleur de domaine.

4. Assurez-vous que votre application est [configurée pour utiliser gMSA](#configure-your-application-to-use-the-gmsa). Le compte d’utilisateur à l’intérieur du conteneur ne change pas lorsque vous utilisez un gMSA. Au lieu de cela, le compte système utilise le gMSA lorsqu’il parle d’autres ressources réseau. Cela signifie que votre application doit s’exécuter en tant que service réseau ou système local pour tirer parti de l’identité gMSA.

    > [!TIP]
    > Si vous exécutez `whoami` ou utilisez un autre outil pour identifier le contexte de l’utilisateur actuel dans le conteneur, vous ne verrez pas le nom gMSA. En effet, vous vous connectez toujours au conteneur en tant qu’utilisateur local au lieu d’une identité de domaine. Le gMSA est utilisé par le compte d’ordinateur chaque fois qu’il parle de ressources réseau, ce qui explique pourquoi votre application doit s’exécuter en tant que service réseau ou système local.

5. Enfin, si votre conteneur semble être correctement configuré, mais que des utilisateurs ou d’autres services ne peuvent pas s’authentifier automatiquement auprès de votre application conteneur, vérifiez les noms d’utilisateur de votre compte gMSA. Les clients peuvent trouver le compte gMSA à l’aide du nom sur lequel ils accèdent à votre application. Cela peut signifier que vous aurez besoin de `host` SPN supplémentaires pour votre gMSA si, par exemple, les clients se connectent à votre application par le biais d’un équilibreur de charge ou d’un autre nom DNS.

## <a name="additional-resources"></a>Ressources supplémentaires

- [vue d’ensemble du compte de service géré par groupe](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)

---
title: Créer des service administrés pour les conteneurs Windows
description: Création de comptes de service administrés de groupe (service administrés) pour les conteneurs Windows.
keywords: ancrage, conteneurs, Active Directory, GMSA, compte de service administré de groupe, comptes de service administrés de groupe
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 9ed9029e534d56bfe1830281d0bfd3ddde0cee9e
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910249"
---
# <a name="create-gmsas-for-windows-containers"></a>Créer des service administrés pour les conteneurs Windows

Les réseaux Windows utilisent couramment Active Directory (AD) pour faciliter l’authentification et l’autorisation entre les utilisateurs, les ordinateurs et d’autres ressources réseau. Les développeurs d’applications d’entreprise conçoivent souvent leurs applications pour qu’elles soient intégrées à AD et s’exécutent sur des serveurs joints à un domaine pour tirer parti de l’authentification Windows intégrée, ce qui permet aux utilisateurs et aux autres services de se connecter de manière automatique et transparente à l’application avec leurs identités.

Bien que les conteneurs Windows ne puissent pas être joints à un domaine, ils peuvent toujours utiliser des identités de domaine Active Directory pour prendre en charge différents scénarios d’authentification.

Pour ce faire, vous pouvez configurer un conteneur Windows pour qu’il s’exécute avec un [compte de service administré de groupe](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA), qui est un type spécial de compte de service introduit dans Windows Server 2012 conçu pour permettre à plusieurs ordinateurs de partager une identité sans avoir besoin de connaître son mot de passe.

Quand vous exécutez un conteneur avec un gMSA, l’hôte de conteneur récupère le mot de passe gMSA à partir d’un contrôleur de domaine Active Directory et le transmet à l’instance de conteneur. Le conteneur utilisera les informations d’identification gMSA chaque fois que son compte d’ordinateur (système) a besoin d’accéder aux ressources réseau.

Cet article explique comment commencer à utiliser Active Directory comptes de service administrés de groupe avec des conteneurs Windows.

## <a name="prerequisites"></a>Conditions préalables

Pour exécuter un conteneur Windows avec un compte de service administré de groupe, vous aurez besoin des éléments suivants :

- Un domaine Active Directory avec au moins un contrôleur de domaine exécutant Windows Server 2012 ou une version ultérieure. Aucune exigence de niveau fonctionnel de forêt ou de domaine n’est requise pour utiliser service administrés, mais les mots de passe gMSA peuvent être distribués uniquement par les contrôleurs de domaine exécutant Windows Server 2012 ou une version ultérieure. Pour plus d’informations, consultez [Active Directory Requirements for service administrés](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req).
- Autorisation de créer un compte gMSA. Pour créer un compte gMSA, vous devez être un administrateur de domaine ou utiliser un compte pour lequel l’autorisation *créer des objets MSDS-GroupManagedServiceAccount* a été déléguée.
- Accès à Internet pour télécharger le module CredentialSpec PowerShell. Si vous travaillez dans un environnement déconnecté, vous pouvez [enregistrer le module](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1) sur un ordinateur disposant d’un accès Internet et le copier sur votre ordinateur de développement ou hôte de conteneur.

## <a name="one-time-preparation-of-active-directory"></a>Préparation unique d’Active Directory

Si vous n’avez pas encore créé de gMSA dans votre domaine, vous devez générer la clé racine du service de distribution de clés (KDS). Le KDS est chargé de créer, de faire pivoter et de libérer le mot de passe gMSA pour les hôtes autorisés. Lorsqu’un hôte de conteneur doit utiliser le gMSA pour exécuter un conteneur, il contacte l’KDS pour récupérer le mot de passe actuel.

Pour vérifier si la clé racine KDS a déjà été créée, exécutez l’applet de commande PowerShell suivante en tant qu’administrateur de domaine sur un contrôleur de domaine ou membre de domaine sur lequel les outils AD PowerShell sont installés :

```powershell
Get-KdsRootKey
```

Si la commande retourne un ID de clé, vous êtes ensemble et pouvez passer directement à la section [créer un compte de service administré de groupe](#create-a-group-managed-service-account) . Sinon, passez à la création de la clé racine KDS.

Dans un environnement de production ou de test avec plusieurs contrôleurs de domaine, exécutez l’applet de commande suivante dans PowerShell en tant qu’administrateur de domaine pour créer la clé racine KDS.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

Bien que la commande implique que la clé sera appliquée immédiatement, vous devrez attendre 10 heures avant que la clé racine KDS soit répliquée et disponible pour une utilisation sur tous les contrôleurs de domaine.

Si vous n’avez qu’un seul contrôleur de domaine dans votre domaine, vous pouvez accélérer le processus en définissant la clé pour qu’elle soit effective il y a 10 heures.

>[!IMPORTANT]
>N’utilisez pas cette technique dans un environnement de production.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>Créer un compte de service administré de groupe

Chaque conteneur qui utilise l’authentification Windows intégrée a besoin d’au moins un gMSA. Le gMSA principal est utilisé chaque fois que les applications qui s’exécutent en tant que service système ou réseau accèdent aux ressources du réseau. Le nom du gMSA deviendra le nom du conteneur sur le réseau, quel que soit le nom d’hôte affecté au conteneur. Les conteneurs peuvent également être configurés avec des service administrés supplémentaires, au cas où vous souhaiteriez exécuter un service ou une application dans le conteneur sous la forme d’une identité différente du compte d’ordinateur de conteneur.

Lorsque vous créez un gMSA, vous créez également une identité partagée qui peut être utilisée simultanément sur plusieurs ordinateurs différents. L’accès au mot de passe gMSA est protégé par une liste de Access Control Active Directory. Nous vous recommandons de créer un groupe de sécurité pour chaque compte gMSA et d’ajouter les hôtes de conteneur appropriés au groupe de sécurité pour limiter l’accès au mot de passe.

Enfin, étant donné que les conteneurs n’inscrivent pas automatiquement les noms de principal du service (SPN), vous devrez créer manuellement au moins un SPN d’hôte pour votre compte gMSA.

En règle générale, l’hôte ou le SPN http est inscrit avec le même nom que le compte gMSA, mais vous devrez peut-être utiliser un nom de service différent si les clients accèdent à l’application en conteneur derrière un équilibreur de charge ou un nom DNS différent du nom de gMSA.

Par exemple, si le compte gMSA est nommé « WebApp01 », mais que vos utilisateurs accèdent au site à `mysite.contoso.com`, vous devez inscrire un SPN `http/mysite.contoso.com` sur le compte gMSA.

Certaines applications peuvent nécessiter des SPN supplémentaires pour leurs protocoles uniques. Par exemple, SQL Server requiert le nom de principal du service `MSSQLSvc/hostname`.

Le tableau suivant répertorie les attributs requis pour la création d’un gMSA.

|propriété gMSA | Valeur obligatoire | Exemple |
|--------------|----------------|--------|
|Nom | N’importe quel nom de compte valide. | `WebApp01` |
|DnsHostName | Nom de domaine ajouté au nom du compte. | `WebApp01.contoso.com` |
|ServicePrincipalNames | Définissez au moins le nom principal de service de l’hôte, puis ajoutez d’autres protocoles si nécessaire. | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | Groupe de sécurité contenant vos hôtes de conteneur. | `WebApp01Hosts` |

Une fois que vous avez choisi le nom de votre gMSA, exécutez les applets de commande suivantes dans PowerShell pour créer le groupe de sécurité et gMSA.

> [!TIP]
> Vous devez utiliser un compte qui appartient au groupe de sécurité **Admins du domaine** , ou l’autorisation **créer des objets MSDS-GroupManagedServiceAccount** pour exécuter les commandes suivantes vous a été déléguée.
> L’applet de commande [New-ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) fait partie des outils ad PowerShell de [Outils d’administration de serveur distant](https://aka.ms/rsat).

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

Nous vous recommandons de créer des comptes gMSA distincts pour vos environnements de développement, de test et de production.

## <a name="prepare-your-container-host"></a>Préparer votre hôte de conteneur

Chaque hôte de conteneur qui exécutera un conteneur Windows avec un gMSA doit être joint à un domaine et avoir accès pour récupérer le mot de passe gMSA.

1. Joignez votre ordinateur à votre domaine Active Directory.
2. Assurez-vous que votre hôte appartient au groupe de sécurité contrôlant l’accès au mot de passe gMSA.
3. Redémarrez l’ordinateur pour qu’il récupère sa nouvelle appartenance au groupe.
4. Configurez le [Bureau de l’amarrage pour Windows 10](https://docs.docker.com/docker-for-windows/install/) ou [docker pour Windows Server](https://docs.docker.com/install/windows/docker-ee/).
5. Recommandations Vérifiez que l’ordinateur hôte peut utiliser le compte gMSA en exécutant [test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount). Si la commande retourne la **valeur false**, suivez les [instructions de résolution des problèmes](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa).

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>Créer une spécification d’informations d’identification

Un fichier de spécification d’informations d’identification est un document JSON qui contient les métadonnées relatives au (x) compte (s) gMSA que vous souhaitez qu’un conteneur utilise. En gardant la configuration d’identité séparée de l’image de conteneur, vous pouvez changer le gMSA utilisé par le conteneur en échangeant simplement le fichier de spécification d’informations d’identification, aucune modification de code n’est nécessaire.

Le fichier de spécification des informations d’identification est créé à l’aide du [module PowerShell CredentialSpec](https://aka.ms/credspec) sur un hôte de conteneur joint à un domaine.
Une fois que vous avez créé le fichier, vous pouvez le copier vers d’autres hôtes de conteneur ou votre Orchestrator de conteneur.
Le fichier de spécification d’informations d’identification ne contient pas de secrets, tels que le mot de passe gMSA, car l’hôte de conteneur récupère le gMSA pour le compte du conteneur.

L’ancrage s’attend à trouver le fichier de spécification des informations d’identification sous le répertoire **CredentialSpecs** dans le répertoire des données de l’arrimeur. Dans une installation par défaut, vous trouverez ce dossier sur `C:\ProgramData\Docker\CredentialSpecs`.

Pour créer un fichier de spécification d’informations d’identification sur votre hôte de conteneur :

1. Installer les outils PowerShell pour les outils d’installation à REINSTALLATION Active Directory
    - Pour Windows Server, exécutez **install-WINDOWSFEATURE RSAT-ad-PowerShell**.
    - Pour Windows 10, version 1809 ou ultérieure, exécutez **Add-WindowsCapability-Online-Name’RSAT. ActiveDirectory. DS-LDS. Tools ~ ~ ~ ~ 0.0.1.0 '** .
    - Pour les versions antérieures de Windows 10, consultez <https://aka.ms/rsat>.
2. Exécutez l’applet de commande suivante pour installer la dernière version du [module PowerShell CredentialSpec](https://aka.ms/credspec):

    ```powershell
    Install-Module CredentialSpec
    ```

    Si vous n’avez pas accès à Internet sur votre hôte de conteneur, exécutez `Save-Module CredentialSpec` sur une machine connectée à Internet et copiez le dossier du module vers `C:\Program Files\WindowsPowerShell\Modules` ou un autre emplacement dans `$env:PSModulePath` sur l’hôte de conteneur.

3. Exécutez l’applet de commande suivante pour créer le fichier de spécification d’informations d’identification :

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    Par défaut, l’applet de commande crée une spécification cred en utilisant le nom gMSA fourni comme compte d’ordinateur pour le conteneur. Le fichier est enregistré dans le répertoire CredentialSpecs de l’ancrage en utilisant le domaine gMSA et le nom de compte pour le nom de fichier.

    Vous pouvez créer des spécifications d’informations d’identification qui incluent des comptes gMSA supplémentaires si vous exécutez un service ou un processus en tant que gMSA secondaire dans le conteneur. Pour ce faire, utilisez le paramètre `-AdditionalAccounts` :

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    Pour obtenir la liste complète des paramètres pris en charge, exécutez `Get-Help New-CredentialSpec`.

4. Vous pouvez afficher une liste de toutes les spécifications d’informations d’identification et leur chemin d’accès complet avec l’applet de commande suivante :

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez configuré votre compte gMSA, vous pouvez l’utiliser pour :

- [Configurer des applications](gmsa-configure-app.md)
- [Exécuter des conteneurs](gmsa-run-container.md)
- [Orchestrer des conteneurs](gmsa-orchestrate-containers.md)

Si vous rencontrez des problèmes lors de l’installation, consultez notre [Guide de résolution](gmsa-troubleshooting.md) des problèmes pour obtenir des solutions possibles.

## <a name="additional-resources"></a>Ressources supplémentaires

- Pour en savoir plus sur service administrés, consultez la [vue d’ensemble des comptes de service administrés de groupe](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).
- Pour une démonstration vidéo, regardez notre [démonstration enregistrée](https://youtu.be/cZHPz80I-3s?t=2672) de l’allumage 2016.

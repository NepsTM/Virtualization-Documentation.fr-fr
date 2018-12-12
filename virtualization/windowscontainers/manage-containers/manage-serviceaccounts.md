---
title: Comptes de service ActiveDirectory pour conteneurs Windows
description: Comptes de service ActiveDirectory pour conteneurs Windows
keywords: docker, conteneurs, activedirectory
author: PatrickLang
ms.date: 11/04/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: d92d14fd10e07e159ff2023b4dd6ade8b11ca2e5
ms.sourcegitcommit: 4090d158dd3573ea90799de5b014c131a206b000
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2018
ms.locfileid: "6121589"
---
# <a name="active-directory-service-accounts-for-windows-containers"></a>Comptes de service ActiveDirectory pour conteneurs Windows

Les utilisateurs et les autres services devront peut-être établir des connexions authentifiées à vos applications et services afin que vous puissiez préserver la sécurité des données et empêcher toute utilisation non autorisée. Les domaines WindowsActiveDirectory (AD) prennent en charge en mode natif l’authentification par mot de passe et par certificat. Si vous générez votre application (ou service) sur un hôte appartenant à un domaine Windows, elle utilise l’identité de l’hôte par défaut si elle s’exécute en tant que système local ou service réseau. Dans le cas contraire, vous pouvez préférer configurer un autre compte ActiveDirectory pour l’authentification.

Même si les conteneurs Windows ne peuvent pas être joints à un domaine, ils peuvent tirer parti des identités de domaine ActiveDirectory semblables à celles qui se présentent quand un appareil est joint à un domaine. Avec les contrôleurs de domaine Windows Server2012, nous avons introduit un nouveau compte appelé compte de service administré de groupe (gMSA), qui a été conçu pour être partagé par les services. En utilisant des comptesgMSA, les conteneurs Windows proprement dits et les services qu’ils hébergent peuvent être configurés pour utiliser un comptegMSA spécifique en tant qu’identité de leur domaine. Tout service exécuté en tant que système local ou service réseau utilise l’identité du conteneur Windows comme il utilise actuellement l’identité de l’hôte joint à un domaine. Aucun mot de passe ni clé privée de certificat stockés dans l’image de conteneur ne peuvent être exposés par inadvertance. En outre, le conteneur peut être redéployé dans les environnements de développement, test et production sans être régénéré pour modifier les mots de passe ou les certificats stockés. 


# <a name="glossary--references"></a>Glossaire et références
- [ActiveDirectory](http://social.technet.microsoft.com/wiki/contents/articles/1026.active-directory-services-overview.aspx) est un service utilisé pour la découverte, la recherche et la réplication des informations de compte d’utilisateur, d’ordinateur et de service sur Windows. 
  - Les services [Active Directory Domain Services](https://technet.microsoft.com/en-us/library/dd448614.aspx) fournissent des domaines Windows ActiveDirectory qui permettent d’authentifier les utilisateurs et les ordinateurs. 
  - Les appareils sont _joints à un domaine_ lorsqu’ils sont membres du domaine ActiveDirectory. L’état «joint à un domaine» est un état d’appareil qui fournit non seulement une identité d’ordinateur de domaine à l’appareil, mais met également en avant différents services joints au domaine.
  - Les [comptes de service administrés de groupe](https://technet.microsoft.com/en-us/library/jj128431(v=ws.11).aspx), souvent abrégés sous la forme gMSA, sont un type de compte ActiveDirectory qui facilite la sécurisation des services à l’aide d’ActiveDirectory sans partager de mot de passe. Plusieurs ordinateurs ou conteneurs partagent au besoin un même compte gMSA pour authentifier les connexions entre les services.
- Module PowerShell _CredentialSpec_: ce module permet de configurer des comptes de service administrés de groupe à utiliser avec des conteneurs. Le module de script et des exemples d’étape sont disponibles dans [windows-server-container-tools](https://github.com/Microsoft/Virtualization-Documentation/tree/live/windows-server-container-tools), voir ServiceAccount

# <a name="how-it-works"></a>Fonctionnement

Pour le moment, les comptes de service administrés de groupe sont souvent utilisés pour sécuriser les connexions établies entre un ordinateur ou un service et un autre. Étapes générales pour utiliser un compte gMSA:

1. Créer un compte gMSA
2. Configurer le service à exécuter en tant que compte gMSA
3. Permettre à l’hôte joint à un domaine, exécutant le service, d’accéder aux secrets du compte gMSA dans ActiveDirectory
4. Autoriser l’accès au compte gMSA sur l’autre service, par exemple une base de données ou des partages de fichiers

Au démarrage du service, l’hôte joint à un domaine obtient automatiquement les secrets gMSA auprès d’ActiveDirectory et exécute le service à l’aide de ce compte. Comme ce service est exécuté en tant que compte gMSA, il peut accéder aux ressources pour lesquelles le compte gMSA dispose de droits d’accès.

Les conteneurs Windows suivent un processus similaire:

1. Créez un compte gMSA. Par défaut, un administrateur de domaine ou un opérateur de compte doit effectuer cette opération. Sinon, ils peuvent déléguer les privilèges permettant de créer et gérer des comptes gMSA aux administrateurs qui gèrent les services qui les utilisent. Il vous faut au moins un serveur WindowsServer2012 ou une version ultérieure d’un contrôleur de domaine dans votre domaine, mais il n’est pas nécessaire d’utiliser un niveau fonctionnel de domaine spécifique. Voir [Prise en main des comptes de service administrés](https://technet.microsoft.com/en-us/library/jj128431(v=ws.11).aspx)
2. Octroyer à l’hôte de conteneur joint à un domaine un accès au compte gMSA
3. Autoriser l’accès au compte gMSA sur l’autre service, par exemple une base de données ou des partages de fichiers
4. Utiliser le module PowerShell CredentialSpec de [windows-server-container-tools](https://github.com/Microsoft/Virtualization-Documentation/tree/live/windows-server-container-tools) pour stocker les paramètres nécessaires pour utiliser le compte gMSA
5. Démarrer le conteneur avec une option supplémentaire `--security-opt "credentialspec=..."`

> [!NOTE]
> Vous devrez peut-être autoriser la traduction de noms/SID anonymes sur l’hôte de conteneur comme décrit [ici](https://docs.microsoft.com/en-us/windows/device-security/security-policy-settings/network-access-allow-anonymous-sidname-translation), car autrement, vous pourriez obtenir des erreurs indiquant que des comptes ne peuvent être traduits en SID.

Toutefois, avant d’explorer la nécessité d'autoriser la traduction de noms/SID anonymes, assurez-vous que les mesures suivantes ont été prises:

1. Le nom du compte gMSA doit correspondre au nom du service (par ex. «myapp»).
2. Inclure l’argument -h pour spécifier le nom d’hôte que le conteneur doit utiliser lors de son lancement. 
```
docker run --security-opt "credentialspec=file://myapp.json" -d -p 80:80 -h myapp.mydomain.local <imagename>
```
3. Le nom principal de service (SPN) utilisé lors de la création du compte gMSA doit correspondre à l’argument -h utilisé lors de l'exécution du conteneur. Si vous n’avez pas ajouté de SPN pour le compte gMSA lors de la création, vous pouvez en ajouter par la suite aux propriétés du compte.

Lorsque le conteneur est lancé, les services installés qui sont exécutés en tant que système local ou service réseau s’affichent pour s’exécuter en tant que compte gMSA. Ce fonctionnement est semblable à celui des comptes sur les hôtes joints à un domaine, sauf qu’un compte gMSA est utilisé à la place d’un compte d’ordinateur. 

![Diagramme - Comptes de service](media/serviceaccount_diagram.png)


# <a name="example-uses"></a>Exemples d’utilisation


## <a name="sql-connection-strings"></a>Chaînes de connexion SQL
Lorsqu’un service est exécuté en tant que système local ou service réseau dans un conteneur, il peut utiliser l’authentification intégrée de Windows pour se connecter à un serveur MicrosoftSQLServer.

Exemple:

```
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

Sur le serveur MicrosoftSQLServer, créez une connexion en utilisant le nom du domaine et du compte gMSA, suivis d’un symbole$. Une fois la connexion créée, elle peut être ajoutée à un utilisateur dans une base de données et bénéficier des autorisations d’accès appropriées.

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

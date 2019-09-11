---
title: Résoudre les problèmes de gMSAs pour les conteneurs Windows
description: Comment résoudre les problèmes liés aux comptes de service géré par le groupe (gMSAs) pour les conteneurs Windows.
keywords: dockeur, conteneurs, Active Directory, GMSA, compte de service géré de groupe, comptes de service géré de groupe, résolution des problèmes, résolution des problèmes
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 00a0d9b1367da55b7669fc26a3eca303272967ab
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079721"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Résoudre les problèmes de gMSAs pour les conteneurs Windows

## <a name="known-issues"></a>Problèmes connus

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Le nom d’hôte du conteneur doit correspondre au nom gMSA pour Windows Server 2016 et Windows 10, versions 1709 et 1803

Si vous exécutez Windows Server 2016, version 1709 ou 1803, le nom d’hôte de votre conteneur doit correspondre au nom de votre compte SAM gMSA.

Lorsque le nom d’hôte ne correspond pas au nom gMSA, les demandes d’authentification NTLM entrantes et la traduction nom/ID de sécurité (utilisée par de nombreuses bibliothèques, comme le fournisseur de rôles d’appartenance ASP.NET) échoueront. Kerberos continuera de fonctionner normalement, même si le nom d’hôte et le nom gMSA ne correspondent pas.

Cette limitation a été résolue dans Windows Server 2019, où le conteneur utilisera désormais son nom gMSA sur le réseau quel que soit le nom d’hôte attribué.

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>L’utilisation d’un gMSA avec plusieurs conteneurs entraîne simultanément des échecs intermittents sur Windows Server 2016 et Windows 10, versions 1709 et 1803

Dans la mesure où tous les conteneurs doivent utiliser le même nom d’hôte, un deuxième problème affecte les versions de Windows antérieures à Windows Server 2019 et Windows 10, version 1809. Lorsque plusieurs conteneurs possèdent la même identité et le même nom d’hôte, une condition de concurrence critique risque de se produire lorsque deux conteneurs parlent simultanément au même contrôleur de domaine. Lorsqu’un autre conteneur parle au même contrôleur de domaine, il annule la communication avec les conteneurs antérieurs utilisant la même identité. Cela peut entraîner des échecs d’authentification intermittente et peut parfois être observé en tant qu’échec d' `nltest /sc_verify:contoso.com` approbation lors de l’exécution à l’intérieur du conteneur.

Nous avons modifié le comportement dans Windows Server 2019 pour séparer l’identité du conteneur du nom de l’ordinateur, ce qui permet à plusieurs conteneurs d’utiliser le même gMSA simultanément.

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Vous ne pouvez pas utiliser gMSAs avec des conteneurs isolés Hyper-V sur Windows 10 versions 1703, 1709 et 1803

L’initialisation d’un conteneur peut se bloquer ou échouer lorsque vous essayez d’utiliser un gMSA avec un conteneur Hyper-V isolé sur Windows 10 et Windows Server versions 1703, 1709 et 1803.

Ce bogue a été résolu dans Windows Server 2019 et Windows 10, version 1809. Vous pouvez également exécuter des conteneurs isolés Hyper-V avec gMSAs sur Windows Server 2016 et Windows 10, version 1607.

## <a name="general-troubleshooting-guidance"></a>Conseils de dépannage généraux

Si vous rencontrez des erreurs lors de l’exécution d’un conteneur avec un gMSA, les instructions suivantes peuvent vous aider à identifier la cause initiale.

### <a name="make-sure-the-host-can-use-the-gmsa"></a>Vérifier que l’hôte peut utiliser le gMSA

1. Vérifiez que l’hôte est joint au domaine et qu’il peut accéder au contrôleur de domaine.
2. Installez les outils AD PowerShell de RSAT et exécutez [test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) pour voir si l’ordinateur a accès à la récupération de gMSA. Si l’applet de passe renvoie **false**, cela signifie que l’ordinateur n’a pas accès au mot de passe gMSA.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
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

### <a name="check-the-firewall-configuration"></a>Vérification de la configuration du pare-feu

Si vous utilisez une stratégie de pare-feu stricte sur le conteneur ou le réseau hôte, il est possible que les connexions requises du contrôleur de domaine Active Directory ou du serveur DNS soient bloquées.

| Protocole et port | Objectif |
|-------------------|---------|
| TCP et UDP 53 | DNS |
| TCP et UDP 88 | Kerberos |
| TCP 139 | Netlogon |
| TCP et UDP 389 | LDAP |
| TCP 636 | SSL LDAP |

Il peut être nécessaire d’autoriser l’accès à des ports supplémentaires en fonction du type de trafic envoyé par votre conteneur à un contrôleur de domaine.
Pour obtenir la liste complète des ports utilisés par Active Directory [, voir Active Directory et la configuration requise](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers) pour les ports de services de domaine Active Directory.

### <a name="check-the-container"></a>Vérifier le conteneur

1. Si vous utilisez une version de Windows antérieure à Windows Server 2019 ou Windows 10, version 1809, le nom d’hôte de votre conteneur doit correspondre au nom gMSA. Assurez `--hostname` -vous que le paramètre correspond au nom court gMSA (sans composant de domaine; par exemple, «webapp01» au lieu de «webapp01.contoso.com»).

2. Vérifiez la configuration du réseau de conteneurs pour vérifier que le conteneur peut résoudre et accéder à un contrôleur de domaine pour le domaine gMSA. Les serveurs DNS incorrectement configurés dans le conteneur sont un problème d’identité courant.

3. Vérifiez si le conteneur dispose d’une connexion valide au domaine en exécutant l’applet de commande suivante dans le conteneur `docker exec` (à l’aide de ou un équivalent):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    La vérification de confiance doit `NERR_SUCCESS` retourner si le gMSA est disponible et que la connectivité réseau permet au conteneur de communiquer avec le domaine. En cas d’échec, vérifiez la configuration réseau de l’hôte et du conteneur. Les deux doivent pouvoir communiquer avec le contrôleur de domaine.

4. Assurez-vous que votre application est [configurée pour utiliser gMSA](gmsa-configure-app.md). Le compte d’utilisateur à l’intérieur du conteneur ne change pas lorsque vous utilisez un gMSA. Au lieu de cela, le compte système utilise le gMSA lorsqu’il parle d’autres ressources réseau. Cela signifie que votre application doit s’exécuter en tant que service réseau ou système local pour tirer parti de l’identité gMSA.

    > [!TIP]
    > Si vous exécutez `whoami` ou utilisez un autre outil pour identifier le contexte de l’utilisateur actuel dans le conteneur, vous ne verrez pas le nom gMSA. En effet, vous vous connectez toujours au conteneur en tant qu’utilisateur local au lieu d’une identité de domaine. Le gMSA est utilisé par le compte d’ordinateur chaque fois qu’il parle de ressources réseau, ce qui explique pourquoi votre application doit s’exécuter en tant que service réseau ou système local.

5. Enfin, si votre conteneur semble être correctement configuré, mais que des utilisateurs ou d’autres services ne peuvent pas s’authentifier automatiquement auprès de votre application conteneur, vérifiez les noms d’utilisateur de votre compte gMSA. Les clients peuvent trouver le compte gMSA à l’aide du nom sur lequel ils accèdent à votre application. Cela peut signifier que vous aurez besoin de `host` SPN supplémentaires pour votre gMSA si, par exemple, les clients se connectent à votre application par le biais d’un équilibreur de charge ou d’un autre nom DNS.

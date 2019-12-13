---
title: Résoudre les problèmes de service administrés pour les conteneurs Windows
description: Comment résoudre les problèmes liés aux comptes de service administrés de groupe (service administrés) pour les conteneurs Windows.
keywords: docker, conteneurs, Active Directory, GMSA, compte de service administré de groupe, comptes de service administrés de groupe, dépannage, résolution des problèmes
author: rpsqrd
ms.date: 10/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 89f255e307c2a48fd743d5abd1a49bba7703aaf3
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910239"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Résoudre les problèmes de service administrés pour les conteneurs Windows

## <a name="known-issues"></a>Problèmes connus

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Le nom d’hôte du conteneur doit correspondre au nom gMSA pour Windows Server 2016 et Windows 10, versions 1709 et 1803

Si vous utilisez Windows Server 2016, version 1709 ou 1803, le nom d’hôte de votre conteneur doit correspondre au nom de votre compte SAM gMSA.

Lorsque le nom d’hôte ne correspond pas au nom gMSA, les demandes d’authentification NTLM entrantes et la traduction de noms/SID (utilisée par de nombreuses bibliothèques, comme le fournisseur de rôle d’appartenance ASP.NET) échouent. Kerberos continue à fonctionner normalement, même si le nom d’hôte et le nom de gMSA ne correspondent pas.

Cette limitation a été corrigée dans Windows Server 2019, où le conteneur utilisera toujours son nom gMSA sur le réseau, quel que soit le nom d’hôte attribué.

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>L’utilisation simultanée d’un gMSA avec plusieurs conteneurs provoque des défaillances intermittentes sur Windows Server 2016 et Windows 10, versions 1709 et 1803

Étant donné que tous les conteneurs sont requis pour utiliser le même nom d’hôte, un deuxième problème affecte les versions de Windows antérieures à Windows Server 2019 et Windows 10, version 1809. Lorsque plusieurs conteneurs reçoivent la même identité et le même nom d’hôte, une condition de concurrence peut se produire lorsque deux conteneurs communiquent simultanément avec le même contrôleur de domaine. Lorsqu’un autre conteneur communique avec le même contrôleur de domaine, il annule la communication avec les conteneurs précédents à l’aide de la même identité. Cela peut entraîner des échecs d’authentification intermittents et peut parfois être observé comme un échec d’approbation lorsque vous exécutez `nltest /sc_verify:contoso.com` à l’intérieur du conteneur.

Nous avons modifié le comportement de Windows Server 2019 pour séparer l’identité du conteneur du nom de l’ordinateur, ce qui permet à plusieurs conteneurs d’utiliser le même gMSA simultanément.

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Vous ne pouvez pas utiliser service administrés avec des conteneurs isolés Hyper-V sur Windows 10 versions 1703, 1709 et 1803

L’initialisation du conteneur se bloque ou échoue quand vous essayez d’utiliser un gMSA avec un conteneur isolé Hyper-V sur Windows 10 et Windows Server versions 1703, 1709 et 1803.

Ce bogue a été corrigé dans Windows Server 2019 et Windows 10, version 1809. Vous pouvez également exécuter des conteneurs isolés Hyper-V avec service administrés sur Windows Server 2016 et Windows 10, version 1607.

## <a name="general-troubleshooting-guidance"></a>Instructions générales pour la résolution des problèmes

Si vous rencontrez des erreurs lors de l’exécution d’un conteneur avec un gMSA, les instructions suivantes peuvent vous aider à identifier la cause racine.

### <a name="make-sure-the-host-can-use-the-gmsa"></a>Assurez-vous que l’hôte peut utiliser le gMSA

1. Vérifiez que l’hôte est joint au domaine et qu’il peut atteindre le contrôleur de domaine.
2. Installez les outils AD PowerShell à partir de RSAT et exécutez [test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) pour voir si l’ordinateur a accès à la récupération du gMSA. Si l’applet de commande retourne la **valeur false**, l’ordinateur n’a pas accès au mot de passe gMSA.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. Si **test-ADServiceAccount** retourne la **valeur false**, vérifiez que l’hôte appartient à un groupe de sécurité qui peut accéder au mot de passe gMSA.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. Si votre hôte appartient à un groupe de sécurité autorisé à récupérer le mot de passe gMSA mais qu’il échoue encore **test-ADServiceAccount**, vous devrez peut-être redémarrer votre ordinateur pour obtenir un nouveau ticket reflétant ses appartenances à des groupes actuels.

#### <a name="check-the-credential-spec-file"></a>Vérifier le fichier de spécification des informations d’identification

1. Exécutez la **CredentialSpec** à partir du [module CredentialSpec PowerShell](https://aka.ms/credspec) pour rechercher toutes les spécifications d’informations d’identification sur l’ordinateur. Les spécifications des informations d’identification doivent être stockées dans le répertoire « CredentialSpecs » sous le répertoire racine de l’amarrage. Vous pouvez trouver le répertoire racine de l’arrimeur en exécutant le **dockr info-f "{{. DockerRootDir}}»** .
2. Ouvrez le fichier CredentialSpec et assurez-vous que les champs suivants sont remplis correctement :
    - **Sid**: le SID de votre compte gMSA
    - **MachineAccountName**: nom du compte Sam gMSA (ne pas inclure le nom de domaine complet ou le signe dollar)
    - **DnsTreeName**: nom de domaine complet de votre forêt Active Directory
    - **DNSName**: nom de domaine complet du domaine auquel appartient le gMSA
    - **NetBIOSName**: nom NetBIOS du domaine auquel appartient le gMSA
    - **GroupManagedServiceAccounts/Name**: nom du compte Sam gMSA (ne pas inclure le nom de domaine complet ou le signe dollar)
    - **GroupManagedServiceAccounts/étendue**: une entrée pour le nom de domaine complet du domaine et une autre pour le NetBIOS

    Votre entrée doit ressembler à l’exemple suivant d’une spécification complète des informations d’identification :

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

3. Vérifiez que le chemin d’accès au fichier de spécification des informations d’identification est correct pour votre solution d’orchestration. Si vous utilisez Dockr, assurez-vous que la commande conteneur Run comprend `--security-opt="credentialspec=file://NAME.json"`, où « NAME. JSON » est remplacé par le nom output par la commande **CredentialSpec**. Le nom est un nom de fichier plat, relatif au dossier CredentialSpecs sous le répertoire racine de l’ancrage.

### <a name="check-the-firewall-configuration"></a>Vérifier la configuration du pare-feu

Si vous utilisez une stratégie de pare-feu stricte sur le réseau de conteneurs ou d’ordinateurs hôtes, elle peut bloquer les connexions requises au contrôleur domaine Active Directory ou au serveur DNS.

| Protocole et port | Objectif |
|-------------------|---------|
| TCP et UDP 53 | DNS |
| TCP et UDP 88 | Kerberos |
| TCP 139 | Accès réseau (Netlogon) |
| TCP et UDP 389 | LDAP |
| TCP 636 | LDAP SSL |

Vous devrez peut-être autoriser l’accès à des ports supplémentaires en fonction du type de trafic que votre conteneur envoie à un contrôleur de domaine.
Pour obtenir la liste complète des ports utilisés par Active Directory [, consultez Active Directory et Active Directory Domain Services Port Requirements](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers) .

### <a name="check-the-container"></a>Vérifier le conteneur

1. Si vous exécutez une version de Windows antérieure à Windows Server 2019 ou Windows 10, version 1809, le nom d’hôte de votre conteneur doit correspondre au nom de gMSA. Assurez-vous que le paramètre `--hostname` correspond au nom abrégé gMSA (aucun composant de domaine, par exemple, « webapp01 » au lieu de « webapp01.contoso.com »).

2. Vérifiez la configuration de mise en réseau de conteneurs pour vérifier que le conteneur peut être résolu et accéder à un contrôleur de domaine pour le domaine du gMSA. Les serveurs DNS mal configurés dans le conteneur sont souvent à l’origine de problèmes d’identité.

3. Vérifiez si le conteneur a une connexion valide au domaine en exécutant l’applet de commande suivante dans le conteneur (à l’aide de `docker exec` ou d’un équivalent) :

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    La vérification d’approbation doit retourner `NERR_SUCCESS` si le gMSA est disponible et que la connectivité réseau permet au conteneur de communiquer avec le domaine. En cas d’échec, vérifiez la configuration réseau de l’ordinateur hôte et du conteneur. Les deux doivent être en mesure de communiquer avec le contrôleur de domaine.

4. Vérifiez si le conteneur peut obtenir un ticket TGT (Ticket Granting Ticket) Kerberos valide :

    ```powershell
    klist get krbtgt
    ```

    Cette commande doit renvoyer « un ticket à krbtgt a été récupéré avec succès » et répertorier le contrôleur de domaine utilisé pour récupérer le ticket. Si vous êtes en mesure d’obtenir un ticket TGT mais que `nltest` de l’étape précédente échoue, cela peut indiquer que le compte gMSA est mal configuré. Pour plus d’informations, consultez [vérifier le compte gMSA](#check-the-gmsa-account) .

    Si vous ne parvenez pas à obtenir un ticket TGT à l’intérieur du conteneur, cela peut indiquer des problèmes de connectivité réseau ou DNS. Assurez-vous que le conteneur peut résoudre un contrôleur de domaine à l’aide du nom DNS du domaine et que le contrôleur de domaine est routable à partir du conteneur.

5. Vérifiez que votre application est [configurée pour utiliser gMSA](gmsa-configure-app.md). Le compte d’utilisateur à l’intérieur du conteneur ne change pas lorsque vous utilisez un gMSA. Au lieu de cela, le compte système utilise le gMSA lorsqu’il communique avec d’autres ressources réseau. Cela signifie que votre application doit s’exécuter en tant que service réseau ou système local pour tirer parti de l’identité gMSA.

    > [!TIP]
    > Si vous exécutez `whoami` ou si vous utilisez un autre outil pour identifier votre contexte utilisateur actuel dans le conteneur, vous ne verrez pas le nom gMSA lui-même. Cela est dû au fait que vous vous connectez toujours au conteneur en tant qu’utilisateur local au lieu d’une identité de domaine. Le gMSA est utilisé par le compte d’ordinateur à chaque fois qu’il parle aux ressources réseau, ce qui explique pourquoi votre application doit s’exécuter en tant que service réseau ou système local.

### <a name="check-the-gmsa-account"></a>Vérifier le compte gMSA

1. Si votre conteneur semble être correctement configuré mais que les utilisateurs ou autres services ne peuvent pas s’authentifier automatiquement auprès de votre application en conteneur, vérifiez les noms de principal du service sur votre compte gMSA. Les clients recherchent le compte gMSA par le nom auquel ils accèdent à votre application. Cela peut signifier que vous aurez besoin de `host` SPN supplémentaires pour votre gMSA si, par exemple, les clients se connectent à votre application via un équilibreur de charge ou un nom DNS différent.

2. Vérifiez que gMSA et l’hôte de conteneur appartiennent au même domaine de Active Directory. L’hôte de conteneur ne peut pas récupérer le mot de passe gMSA si le gMSA appartient à un autre domaine.

3. Assurez-vous qu’il n’existe qu’un seul compte dans votre domaine portant le même nom que votre gMSA. les objets gMSA ont des signes dollar ($) ajoutés à leur nom de compte SAM. il est donc possible qu’un gMSA soit nommé « mon compte » et qu’un compte d’utilisateur non lié soit nommé « mon » dans le même domaine. Cela peut entraîner des problèmes si le contrôleur de domaine ou l’application doit rechercher le gMSA par son nom. Vous pouvez rechercher des objets nommés de la même façon dans Active Directory à l’aide de la commande suivante :

    ```powershell
    # Replace "GMSANAMEHERE" with your gMSA account name (no trailing dollar sign)
    Get-ADObject -Filter 'sAMAccountName -like "GMSANAMEHERE*"'
    ```

4. Si vous avez activé la délégation sans contrainte sur le compte gMSA, assurez-vous que l’indicateur `WORKSTATION_TRUST_ACCOUNT` est toujours activé sur l' [attribut userAccountControl](https://support.microsoft.com/en-us/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties) . Cet indicateur est requis pour que NETLOGon dans le conteneur communique avec le contrôleur de domaine, comme c’est le cas lorsqu’une application doit résoudre un nom en SID ou vice versa. Vous pouvez vérifier si l’indicateur est configuré correctement avec les commandes suivantes :

    ```powershell
    $gMSA = Get-ADServiceAccount -Identity 'yourGmsaName' -Properties UserAccountControl
    ($gMSA.UserAccountControl -band 0x1000) -eq 0x1000
    ```

    Si les commandes ci-dessus retournent `False`, utilisez la commande suivante pour ajouter l’indicateur `WORKSTATION_TRUST_ACCOUNT` à la propriété UserAccountControl du compte gMSA. Cette commande efface également les indicateurs `NORMAL_ACCOUNT`, `INTERDOMAIN_TRUST_ACCOUNT`et `SERVER_TRUST_ACCOUNT` de la propriété UserAccountControl.

    ```powershell
    Set-ADObject -Identity $gMSA -Replace @{ userAccountControl = ($gmsa.userAccountControl -band 0x7FFFC5FF) -bor 0x1000 }
    ```

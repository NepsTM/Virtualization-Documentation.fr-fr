---
title: Exécuter un conteneur avec un gMSA
description: Comment exécuter un conteneur Windows avec un compte de service géré de groupe (gMSA).
keywords: dockeur, conteneurs, Active Directory, GMSA, compte de service géré de groupe, comptes de service géré par groupe
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b9c0406b5fe9527d88365dabf0cfd10114c34c74
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079716"
---
# <a name="run-a-container-with-a-gmsa"></a>Exécuter un conteneur avec un gMSA

Pour exécuter un conteneur avec un [compte de service](https://docs.docker.com/engine/reference/run)géré par groupe (gMSA), spécifiez le fichier de `--security-opt` spécifications d’informations d’identification pour le paramètre de l’administrateur.

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

Si l’état de la connexion à un CD de confiance et `NERR_Success`l’état de vérification de l’approbation ne le sont pas, suivez les [instructions de dépannage](gmsa-troubleshooting.md#check-the-container) pour déboguer le problème.

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

## <a name="next-steps"></a>Étapes suivantes

En plus des conteneurs en cours d’exécution, vous pouvez également utiliser gMSAs pour:

- [Configurer des applications](gmsa-configure-app.md)
- [Orchestrer des conteneurs](gmsa-orchestrate-containers.md)

Si vous rencontrez des problèmes lors de l’installation, consultez notre [Guide de résolution des problèmes](gmsa-troubleshooting.md) pour consulter les solutions possibles.

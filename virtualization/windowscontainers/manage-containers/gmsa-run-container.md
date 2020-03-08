---
title: Exécuter un conteneur avec un gMSA
description: Comment exécuter un conteneur Windows avec un compte de service administré de groupe (gMSA).
keywords: ancrage, conteneurs, Active Directory, GMSA, compte de service administré de groupe, comptes de service administrés de groupe
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b997cf79cdf7f1782b6299198859714563c45f8c
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853933"
---
# <a name="run-a-container-with-a-gmsa"></a>Exécuter un conteneur avec un gMSA

Pour exécuter un conteneur avec un compte de service administré de groupe (gMSA), fournissez le fichier de spécification des informations d’identification au paramètre `--security-opt` de l’écran d' [ancrage](https://docs.docker.com/engine/reference/run):

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>Sur les versions 1709 et 1803 de Windows Server 2016, le nom d’hôte du conteneur doit correspondre au nom abrégé gMSA.

Dans l’exemple précédent, le nom de compte SAM gMSA est « webapp01 », de sorte que le nom d’hôte du conteneur est également nommé « webapp01 ».

Sur Windows Server 2019 et versions ultérieures, le champ nom d’hôte n’est pas obligatoire, mais le conteneur s’identifie toujours par le nom gMSA au lieu du nom d’hôte, même si vous en fournissez un explicitement différent.

Pour vérifier si le gMSA fonctionne correctement, exécutez l’applet de commande suivante dans le conteneur :

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

Si l’état de la connexion DC approuvée et l’état de vérification de l’approbation ne sont pas `NERR_Success`, suivez les [instructions de résolution des problèmes](gmsa-troubleshooting.md#check-the-container) pour déboguer le problème.

Vous pouvez vérifier l’identité gMSA à partir du conteneur en exécutant la commande suivante et en vérifiant le nom du client :

```powershell
PS C:\> klist get webapp01

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

Pour ouvrir PowerShell ou une autre application console en tant que compte gMSA, vous pouvez demander au conteneur de s’exécuter sous le compte service réseau au lieu du compte normal ContainerAdministrator (ou ContainerUser pour le serveur) :

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Quand vous exécutez en tant que service réseau, vous pouvez tester l’authentification réseau en tant que gMSA en tentant de se connecter à SYSVOL sur un contrôleur de domaine :

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>Étapes suivantes :

En plus des conteneurs en cours d’exécution, vous pouvez également utiliser service administrés pour :

- [Configurer des applications](gmsa-configure-app.md)
- [Orchestrer des conteneurs](gmsa-orchestrate-containers.md)

Si vous rencontrez des problèmes lors de l’installation, consultez notre [Guide de résolution](gmsa-troubleshooting.md) des problèmes pour obtenir des solutions possibles.

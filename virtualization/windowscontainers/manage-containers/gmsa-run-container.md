---
title: Exécuter un conteneur avec un compte de service administré de groupe
description: Comment exécuter un conteneur Windows avec un compte de service administré de groupe.
keywords: docker, conteneurs, Active Directory, compte de service administré de groupe, comptes de service administré de groupe
author: rpsqrd
ms.date: 09/10/2019
ms.topic: how-to
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 6168e882b8dffcfcec21648a637b04fe3478e4df
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985243"
---
# <a name="run-a-container-with-a-gmsa"></a>Exécuter un conteneur avec un compte de service administré de groupe

Pour exécuter un conteneur avec un compte de service administré de groupe, fournissez le fichier de spécifications d’informations d’identification au paramètre `--security-opt` de la commande [docker run](https://docs.docker.com/engine/reference/run) :

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>Sur les versions 1709 et 1803 de Windows Server 2016, le nom d’hôte du conteneur doit correspondre au nom abrégé du compte de service administré de groupe.

Dans l’exemple précédent, le nom de compte SAM du compte de service administré de groupe est « webapp01 », de sorte que le nom d’hôte du conteneur est également nommé « webapp01 ».

Sur Windows Server 2019 et versions ultérieures, le champ du nom d’hôte n’est pas obligatoire, mais le conteneur s’identifie toujours par le nom du compte de service administré de groupe au lieu du nom d’hôte, même si vous fournissez explicitement un nom différent.

Pour vérifier si le compte de service administré de groupe fonctionne correctement, exécutez l’applet de commande suivante dans le conteneur :

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

Si le Statut de la connexion du contrôleur de domaine approuvé et le Statut de vérification de l’approbation ne sont pas `NERR_Success`, suivez les [instructions de résolution des problèmes](gmsa-troubleshooting.md#check-the-container) pour effectuer le débogage du problème.

Vous pouvez vérifier l’identité du compte de service administré de groupe à partir du conteneur en exécutant la commande suivante et en vérifiant le nom du client :

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

Pour ouvrir PowerShell ou une autre application console en tant que compte de service administré de groupe, vous pouvez demander que le conteneur s’exécute en tant que compte Network Service plutôt qu’en tant que compte normal ContainerAdministrator (ou ContainerUser pour Nano Server) :

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Quand vous exécutez le conteneur en tant que Network Service, vous pouvez tester l’authentification réseau en tant que compte de service administré de groupe en tentant une connexion à SYSVOL sur un contrôleur de domaine :

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>Étapes suivantes

Outre l’exécution de conteneurs, vous pouvez utiliser des comptes de service administré de groupe pour effectuer les opérations suivantes :

- [Configurer des applications](gmsa-configure-app.md)
- [Orchestrer des conteneurs](gmsa-orchestrate-containers.md)

Si vous rencontrez des problèmes lors de la configuration, vous trouverez peut-être des solutions dans notre [Guide de résolution des problèmes](gmsa-troubleshooting.md).

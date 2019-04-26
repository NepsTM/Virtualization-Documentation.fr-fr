---
title: Gestion à distance d’un hôte WindowsDocker
description: Comment gérer en toute sécurité un hôte Docker distant exécutant Windows Server.
keywords: docker, conteneurs
author: taylorb-microsoft
ms.date: 02/14/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 0cc1b621-1a92-4512-8716-956d7a8fe495
ms.openlocfilehash: b975c593bd5c736ec3e7e1e21b76b2f6a2c8f8a4
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9575360"
---
# <a name="remote-management-of-a-windows-docker-host"></a>Gestion à distance d’un hôte WindowsDocker

Même en l’absence de `docker-machine`, il est toujours possible de créer un hôte Docker accessible à distance sur un ordinateur virtuel Windows Server2016.

Les étapes sont très simples:

* Créez les certificats sur le serveur à l’aide de [dockertls](https://hub.docker.com/r/stefanscherer/dockertls-windows/). Si vous créez les certificats avec une adresse IP, vous voudrez peut-être utiliser une adresse IP statique pour éviter d’avoir à recréer des certificats si l’adresse IP est modifiée.

* Redémarrer le service Docker `Restart-Service Docker`
* Rendez les ports TLS2375 et 2376 de Docker disponibles en créant une règle NSG permettant le trafic entrant. Notez que pour les connexions sécurisées, vous devez uniquement autoriser le port2376.  
  Le portail doit présenter une configuration NSG semblable à ce qui suit:  
  ![NGS](media/nsg.png)  
  
* Autorisez les connexions entrantes via le Pare-feu Windows. 
```
New-NetFirewallRule -DisplayName 'Docker SSL Inbound' -Profile @('Domain', 'Public', 'Private') -Direction Inbound -Action Allow -Protocol TCP -LocalPort 2376
```
* Copiez les fichiers `ca.pem`, cert.pem et key.pem contenus dans le dossier Docker de vos utilisateurs sur votre ordinateur, par exemple, `c:\users\chris\.docker` et collez-les sur l’ordinateur local. Par exemple, vous pouvez copier (ctrl-c), puis coller (ctrl-v) les fichiers d’une session RDP. 
* Vérifiez que vous pouvez vous connecter à l’hôte Docker distant. Exécuter
```
docker -D -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify --tlscacert=c:\
users\foo\.docker\client\ca.pem --tlscert=c:\users\foo\.docker\client\cert.pem --tlskey=c:\users\foo\.doc
ker\client\key.pem ps
```


## <a name="troubleshooting"></a>Résolution des problèmes
### <a name="try-connecting-without-tls-to-determine-your-nsg-firewall-settings-are-correct"></a>Essayez de vous connecter sans TLS pour vérifier que les paramètres du pare-feu NSG sont corrects
Des erreurs de connexion se traduisent généralement par des erreurs telles que:
```
error during connect: Get https://wsdockerhost.southcentralus.cloudapp.azure.com:2376/v1.25/version: dial tcp 13.85.27.177:2376: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.
```

Autorisez des connexions non chiffrées en ajoutant 
```
{
    "tlsverify":  false,
}
```
à `c"\programdata\docker\config\daemon.json`, puis redémarrez le service.

Connectez-vous à l’hôte distant avec une ligne de commande comme:
```
docker -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify=0 version
```

### <a name="cert-problems"></a>Problèmes de certificat
L’accès à l’hôte Docker avec un certificat non créé pour l’adresse IP ou le nom DNS se traduira par une erreur:
```
error during connect: Get https://w.x.y.c.z:2376/v1.25/containers/json: x509: certificate is valid for 127.0.0.1, a.b.c.d, not w.x.y.z
```
Vérifiez que w.x.y.z est le nom DNS correspondant à l’adresse IP publique de l’hôte et que le nom DNS correspond à celui du certificat [Nom commun](https://www.ssl.com/faqs/common-name/), qui était la variable d’environnement `SERVER_NAME` ou l’une des adresses IP dans la variable `IP_ADDRESSES` fournie à dockertls

### <a name="cryptox509-warning"></a>avertissement crypto/x509
Vous pouvez recevoir un avertissement 
```
level=warning msg="Unable to use system certificate pool: crypto/x509: system root pool is not available on Windows"
```
Cet avertissement est bénin.

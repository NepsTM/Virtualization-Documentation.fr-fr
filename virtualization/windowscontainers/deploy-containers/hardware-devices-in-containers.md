---
title: Appareils dans les conteneurs sur Windows
description: Prise en charge d’appareils pour les conteneurs sur Windows
keywords: station d’accueil, conteneurs, appareils, matériel
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910599"
---
# <a name="devices-in-containers-on-windows"></a>Appareils dans les conteneurs sur Windows

Par défaut, les conteneurs Windows bénéficient d’un accès minimal aux appareils hôtes, tout comme les conteneurs Linux. Certaines charges de travail sont utiles, voire impératives, pour accéder à des périphériques matériels hôtes et communiquer avec eux. Ce guide traite des appareils pris en charge dans les conteneurs et de la prise en main.

## <a name="requirements"></a>Configuration requise

Pour que cette fonctionnalité fonctionne, votre environnement doit répondre aux exigences suivantes :
- L’hôte de conteneur doit exécuter Windows Server 2019 ou Windows 10, version 1809 ou ultérieure.
- La version de l’image de base du conteneur doit être 1809 ou une version ultérieure.
- Vos conteneurs doivent être des conteneurs Windows exécutés en mode isolé du processus.
- L’hôte de conteneur doit exécuter le moteur d’ancrage 19,03 ou une version plus récente.

## <a name="run-a-container-with-a-device"></a>Exécuter un conteneur avec un appareil

Pour démarrer un conteneur avec un appareil, utilisez la commande suivante :

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Vous devez remplacer le `{interface class guid}` par un [GUID de classe d’interface d’appareil](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)approprié, qui se trouve dans la section ci-dessous.

Pour démarrer un conteneur avec plusieurs appareils, utilisez la commande et la chaîne suivantes ensemble plusieurs `--device` arguments :

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Dans Windows, tous les appareils déclarent une liste des classes d’interface qu’ils implémentent. En passant cette commande à l’Ancreur, il s’assure que tous les appareils qui identifient comme implémentant la classe demandée seront montés dans le conteneur.

Cela signifie que vous n’affectez **pas** l’appareil à l’ordinateur hôte. Au lieu de cela, l’hôte le partage avec le conteneur. De même, étant donné que vous spécifiez un GUID de classe, _tous les_ appareils qui implémentent ce GUID seront partagés avec le conteneur.

## <a name="what-devices-are-supported"></a>Appareils pris en charge

Les appareils suivants (et leurs GUID de classe d’interface appareil) sont pris en charge aujourd’hui :
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Device Type</center></th>
<th><center>GUID de la classe d’interface</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>GPIO</center></td>
<td><center>916EF1CB-8426-468D-A6F7-9AE8076881B3</center></td>
</tr>
<tr valign="top">
<td><center>Bus I2C</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>Port COM</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>Bus SPI</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>Accélération du GPU DirectX</center></td>
<td><center>Voir la documentation sur l' <a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">accélération GPU</a></center></td>
</tr>
</tbody>
</table>

> [!IMPORTANT]
> La prise en charge des appareils dépend du pilote. Toute tentative de passer des GUID de classe non définis dans le tableau ci-dessus peut entraîner un comportement indéfini.

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-V-prise en charge des conteneurs Windows isolés

L’attribution d’appareil et le partage d’appareil pour les charges de travail dans les conteneurs Windows isolés dans Hyper-V ne sont pas pris en charge actuellement.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-V-prise en charge des conteneurs Linux isolés

L’affectation d’appareils et le partage d’appareils pour les charges de travail dans les conteneurs Linux isolés par Hyper-V ne sont pas pris en charge actuellement.

---
title: Appareils dans les conteneurs sur Windows
description: La prise en charge des appareils pour les conteneurs sur Windows
keywords: arrimeur, conteneurs, appareils, matériel
author: cwilhit
ms.openlocfilehash: ee9c5da5ef87dceb3374977670da2ea50ea87382
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883162"
---
# <a name="devices-in-containers-on-windows"></a>Appareils dans les conteneurs sur Windows

Par défaut, les conteneurs Windows disposent d’un accès minimal aux appareils hôtes, de la même façon que les conteneurs Linux. Il existe certaines charges de travail pour lesquelles il est utile, ou même impératif, d’accéder aux périphériques matériels de l’hôte et de communiquer avec eux. Ce guide couvre les appareils qui sont pris en charge dans les conteneurs et la façon de commencer.

## <a name="requirements"></a>Spécifications

Pour que cette fonctionnalité fonctionne, votre environnement doit présenter la configuration suivante:
- L’hôte de conteneur doit exécuter Windows Server 2019 ou Windows 10 version 1809 ou ultérieure.
- Votre version d’image de base Container doit être 1809 ou une version ultérieure.
- Les conteneurs doivent être des conteneurs Windows en mode isolé du processus.
- L’hôte de conteneur doit exécuter le moteur d’amarrage 19,03 ou une version ultérieure.

## <a name="run-a-container-with-a-device"></a>Exécuter un conteneur avec un appareil

Pour démarrer un conteneur avec un appareil, utilisez la commande suivante:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Vous devez remplacer `{interface class guid}` par un GUID de [classe d’interface d’appareil](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)approprié, qui se trouve dans la section ci-dessous.

Pour démarrer un conteneur avec plusieurs appareils, utilisez la commande et la chaîne de caractères `--device` suivants avec plusieurs arguments:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Dans Windows, tous les appareils déclarent une liste de classes d’interface qu’ils implémentent. En passant cette commande à docker, elle garantit que tous les appareils qui identifient l’implémentation de la classe demandée seront montés dans le conteneur.

Cela signifie que vous n’affectez **pas** l’appareil à l’hôte. Au lieu de cela, l’hôte le partage avec le conteneur. De même, dans la mesure où vous spécifiez un GUID de classe, _tous les_ appareils qui implémentent ce GUID seront partagés avec le conteneur.

## <a name="what-devices-are-supported"></a>Quels sont les appareils pris en charge

Les appareils suivants (et leurs GUID de classe d’interface de périphérique) sont pris en charge pour le moment:
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Type d’appareil</center></th>
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
<td><center>Accélération GPU de DirectX</center></td>
<td><center>Voir les documents sur l' <a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">accélération GPU</a></center></td>
</tr>
</tbody>
</table>

> [!TIP]
> Les appareils indiqués ci-dessus sont les _seuls_ périphériques pris en charge dans les conteneurs Windows. La tentative de passage de tout autre GUID de classe provoquera un échec du démarrage du conteneur.

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-V-prise en charge de conteneur Windows isolés

L’affectation d’appareil et le partage d’appareil pour les charges de travail dans les conteneurs Windows isolés de Hyper-V ne sont pas pris en charge pour le moment.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-V-prise en charge des conteneurs Linux isolés

L’affectation d’appareil et le partage d’appareil pour les charges de travail dans les conteneurs Linux isolés d’Hyper-V ne sont pas pris en charge pour le moment.
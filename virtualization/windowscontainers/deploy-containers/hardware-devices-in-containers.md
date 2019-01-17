---
title: Appareils dans des conteneurs sur Windows
description: Quelle prise en charge de l’appareil existe pour les conteneurs sur Windows
keywords: docker, conteneurs, les appareils, matériel
author: cwilhit
ms.openlocfilehash: 6397a5050ee0c7cb4b62dc935af4975d9ab6b3db
ms.sourcegitcommit: 1b6a244c3604e48c42c851e580e3b59e2384c91a
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/17/2019
ms.locfileid: "9014516"
---
# <a name="devices-in-containers-on-windows"></a>Appareils dans des conteneurs sur Windows

Par défaut, les conteneurs Windows disposent d’un accès minimal aux appareils hôte--tout comme les conteneurs Linux. Il existe certaines charges de travail où il est utile--ou même impératif--pour accéder et communiquer avec les périphériques de matériel hôte. Ce guide couvre les appareils sont pris en charge dans les conteneurs et comment commencer.

## <a name="requirements"></a>Conditions préalables

- Vous devez exécuter Windows Server 2019 ou version ultérieure ou Windows 10 Professionnel/entreprise avec le 2018 octobre mettre à jour
- Votre version d’image de conteneur doit être 1809 ou une version ultérieure.
- Vos conteneurs doivent être en cours d’exécution en mode isolées du processus les conteneurs Windows.
- Tandis que la fonctionnalité d’appareils Windows existe dans le démon Docker, il n’existe pas encore dans le client Docker (voir cette [requête de tirage](https://github.com/docker/cli/pull/1606) pour effectuer le suivi). Dans l’interrim, vous devez [créer vos propres fichiers exécutables docker](https://github.com/moby/moby/blob/master/docs/contributing/software-req-win.md) à partir de la source de moby comme une solution de contournement. Si vous n’êtes pas cette opération à l’aise, nous vous recommandons de que vous attendre avant de tester cette fonctionnalité jusqu'à ce que la PR liée ci-dessus est fusionnée.

## <a name="run-a-container-with-a-device"></a>Exécuter un conteneur avec un appareil

Pour démarrer un conteneur avec un appareil, utilisez la commande suivante:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Vous devez remplacer le `{interface class guid}` avec une [classe d’interface de périphérique GUID](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/overview-of-device-interface-classes)appropriée, qui peut être trouvé dans la section ci-dessous.

Pour démarrer un conteneur avec différents appareils, utilisez la commande suivante et créer une chaîne de plusieurs `--device` arguments:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Dans Windows, tous les appareils déclarent une liste des classes d’interface qu’ils implémentent. En transmettant cette commande à Docker, il permet de garantir que tous les appareils qui identifient les en tant que l’implémentation de la classe demandée seront être montées dans le conteneur.

Cela signifie que vous affectez **pas** l’appareil hôte. Au lieu de cela, l’hôte est le partage avec le conteneur. De même, dans la mesure où vous spécifiez un GUID de la classe, _tous les_ périphériques qui implémentent ce GUID seront partagées avec le conteneur.

## <a name="what-devices-are-supported"></a>Ce que les appareils sont pris en charge

Les périphériques suivants (et leur appareil GUID de classe d’interface) sont pris en charge dès aujourd'hui:
  
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
</tbody>
</table>

> [!TIP]
> Les appareils répertoriés ci-dessus sont les _seuls_ les appareils pris en charge dans les conteneurs Windows aujourd'hui. Vous tentez de passer n’importe quel autre GUID de classe entraîne l’échec du démarrage de conteneur.

## <a name="hyper-v-container-device-support"></a>Prise en charge des périphériques de conteneur Hyper-V

Affectation de périphérique et le partage d’appareils ne sont pas pris en charge dans les conteneurs Hyper-V isolé aujourd'hui.

## <a name="linux-containers-on-windows-lcow-device-support"></a>Conteneurs Linux sur la prise en charge des périphériques Windows (LCOW)

Affectation de périphérique et le partage d’appareils ne sont pas pris en charge LCOW aujourd'hui.
---
title: Appareils dans des conteneurs sur Windows
description: Quelle prise en charge de l’appareil existe pour les conteneurs sur Windows
keywords: docker, conteneurs, les appareils, matériel
author: cwilhit
ms.openlocfilehash: feff730ed21c439312cda65c7b5ccc1a6cf5ae86
ms.sourcegitcommit: 2b456022ee666863ef53082580ac1d432de86939
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/16/2019
ms.locfileid: "9657357"
---
# <a name="devices-in-containers-on-windows"></a>Appareils dans des conteneurs sur Windows

Par défaut, les conteneurs Windows disposent d’un accès minimal aux appareils hôte--tout comme les conteneurs Linux. Il existe certaines charges de travail où il est utile--ou même impératif--pour accéder et communiquer avec les périphériques de matériel hôte. Ce guide couvre les appareils sont pris en charge dans les conteneurs et comment commencer.

> [!IMPORTANT]
> Cette fonctionnalité nécessite une version de Docker qui prend en charge la `--device` une option de ligne de commande pour les conteneurs Windows. Prise en charge de Docker formel est planifiée pour la prochaine version de Docker EE moteur 19.03. En attendant, la [source en amont](https://master.dockerproject.org/) pour Docker contient les bits nécessaires.

## <a name="requirements"></a>Spécifications

Pour cette fonctionnalité fonctionne, votre environnement doit satisfaire les conditions suivantes:
- L’hôte de conteneur doit exécuter Windows Server 2019 ou Windows 10, version 1809 ou une version ultérieure.
- Votre version d’image de base de conteneur doit être 1809 ou une version ultérieure.
- Vos conteneurs doivent être en cours d’exécution en mode isolées du processus les conteneurs Windows.
- L’hôte de conteneur doit être en cours d’exécution du moteur Docker 19.03 ou une version ultérieure.

## <a name="run-a-container-with-a-device"></a>Exécuter un conteneur avec un appareil

Pour démarrer un conteneur avec un appareil, utilisez la commande suivante:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Vous devez remplacer le `{interface class guid}` avec une [classe d’interface de périphérique GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)appropriée, qui peut être trouvé dans la section ci-dessous.

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
<tr valign="top">
<td><center>Accélération GPU DirectX</center></td>
<td><center>Voir la documentation de <a href="https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/gpu-acceleration">l’accélération GPU</a></center></td>
</tr>
</tbody>
</table>

> [!TIP]
> Les appareils répertoriés ci-dessus sont les _seuls_ les appareils pris en charge dans les conteneurs Windows aujourd'hui. Vous tentez de passer n’importe quel autre GUID de classe entraîne l’échec du démarrage de conteneur.

## <a name="hyper-v-isolated-windows-container-support"></a>Prise en charge du conteneur Hyper-V-isolé de Windows

Affectation de périphérique et d’appareil partage pour les charges de travail dans les conteneurs Windows isolé Hyper-V n'est pas pris en charge dès aujourd'hui.

## <a name="hyper-v-isolated-linux-container-support"></a>Prise en charge des conteneurs Linux Hyper-V-isolé

Affectation de périphérique et d’appareil partage pour les charges de travail dans des conteneurs Linux isolé Hyper-V n'est pas pris en charge dès aujourd'hui.

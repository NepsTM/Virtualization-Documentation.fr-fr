---
title: Configuration requise pour un conteneur Windows
description: Configuration requise pour un conteneur Windows.
keywords: "métadonnées, conteneurs"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: 08355d7a7da50d0f244bd750508fd42084818d7f
ms.openlocfilehash: ac48bc7ee7b70483d8a368749aea0862c52f049c

---

# Configuration requise pour un conteneur Windows

Ces guides fournissent la configuration requise pour un hôte de conteneur Windows.

## Configuration requise du système d’exploitation

- La fonctionnalité de conteneur Windows est disponible uniquement sur Windows Server 2016 (Core et avec expérience utilisateur), Nano Server ainsi que Windows 10 Professionnel et Entreprise (Édition anniversaire).
- Si les conteneurs Hyper-V sont exécutés, le rôle Hyper-V doit être installé.
- Windows doit être installé sur les hôtes de conteneur Windows Server, sur C:\. Si seuls des conteneurs Hyper-V sont déployés, cette restriction ne s’applique pas.

## Hôtes de conteneurs virtualisés

Si un hôte de conteneur Windows est exécuté sur une machine virtuelle Hyper-V et héberge également des conteneurs Hyper-V, la virtualisation imbriquée doit être activée. La configuration requise pour la virtualisation imbriquée est la suivante :

- Au moins 4 Go de RAM disponible pour l’hôte Hyper-V virtualisé.
- Windows Server 2016 ou Windows 10 sur le système hôte, et Windows Server (Full, Core) ou Nano Server sur la machine virtuelle.
- Un processeur Intel VT-x (cette fonctionnalité est actuellement disponible pour les processeurs Intel uniquement).
- La machine virtuelle de l’hôte de conteneur doit aussi disposer d’au moins 2 processeurs virtuels.

## Images de base prises en charge

Les conteneurs Windows fournissent deux images de base de conteneur (Windows Server Core et Nano Server). Les configurations ne prennent pas toutes en charge les deux images de système d’exploitation. Ce tableau détaille les configurations prises en charge.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Système d’exploitation invité</center></th>
<th><center>Conteneur Windows Server</center></th>
<th><center>Conteneur Hyper-V</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016 avec Expérience utilisateur</center></td>
<td><center>Server Core / Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Server Core / Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server</center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Professionnel / Entreprise</center></td>
<td><center>Non disponible</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
</tbody>
</table>


<!--HONumber=Oct16_HO2-->



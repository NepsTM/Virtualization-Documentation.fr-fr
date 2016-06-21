---
title: Configuration requise pour un conteneur Windows
description: Configuration requise pour un conteneur Windows.
keywords: metadata, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
---

# Configuration requise pour un conteneur Windows

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

Ces guides fournissent la configuration requise pour un hôte de conteneur Windows.

## Configuration requise du système d’exploitation

- Le rôle de conteneur Windows est uniquement disponible sur Windows Server 2016 TP5 (Full et Core), Nano Server et Windows 10 (Insiders build 14352 et ultérieures).
- Si les conteneurs Hyper-V sont exécutés, le rôle Hyper-V doit être installé.
- Windows doit être installé sur C:\\ sur les hôtes de conteneur Windows Server. Si seuls les conteneurs Hyper-V sont déployés, cette restriction ne s’applique pas.

## Hôtes de conteneurs virtualisés

Si un hôte de conteneur Windows est exécuté sur une machine virtuelle Hyper-V et héberge également des conteneurs Hyper-V, la virtualisation imbriquée doit être activée. La configuration requise pour la virtualisation imbriquée est la suivante :

- Au moins 4 Go de RAM disponible pour l’hôte Hyper-V virtualisé.
- Windows Server 2016 Technical Preview 5 ou Windows 10 build 10565 sur le système hôte, et Windows Server Technical Preview 5 (Full, Core) ou Nano Server sur la machine virtuelle.
- Un processeur Intel VT-x (cette fonctionnalité est actuellement disponible pour les processeurs Intel uniquement).
- La machine virtuelle de l’hôte de conteneur doit aussi disposer d’au moins 2 processeurs virtuels.

## Images de système d’exploitation prises en charge

Windows Server Technical Preview 5 est fourni avec deux images de système d’exploitation de conteneur, Windows Server Core et Nano Server. Les configurations ne prennent pas toutes en charge les deux images de système d’exploitation. Ce tableau détaille les configurations prises en charge.

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
<td><center>Interface utilisateur Windows Server 2016 Full</center></td>
<td><center>Image Server Core</center></td>
<td><center>Image Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Image Server Core</center></td>
<td><center> Image Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Image Nano Server</center></td>
<td><center>Image Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Versions de Windows 10 Insider</center></td>
<td><center>Non disponible</center></td>
<td><center>Image Nano Server</center></td>
</tr>
</tbody>
</table>


<!--HONumber=May16_HO4-->



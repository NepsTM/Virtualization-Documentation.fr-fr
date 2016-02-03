# Configuration requise pour un conteneur Windows

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.**

Ces guides fournissent la configuration requise pour un hôte de conteneur Windows.

## Images de système d’exploitation prises en charge

Windows Server Technical Preview 4 est offert avec deux images de système d’exploitation de conteneur, Windows Server Core et Nano Server. Les configurations ne prennent pas toutes en charge les deux images de système d’exploitation. Ce tableau détaille les configurations prises en charge.

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
<td><center>Image de système d’exploitation de base</center></td>
<td><center>Image de système d’exploitation Nano</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Image de système d’exploitation de base</center></td>
<td><center> Image de système d’exploitation Nano</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Image de système d’exploitation Nano</center></td>
<td><center>Image de système d’exploitation Nano</center></td>
</tr>
</tbody>
</table>

## Configuration requise pour un conteneur Hyper-V

Si un hôte de conteneur Windows est exécuté sur une machine virtuelle Hyper-V et héberge également des conteneurs Hyper-V, la virtualisation imbriquée doit être activée. La configuration requise pour la virtualisation imbriquée est la suivante :

- Au moins 4 Go de RAM disponible pour l’hôte Hyper-V virtualisé.
- Windows Server 2016 Technical Preview 4 ou Windows 10 build 10565, à la fois sur l’hôte physique et l’hôte virtualisé.
- Un processeur Intel VT-x (cette fonctionnalité est actuellement disponible pour les processeurs Intel uniquement).
- La machine virtuelle de l’hôte de conteneur doit aussi avoir au moins 2 processeurs virtuels.





<!--HONumber=Jan16_HO1-->

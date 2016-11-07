---
title: Documentation sur les conteneurs Windows
description: Documentation sur les conteneurs Windows
keywords: docker, conteneurs
author: enderb-ms
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 74c9d604-0915-4d89-bc69-0263b76bc66b
translationtype: Human Translation
ms.sourcegitcommit: 1787637fdd2c3bf8ef453a7425dc965e65e5ce12
ms.openlocfilehash: a1b876d01b8076ee9feb275bd09247775bfcef69

---

# Documentation sur les conteneurs Windows

Les conteneurs Windows offrent une virtualisation au niveau du système d’exploitation qui permet à plusieurs applications isolées d’être exécutées sur un seul système. Deux types de runtime de conteneurs différents sont compris dans la fonctionnalité, chacun avec un degré différent d’isolation d’application. Pour procéder à l’isolation, les conteneurs Windows Server passent par l’isolation de processus et d’espace de noms. Les conteneurs Hyper-V encapsulent chaque conteneur dans une machine virtuelle légère. L’ensemble de cette documentation fournit des guides de démarrage rapide, des guides de déploiement et des détails techniques sur les opérations de gestion.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:90%" cellpadding="25" cellspacing="5">
<tr>
<td ><center>![](media/try.png)</center></td>
<td>**Démarrage rapide de la**<br /><br />
Démarrage rapide de Windows Server<br /><br />
<ul>
<li>[Étape 1 : Concepts et terminologie](quick_start/quick_start.md)<br /><br /></li>
<li>[Étape 2 : Configurer Windows Server et le premier conteneur](quick_start/quick_start_windows_server.md)<br /><br /></li>
<li>[Étape 3 : Créer et transférer (push) des images de conteneur](quick_start/quick_start_images.md)<br /><br /></li>
</ul>
Démarrage rapide de Windows 10<br /><br />
<ul>
<li>[Étape 1 : Concepts et terminologie](quick_start/quick_start.md)<br /><br /></li>
<li>[Étape 2 : Configurer Windows 10 et le premier conteneur](quick_start/quick_start_windows_10.md)<br /><br /></li>
</ul>
</td>
</tr>
<tr>
<td ><center>![](media/1.png)</center></td>
<td>**Déploiement**<br /><br />
Découvrez comment déployer les conteneurs Windows sur Windows Server 2016 et Nano Server.<br /><br />
<ul>
<li>[Configuration requise](deployment/system_requirements.md)<br /><br /></li>
<li>[Déployer l’hôte de conteneur - Windows Server](deployment/deployment.md)<br /><br /></li>
<li>[Déployer l’hôte de conteneur - Nano Server](deployment/deployment_nano.md)<br /><br /></li>
<li>[Optimisation de la protection antivirus](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)<br /><br /></li>
</ul>
</td>
</tr>

<tr>
<td ><center>![](media/explore.png)</center></td>
<td>**Docker sur Windows**<br /><br />
Découvrez comment gérer Docker sur Windows.<br /><br />
<ul>
<li>[Moteur Docker sur Windows](docker/configure_docker_daemon.md)<br /><br /></li>
<li>[Fichiers Dockerfile sur Windows](docker/manage_windows_dockerfile.md)<br /><br /></li>
<li>[Gérer les données de conteneur](management/manage_data.md)<br /><br /></li>
<li>[Optimiser les fichiers Dockerfile](docker/optimize_windows_dockerfile.md)<br /><br /></li>
<li>[Mise en réseau de conteneur](management/container_networking.md)<br /><br /></li>
</ul>
</td>
</tr>

<tr>
<td ><center>![](media/video.png)</center></td>
<td>**Regardez**<br /><br />
Vous êtes intéressé par des démonstrations et des interviews de l’équipe des conteneurs Windows ?<br /><br />
<ul>
<li>[Chaîne Conteneurs](https://channel9.msdn.com/Blogs/containers)</li>
</ul>
<br />
</td>
</tr>

<tr>
<td ><center>![](media/question.png)</center></td>
<td>**Communauté**<br /><br />
Interagissez avec la communauté, testez des exemples et trouvez d’autres ressources.<br /><br />
<ul>
<li>[Forum sur les conteneurs](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)<br /><br /></li>
<li>[Vidéos et blogs de la communauté](communitylinks.md)<br /><br /></li>
<li>[Ressources de conteneur](https://msdn.microsoft.com/virtualization/community/community_overview)<br /><br /></li>
</ul>
</td>
</tr>
</table>



<!--HONumber=Nov16_HO1-->



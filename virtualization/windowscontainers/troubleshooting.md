---
title: "Résolution des problèmes liés aux conteneurs Windows"
description: "Conseils de dépannage, scripts automatisés et informations de journal pour les conteneurs Windows et Docker"
keywords: "Docker, conteneurs, résolution des problèmes, journaux"
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
translationtype: Human Translation
ms.sourcegitcommit: c65353d0b6dff233819dcc8f4f92eb186bf3b8fc
ms.openlocfilehash: 9f28c35c6eaddd8bcf3883863b63251378f845a7

---

# Résolution des problèmes

Vous rencontrez des difficultés pour configurer votre ordinateur ou exécuter un conteneur ? Nous avons créé un script PowerShell pour rechercher les problèmes courants. Commencez par le tester pour voir ce qu’il trouve, puis partagez vos résultats.

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
La liste de tous les tests qu’il exécute avec des solutions courantes est disponible dans le [fichier Lisez-moi](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md) du script.

Si cela ne permet pas de trouver la source du problème, publiez la sortie de votre script sur le [Forum sur les conteneurs](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers). C’est l’endroit idéal pour obtenir de l’aide de la Communauté, notamment des Windows Insiders et des développeurs Windows.


## Où trouver les journaux
Plusieurs services sont utilisés pour gérer les conteneurs Windows. Les sections suivantes indiquent où obtenir les journaux de chaque service.

### Moteur Docker
Les enregistrements du moteur Docker sont consignés dans le journal des événements des applications Windows, plutôt que dans un fichier. Ces enregistrements peuvent facilement être lus, triés et filtrés à l’aide de Windows PowerShell

Par exemple, cette commande affiche les enregistrements du moteur Docker des 5 dernières minutes en commençant par le plus ancien.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

Ces enregistrements peuvent aussi être facilement redirigés dans un fichier CSV pour être lus par un autre outil ou une feuille de calcul.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

#### Activation de la journalisation du débogage
Vous pouvez également activer la journalisation au niveau du débogage sur le moteur Docker. Cette opération peut être utile pour la résolution des problèmes si les journaux ordinaires ne contiennent pas suffisamment de détails.

Tout d’abord, ouvrez une invite de commandes avec élévation de privilèges, puis exécutez `sc.exe qc docker` pour obtenir la ligne de commande actuelle pour le service Docker.
Exemple :
```none
C:\> sc.exe qc docker
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: docker
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\Docker\dockerd.exe" --run-service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Docker Engine
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

Prenez la valeur `BINARY_PATH_NAME` actuelle, puis modifiez-la :
- Ajoutez -D à la fin
- Placez chaque " dans une séquence d’échappement avec \
- Placez l’ensemble de la commande entre "

Exécutez ensuite la commande `sc.exe config docker binpath= ` suivie de la nouvelle chaîne. Exemple : 
```none
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


Redémarrez maintenant le service Docker
```none
sc.exe stop docker
sc.exe start docker
```

Beaucoup plus d’informations sont ainsi consignées dans le journal des événements de l’application. Il est donc préférable de supprimer l’option `-D` une fois la résolution des problèmes terminée. Suivez la même procédure que celle mentionnée ci-dessus sans `-D` et redémarrez le service pour désactiver la journalisation du débogage.


### Service de conteneur hôte
Le moteur Docker dépend d’un service de conteneur hôte spécifique à Windows. Il a des journaux distincts : 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

Ils sont visibles dans l’Observateur d’événements et peuvent également être interrogés avec PowerShell.

Exemple :
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```




<!--HONumber=Jan17_HO4-->



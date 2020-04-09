---
title: Résolution des problèmes liés aux conteneurs Windows
description: Conseils de dépannage, scripts automatisés et informations de journal pour les conteneurs Windows et Docker
keywords: Docker, conteneurs, résolution des problèmes, journaux
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
ms.openlocfilehash: 752df5de208887b149460a204bbb6ff74393e809
ms.sourcegitcommit: b140ac14124e4bee3c7f31a7f8274d4a0ccb2dda
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/08/2020
ms.locfileid: "80929973"
---
# <a name="troubleshooting"></a>Résolution des problèmes

Vous rencontrez des difficultés pour configurer votre ordinateur ou exécuter un conteneur ? Nous avons créé un script PowerShell pour rechercher les problèmes courants. Commencez par le tester pour voir ce qu’il trouve, puis partagez vos résultats.

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
La liste de tous les tests qu’il exécute avec des solutions courantes est disponible dans le [fichier Lisez-moi](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md) du script.

Si cela ne permet pas de trouver la source du problème, publiez la sortie de votre script sur le [Forum sur les conteneurs](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers). C’est l’endroit idéal pour obtenir de l’aide de la Communauté, notamment des Windows Insiders et des développeurs Windows.


### <a name="finding-logs"></a>Où trouver les journaux
Plusieurs services sont utilisés pour gérer les conteneurs Windows. Les sections suivantes indiquent où obtenir les journaux de chaque service.

## <a name="docker-container-logs"></a>Journaux de conteneurs de l’ancrage 
La commande `docker logs` extrait les journaux d’un conteneur à partir de STDOUT/STDERR, les emplacements de dépôt des journaux des applications standard pour les applications Linux. Les applications Windows ne se connectent généralement pas à STDOUT/STDERR ; au lieu de cela, ils sont consignés dans ETW, les journaux des événements ou les fichiers journaux, entre autres. 

Le [moniteur du journal](https://github.com/microsoft/windows-container-tools/tree/master/LogMonitor), un outil open source pris en charge par Microsoft, est désormais disponible sur GitHub. Le moniteur du journal relie les journaux des applications Windows à STDOUT/STDERR. Le moniteur du journal est configuré via un fichier de configuration. 

### <a name="log-monitor-usage"></a>Utilisation du moniteur de journal

LogMonitor. exe et LogMonitorConfig. JSON doivent être tous deux inclus dans le même répertoire LogMonitor. 

Le moniteur du journal peut être utilisé dans un modèle d’utilisation de l’INTERPRÉTeur de commandes :

```
SHELL ["C:\\LogMonitor\\LogMonitor.exe", "cmd", "/S", "/C"]
CMD c:\windows\system32\ping.exe -n 20 localhost
```

Ou un modèle d’utilisation de point d’entrée :

```
ENTRYPOINT C:\LogMonitor\LogMonitor.exe c:\windows\system32\ping.exe -n 20 localhost
```

Les deux exemples d’utilisation encapsulent l’application ping. exe. Autres applications (comme [IIS). ServiceMonitor]( https://github.com/microsoft/IIS.ServiceMonitor)) peut être imbriqué avec le moniteur de journal de la même manière :

```
COPY LogMonitor.exe LogMonitorConfig.json C:\LogMonitor\
WORKDIR /LogMonitor
SHELL ["C:\\LogMonitor\\LogMonitor.exe", "powershell.exe"]
 
# Start IIS Remote Management and monitor IIS
ENTRYPOINT      Start-Service WMSVC; `
                    C:\ServiceMonitor.exe w3svc;
```


Le moniteur de journal démarre l’application encapsulée en tant que processus enfant et surveille la sortie STDOUT de l’application.

Notez que dans le modèle d’utilisation de l’INTERPRÉTeur de commandes, l’instruction CMD/ENTRYPOINT doit être spécifiée dans le formulaire de l’INTERPRÉTeur de commandes et non dans le formulaire exec. Quand le formulaire exec de l’instruction CMD/ENTRYPOINT est utilisé, l’INTERPRÉTeur de commandes n’est pas lancé et l’outil d’analyse du journal n’est pas lancé à l’intérieur du conteneur.

Vous trouverez plus d’informations sur l’utilisation sur le [wiki du moniteur des journaux](https://github.com/microsoft/windows-container-tools/wiki). Vous trouverez des exemples de fichiers de configuration pour les principaux scénarios de conteneur Windows (IIS, etc.) dans le [référentiel GitHub](https://github.com/microsoft/windows-container-tools/tree/master/LogMonitor/src/LogMonitor/sample-config-files). Vous trouverez un contexte supplémentaire dans ce billet de [blog](https://techcommunity.microsoft.com/t5/Containers/Windows-Containers-Log-Monitor-Opensource-Release/ba-p/973947).

## <a name="docker-engine"></a>Moteur Docker
Les enregistrements du moteur Docker sont consignés dans le journal des événements des applications Windows, plutôt que dans un fichier. Ces enregistrements peuvent facilement être lus, triés et filtrés à l’aide de Windows PowerShell

Par exemple, cette commande affiche les enregistrements du moteur Docker des 5 dernières minutes en commençant par le plus ancien.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

Ces enregistrements peuvent aussi être facilement redirigés dans un fichier CSV pour être lus par un autre outil ou une feuille de calcul.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

### <a name="enabling-debug-logging"></a>Activation de la journalisation du débogage
Vous pouvez également activer la journalisation au niveau du débogage sur le moteur Docker. Cette opération peut être utile pour la résolution des problèmes si les journaux ordinaires ne contiennent pas suffisamment de détails.

Tout d’abord, ouvrez une invite de commandes avec élévation de privilèges, puis exécutez `sc.exe qc docker` pour obtenir la ligne de commande actuelle pour le service Docker.
Exemple :
```
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

Exécutez ensuite la commande `sc.exe config docker binpath=` suivie de la nouvelle chaîne. Par exemple : 
```
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


Redémarrez maintenant le service Docker
```
sc.exe stop docker
sc.exe start docker
```

Beaucoup plus d’informations sont ainsi consignées dans le journal des événements de l’application. Il est donc préférable de supprimer l’option `-D` une fois la résolution des problèmes terminée. Suivez la même procédure que celle mentionnée ci-dessus sans `-D` et redémarrez le service pour désactiver la journalisation du débogage.

Vous pouvez aussi exécuter le démon Docker en mode débogage à partir d’une invite PowerShell avec élévation de privilèges, capturant ainsi la sortie directement dans un fichier.
```PowerShell
sc.exe stop docker
<path\to\>dockerd.exe -D > daemon.log 2>&1
```

### <a name="obtaining-stack-dump"></a>Obtention d’un vidage de pile

En règle générale, cela s’avère utile uniquement si le support technique de Microsoft ou les développeurs de l’arrimeur le demandent explicitement. Il peut être utilisé pour aider à diagnostiquer une situation où la station d’accueil semble avoir été suspendue. 

Télécharger [docker-signal.exe](https://github.com/moby/docker-signal).

Utilisation :
```PowerShell
docker-signal --pid=$((Get-Process dockerd).Id)
```

Le fichier de sortie se trouve dans le répertoire de la racine de données que la station d’accueil s’exécute dans. Le répertoire par défaut est `C:\ProgramData\Docker`. Le répertoire réel peut être confirmé en exécutant `docker info -f "{{.DockerRootDir}}"`.

Le fichier sera `goroutine-stacks-<timestamp>.log`.

Notez que `goroutine-stacks*.log` ne contient pas d’informations personnelles.


## <a name="host-compute-service"></a>Service de calcul hôte
Le moteur Docker dépend d’un service de calcul hôte spécifique à Windows. Il a des journaux distincts : 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

Ils sont visibles dans observateur d’événements et peuvent également être interrogés avec PowerShell.

Par exemple :
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```

### <a name="capturing-hcs-analyticdebug-logs"></a>Enregistrement des journaux d’analyse/débogage HCS

Pour activer les journaux d’analyse/débogage de calcul Hyper-V et les enregistrer sur `hcslog.evtx`.

```PowerShell
# Enable the analytic logs
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:true /q:true

# <reproduce your issue>

# Export to an evtx
wevtutil.exe epl Microsoft-Windows-Hyper-V-Compute-Analytic <hcslog.evtx>

# Disable
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:false /q:true
```

### <a name="capturing-hcs-verbose-tracing"></a>Capturer le suivi HCS en clair

En règle générale, cela n'est utile que si le support Microsoft le demande. 

Télécharger [HcsTraceProfile.wprp](https://github.com/MicrosoftDocs/Virtualization-Documentation/blob/master/windows-server-container-tools/wpr-profiles/HcsTraceProfile.wprp)

```PowerShell
# Enable tracing
wpr.exe -start HcsTraceProfile.wprp!HcsArgon -filemode

# <reproduce your issue>

# Capture to HcsTrace.etl
wpr.exe -stop HcsTrace.etl "some description"
```

Fournir `HcsTrace.etl` à votre contact de support.

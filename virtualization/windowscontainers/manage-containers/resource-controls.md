---
title: Implémentation de contrôles de ressources
description: Détails concernant les contrôles de ressources pour les conteneurs Windows
keywords: docker, conteneurs, processeur, mémoire, disque, ressources
author: taylorb-microsoft
ms.date: 11/21/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8ccd4192-4a58-42a5-8f74-2574d10de98e
ms.openlocfilehash: 3e9f7e3208222cd6c0f512c5f892453ac6e6980c
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910169"
---
# <a name="implementing-resource-controls-for-windows-containers"></a>Implémentation de contrôles de ressources pour les conteneurs Windows
Plusieurs contrôles de ressources peuvent être implémentés par conteneur et par ressource.  Par défaut, les conteneurs exécutés sont soumis à la gestion des ressources Windows classique. Celle-ci est généralement exécutée de manière équitable, mais en configurant ces contrôles, un développeur ou un administrateur est en mesure de limiter ou d’influencer l’utilisation des ressources.  Les ressources pouvant être contrôlées sont notamment : UC/processeur, mémoire/mémoire RAM, disque/stockage et réseau/débit.

Les conteneurs Windows utilisent des [objets de traitement](https://docs.microsoft.com/windows/desktop/ProcThread/job-objects) pour regrouper les processus associés à chaque conteneur et en assurer le suivi.  Les contrôles de ressources sont implémentés sur l’objet de traitement parent associé au conteneur. 

Dans le cas d’[isolation Hyper-V](./hyperv-container.md), les contrôles de ressources sont appliqués à la fois à l’ordinateur virtuel et à l’objet de traitement du conteneur en cours d’exécution automatique sur l’ordinateur virtuel. Cela garantit que, même si un processus en cours d’exécution dans le conteneur contourne ou échappe aux contrôles de l’objet de traitement, l’ordinateur virtuel s’assurera qu’il ne peut pas dépasser les contrôles de ressources définis.

## <a name="resources"></a>Ressources
Pour chaque ressource, cette section fournit un mappage entre l’interface de ligne de commande Docker, comme exemple d’utilisation du contrôle de ressources (il peut être configuré par un orchestrateur ou d’autres outils) et l’API du service de calcul hôte (HCS) Windows correspondante, en indiquant la façon dont le contrôle de ressources a généralement été implémenté par Windows (notez que cette description est une vue d’ensemble et que l’implémentation sous-jacente est susceptible d’être modifiée).

|  | |
| ----- | ------|
| *Mémoire* ||
| Interface Docker | [--mémoire](https://docs.docker.com/engine/admin/resource_constraints/#memory) |
| Interface HCS | [MemoryMaximumInMB](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Noyau partagé | [JOB_OBJECT_LIMIT_JOB_MEMORY](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_basic_limit_information) |
| Isolation Hyper-V | Mémoire de l’ordinateur virtuel |
| _Remarque relative à l’isolation Hyper-V dans Windows Server 2016 : quand vous utilisez une limite de mémoire, vous voyez que le conteneur alloue initialement la quantité de mémoire Cap, puis commence à le renvoyer à l’hôte de conteneur.  Dans les versions ultérieures (1709 ou supérieur), cela a été optimisé._ |
| ||
| *UC (nombre)* ||
| Interface Docker | [--processeurs](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| Interface HCS | [ProcessorCount](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Noyau partagé | Simulé avec [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information)* |
| Isolation Hyper-V | Nombre de processeurs virtuels exposés |
| ||
| *PROCESSEUR (en pourcentage)* ||
| Interface Docker | [--UC-pourcentage](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| Interface HCS | [ProcessorMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Noyau partagé | [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Isolation Hyper-V | Limites de l’hyperviseur sur les processeurs virtuels |
| ||
| *UC (partages)* ||
| Interface Docker | [--cpu-shares](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| Interface HCS | [ProcessorWeight](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Noyau partagé | [JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Isolation Hyper-V | Poids de processeurs virtuels de l’hyperviseur |
| ||
| *Stockage (image)* ||
| Interface Docker | [--IO-MaxBandwidth/--IO-maxiops](https://docs.docker.com/edge/engine/reference/commandline/run/#usage) |
| Interface HCS | [StorageIOPSMaximum et StorageBandwidthMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Noyau partagé | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Isolation Hyper-V | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| ||
| *Stockage (volumes)* ||
| Interface Docker | [--Storage-opt Size =](https://docs.docker.com/edge/engine/reference/commandline/run/#set-storage-driver-options-per-container) |
| Interface HCS | [StorageSandboxSize](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Noyau partagé | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Isolation Hyper-V | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |

## <a name="additional-notes-or-details"></a>Remarques supplémentaires ou détails

### <a name="memory"></a>Mémoire

Les conteneurs Windows exécutent quelques processus système dans chaque conteneur. Il s’agit généralement de ceux qui fournissent des fonctionnalités par conteneur, comme la gestion des utilisateurs, la mise en réseau, etc… Si la majeure partie de la mémoire requise par ces processus est partagée entre les conteneurs, la limite de mémoire doit être suffisamment élevée pour l’ensemble des conteneurs.  Un tableau est fourni dans le document [Configuration requise](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/system-requirements#memory-requirments) pour chaque type d’image de base, avec et sans isolation Hyper-V.

### <a name="cpu-shares-without-hyper-v-isolation"></a>Partages UC (sans isolation Hyper-V)

Lors de l’utilisation de partages de processeur, l’implémentation sous-jacente (lorsque l’isolation Hyper-V n’est pas utilisée) configure [JOBOBJECT_CPU_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information), en définissant de manière spécifique l’indicateur du contrôle sur JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED et en fournissant un poids approprié.  Les plages de poids valides pour l’objet de traitement sont comprises entre 1 et 9 et la valeur par défaut est 5, ce qui correspond à une fidélité plus faible que les valeurs de services de calcul hôtes qui sont comprises entre 1 à 10 000.  Par exemple, un poids de partage de 7 500 entraînerait un poids égal à 7, tandis qu’un poids de partage de 2 500 produirait la valeur 2.

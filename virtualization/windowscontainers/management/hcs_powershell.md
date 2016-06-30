---
title: HCS PowerShell
description: Utilisez des conteneurs HCS PowerShell et Windows.
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 45144ec5-f76a-4460-abd1-9b60e47506d6
translationtype: Human Translation
ms.sourcegitcommit: cfa3c14e932f8b86edf6667200ac028ea0a16b67
ms.openlocfilehash: 413b9de08d182635908bc11ce3efc7623bb440e4

---

# Interopérabilité de gestion

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.** 

## Afficher tous les conteneurs

Pour retourner une liste de conteneurs, utilisez la commande `Get-ComputeProcess`.

```none
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## Arrêter un conteneur

Pour arrêter un conteneur créé avec PowerShell ou Docker, utilisez la commande `Stop-ComputeProcess`.

> À l’heure où nous écrivons cet article, le service VMMS doit être redémarré pour que les conteneurs apparaissent arrêtés lors de l’utilisation de la commande `Get-Container`.

```none
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```



<!--HONumber=Jun16_HO4-->



---
title: Images de base du conteneur Windows
description: Vue d’ensemble des images de base du conteneur Windows et de leur utilisation.
keywords: Docker, conteneurs, hachages
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 2a69fbace51589cce08476bd68fdb5c34a7907e6
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254269"
---
# <a name="container-base-images"></a>Images de base du conteneur

Windows propose quatre images de base Container à partir desquelles les utilisateurs peuvent créer. Chaque image de base est une version différente du système d’exploitation Windows, présente un encombrement sur disque différent et utilise un volume différent de l’API Windows définie.

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-servercore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows Server Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Prend en charge les applications .NET Framework traditionnelles.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-nanoserver" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Nano Server</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Conçu pour les applications .NET Core.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Fournit l’ensemble d’API Windows complet.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-iotcore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">WindowsIoT Standard</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Conçu spécialement pour les applications IoT.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>Découverte d’images

Toutes les images de base conteneur Windows peuvent être découvertes par le biais du Hub de l' [amarrage](https://hub.docker.com/_/microsoft-windows-base-os-images). Les images de base du conteneur Windows sont prises en charge par [MCR.Microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/), le registre de conteneurs Microsoft. C’est pourquoi les commandes de collecte pour les images de la base du conteneur Windows se présentent comme suit:

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

La prise en charge des compartiments ne dispose pas de son propre environnement et est conçue pour prendre en charge des catalogues existants tels que le hub de l’amarrage. Grâce à l’encombrement mondial d’Azure et au contenu d’Azure CDN, la fonction de multifonction fournit une interface d’extraction d’image qui est cohérente et rapide. Les clients Azure, qui exécutent leurs charges de travail dans Azure, bénéficient d’améliorations apportées aux performances du réseau, d’une intégration étroite avec la fonction de multifonction (source pour les images de conteneurs Microsoft), d’Azure Marketplace et du nombre étendu de services dans Azure. conteneurs en tant que format de package de déploiement.

## <a name="choosing-a-base-image"></a>Choix d’une image de base

Comment choisir l’image de base appropriée pour la création? Pour la plupart des `Windows Server Core` utilisateurs `Nanoserver` et sera l’image la plus appropriée à utiliser.

### <a name="guidelines"></a>Recommandations

 Même si vous êtes libre de cibler l’image que vous souhaitez, voici quelques conseils pour vous aider à mieux diriger votre choix:

- **Votre application nécessite-t-elle l’intégralité du .NET Framework?** Si la réponse à cette question est oui, vous devez cibler `Windows Server Core`.
- **Vous créez une application Windows basée sur .NET Core?** Si la réponse à cette question est oui, vous devez cibler `Nanoserver`.
- **Créez-vous une application IoT?** Si la réponse à cette question est oui, vous devez cibler `IoT Core`.
- **L’image du conteneur Windows Server Core ne dispose-t-elle pas d’une dépendance requise par votre application?** Si la réponse à cette question est oui, essayez de vous cibler `Windows`. Cette image est beaucoup plus grande que les autres images de base, mais elle comporte de nombreuses bibliothèques Windows principales (par exemple, la bibliothèque GDI).
- **Êtes-vous un Windows Insider?** Si oui, vous devez envisager d’utiliser la version Insider des images. Voir «images de base pour les Windows Insiders» ci-dessous.

> [!TIP]
> De nombreux utilisateurs Windows souhaitent emporter des applications qui ont une dépendance sur .NET. Outre les quatre images de base décrites dans cet article, Microsoft publie plusieurs images de conteneur Windows qui sont préconfigurées avec des infrastructures Microsoft populaires, telles qu’une image [.NET Framework](https://hub.docker.com/_/microsoft-dotnet-framework) et l’image [ASP .net](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/) .

### <a name="base-images-for-windows-insiders"></a>Images de base pour les Windows Insiders

Microsoft fournit une version «Insider» de chaque image de base du conteneur. Ces images de conteneur Insider comportent le plus récent et le plus grand du développement des fonctionnalités dans les images de conteneurs. Lorsque vous exécutez un hôte qui est une version Insider de Windows (Windows Insider ou Windows Server Insider), il est préférable d’utiliser ces images. Les images Insider sont disponibles dans le hub d’amarrage:

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

Pour en savoir plus, consultez [les conteneurs d’utilisation avec le programme Windows Insider](../deploy-containers/insider-overview.md) .

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core vs Server

`Windows Server Core` et `Nanoserver` sont les principales images de base à cibler. La principale différence entre ces images réside dans le fait que le serveur possède une surface d’API sensiblement plus petite. PowerShell, WMI et la pile de maintenance Windows sont absents de l’image de serveur.

Le serveur a été conçu pour fournir une surface d’API suffisante pour exécuter des applications qui ont une dépendance sur .NET Core ou d’autres infrastructures Open source modernes. En guise de compromis pour la surface d’APi plus petite, l’image de serveur présente un encombrement sur disque sensiblement inférieur au reste des images de base Windows. N’oubliez pas que vous pouvez ajouter autant de couches que vous le souhaitez sur NanoServer. Pour obtenir un exemple, consultez le [fichier Dockerfile NanoServer .NETCore](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).

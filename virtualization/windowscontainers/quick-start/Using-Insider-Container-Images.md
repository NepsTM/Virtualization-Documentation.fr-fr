
# <a name="using-insider-container-images"></a>Utilisation d’images de conteneur Insider

Cet exercice va vous guider tout au long du déploiement et de l’utilisation de la fonctionnalité de conteneur Windows de la dernière build Insider de WindowsServer du programme WindowsInsiderPreview. Au cours de cet exercice, vous allez installer le rôle de conteneur et déployer une version d’évaluation des images de système d’exploitation de base. Si vous voulez vous familiariser avec les conteneurs, vous trouverez des informations dans la rubrique [À propos des conteneurs](../about/index.md).

Ce démarrage rapide est spécifique aux conteneurs WindowsServer du programme WindowsServerInsiderPreview. Familiarisez-vous avec le programme avant de poursuivre ce démarrage rapide.

## <a name="prerequisites"></a>Conditions préalables:

- Être membre du [programme Windows Insider](https://insider.windows.com/GettingStarted) et avoir consulté les conditions d’utilisation.
- Utiliser un système informatique (physique ou virtuel) qui exécute la dernière version de WindowsServer du programme WindowsInsider et/ou la dernière version de Windows10 du programme WindowsInsider.

> [!IMPORTANT]
> Pour Windows, la version du système d’exploitation hôte doit correspondre à la version du système d’exploitation du conteneur. Si vous souhaitez exécuter un conteneur basé sur une nouvelle build Windows, vérifiez que vous disposez d’une build hôte équivalente. Dans le cas contraire, vous pouvez utiliser l’isolation Hyper-V pour exécuter des conteneurs plus anciens sur de nouvelles builds d’hôte. Pour plus d’informations sur la compatibilité de la version du conteneur Windows, consultez la documentation de ce conteneur.

## <a name="install-docker-enterprise-edition-ee"></a>Installer Docker Enterprise Edition (EE)

Vous devez installer Docker EE pour utiliser les conteneurs Windows. Docker EE comprend le moteur Docker et le client Docker.

Pour installer Docker EE, nous allons utiliser le module PowerShell de fournisseur OneGet. Le fournisseur active la fonctionnalité de conteneurs sur votre ordinateur et installe Docker EE. Cette opération nécessite un redémarrage. Ouvrez une session PowerShell avec élévation de privilèges, puis exécutez les commandes suivantes.

> [!NOTE]
> L’installation de docker EE avec les builds Windows Server Insider nécessite un fournisseur de services de OneGet différent de celui utilisé pour les builds non-Insider. Si Docker EE et le fournisseur OneGet DockerMsftProvider sont déjà installés, supprimez-les avant de continuer.

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

Installez le module OneGet PowerShell pour une utilisation avec les builds Windows Insider.

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

Utilisez OneGet pour installer la dernière version de Docker EE Preview.

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

Une fois l’installation terminée, redémarrez l’ordinateur.

```powershell
Restart-Computer -Force
```

## <a name="install-base-container-image"></a>Installer l’image de conteneur de base

Avant d’utiliser des conteneurs Windows, une image de base doit être installée. En tant que membre du programme WindowsInsider, vous pouvez également tester les images de base de nos builds les plus récentes. Avec les images de base Insider, il existe désormais 6 images de base disponibles basées sur Windows Server. Reportez-vous au tableau ci-dessous pour vérifier dans quel contexte les utiliser:

| Image de système d’exploitation de base                       | Utilisation                      |
|-------------------------------------|----------------------------|
| mcr.microsoft.com/windows/servercore         | Production et développement |
| mcr.microsoft.com/windows/nanoserver              | Production et développement |
| mcr.microsoft.com/windows/              | Production et développement |
| mcr.microsoft.com/windows/servercore/insider | Développement uniquement           |
| mcr.microsoft.com/windows/nanoserver/insider        | Développement uniquement           |
| mcr.microsoft.com/windows/insider        | Développement uniquement           |

Pour tirer partie de l’image de base du serveur Core Insider, reportez-vous aux balises proposées sur le [concentrateur du serveur principal référentiel Samples](https://hub.docker.com/_/microsoft-windows-servercore-insider) pour utiliser le format suivant:

```console
docker pull mcr.microsoft.com/windows/servercore/insider:10.0.{build}.{revision}
```

Pour tirer l’image de base de nano Server, reportez-vous aux balises proposées sur le hub de la [station d’accueil nano Server](https://store.docker.com/_/microsoft-windows-nanoserver-insider) Insider référentiel samples pour utiliser le format suivant:

```console
docker pull mcr.microsoft.com/windows/nanoserver/insider:10.0.{build}.{revision}
```

Pour tirer l’image de base Windows Insider, consultez la ou les balises proposées sur le [concentrateur du référentiel samples d’amarrage Windows Insider](https://store.docker.com/_/microsoft-windows-insider) pour utiliser le format suivant:

```console
docker pull mcr.microsoft.com/windows/insider:10.0.{build}.{revision}
```

> [!IMPORTANT]
> Prenez connaissance de [l’image du](../EULA.md ) système d’exploitation conteneurs Windows et des [conditions d’utilisation du](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)programme Windows Insider.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Générer et exécuter un exemple d’application](./Nano-RS3-.NET-Core-and-PS.md)

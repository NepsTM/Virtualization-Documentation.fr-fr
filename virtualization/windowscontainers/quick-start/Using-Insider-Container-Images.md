
# <a name="using-insider-container-images"></a>Utilisation d’images de conteneur Insider

Cet exercice va vous guider tout au long du déploiement et de l’utilisation de la fonctionnalité de conteneur Windows de la dernière build Insider de WindowsServer du programme WindowsInsiderPreview. Au cours de cet exercice, vous allez installer le rôle de conteneur et déployer une version d’évaluation des images de système d’exploitation de base. Si vous voulez vous familiariser avec les conteneurs, vous trouverez des informations dans la rubrique [À propos des conteneurs](../about/index.md).

Ce démarrage rapide est spécifique aux conteneurs WindowsServer du programme WindowsServerInsiderPreview. Familiarisez-vous avec le programme avant de poursuivre ce démarrage rapide.

## <a name="prerequisites"></a>Conditions préalables:

- Être membre du [programme Windows Insider](https://insider.windows.com/GettingStarted) et avoir consulté les conditions d’utilisation.
- Utiliser un système informatique (physique ou virtuel) qui exécute la dernière version de WindowsServer du programme WindowsInsider et/ou la dernière version de Windows10 du programme WindowsInsider.

> [!IMPORTANT]
> Vous devez utiliser une build de Windows Server à partir du programme Windows Server Insider Preview ou une build de Windows 10 à partir du programme Windows Insider Preview pour utiliser l’image de base décrites ci-dessous. Si vous n’utilisez pas l’une de ces builds, la création d’un conteneur échouera lors de l’utilisation de ces images de base.

## <a name="install-docker-enterprise-edition-ee"></a>Installer Docker Enterprise Edition (EE)

Vous devez installer Docker EE pour utiliser les conteneurs Windows. Docker EE comprend le moteur Docker et le client Docker.

Pour installer Docker EE, nous allons utiliser le module PowerShell de fournisseur OneGet. Le fournisseur active la fonctionnalité de conteneurs sur votre ordinateur et installe Docker EE. Cette opération nécessite un redémarrage. Ouvrez une session PowerShell avec élévation de privilèges, puis exécutez les commandes suivantes.

> [!NOTE]
> L’installation de Docker EE avec les builds Windows Server Insider nécessite un autre fournisseur OneGet que celui utilisé pour les builds non-Insider. Si Docker EE et le fournisseur OneGet DockerMsftProvider sont déjà installés, supprimez-les avant de continuer.

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

Avant d’utiliser des conteneurs Windows, une image de base doit être installée. En tant que membre du programme WindowsInsider, vous pouvez également tester les images de base de nos builds les plus récentes. Les images de base Insider comptent désormais 4images de base basées sur WindowsServer. Reportez-vous au tableau ci-dessous pour vérifier dans quel contexte les utiliser:

| Image de système d’exploitation de base                       | Utilisation                      |
|-------------------------------------|----------------------------|
| MCR.Microsoft.com/Windows/servercore         | Production et développement |
| MCR.Microsoft.com/Windows/nanoserver              | Production et développement |
| MCR.Microsoft.com/Windows/servercore/Insider | Développement uniquement           |
| MCR.Microsoft.com/Windows/nanoserver/Insider        | Développement uniquement           |

Pour extraire l’image de base Insider NanoServer, exécutez la commande suivante:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Exécutez la commande suivante pour extraire l’image de base Insider WindowsServerCore:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> Veuillez lire l' Image de système d’exploitation Windows conteneurs [CLUF](../EULA.md ) et le [Conditions d’utilisation](https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewserver)du programme Windows Insider.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Générer et exécuter un exemple d’application](./Nano-RS3-.NET-Core-and-PS.md)

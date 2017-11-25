# <a name="using-insider-container-images"></a>Utilisation d’images de conteneur Insider

Cet exercice va vous guider tout au long du déploiement et de l’utilisation de la fonctionnalité de conteneur Windows de la dernière build Insider de WindowsServer du programme WindowsInsiderPreview. Au cours de cet exercice, vous allez installer le rôle de conteneur et déployer une version d’évaluation des images de système d’exploitation de base. Si vous voulez vous familiariser avec les conteneurs, vous trouverez des informations dans la rubrique [À propos des conteneurs](../about/index.md).

Ce démarrage rapide est spécifique aux conteneurs WindowsServer du programme WindowsServerInsiderPreview. Familiarisez-vous avec le programme avant de poursuivre ce démarrage rapide.

**Conditions préalables:**

- Être membre du [programme Windows Insider](https://insider.windows.com/GettingStarted) et avoir consulté les conditions d’utilisation.
- Utiliser un système informatique (physique ou virtuel) qui exécute la dernière version de WindowsServer du programme WindowsInsider et/ou la dernière version de Windows10 du programme WindowsInsider.

>Vous devez utiliser une build de WindowsServer du programme WindowsServerInsiderPreview ou une build de Windows10 du programme WindowsInsiderPreview pour utiliser l’image de base décrite ci-dessous. Si vous n’utilisez pas l’une de ces builds, la création d’un conteneur échouera lors de l’utilisation de ces images de base.

## <a name="install-docker"></a>Installer Docker
Docker est nécessaire pour utiliser les conteneurs Windows. Docker comprend le moteur Docker et le client Docker. Vous devez également utiliser une version de Docker qui prend en charge les créations échelonnées pour une expérience optimale à l’aide de l’image NanoServer optimisée pour les conteneurs.

Pour installer Docker, nous allons utiliser le module PowerShell de fournisseur OneGet. Le fournisseur active la fonctionnalité de conteneurs sur votre ordinateur et installe Docker. Cette opération nécessite un redémarrage. Notez qu’il existe plusieurs canaux avec une version différente de Docker, selon le cas d’utilisation. Dans cet exercice, nous allons utiliser la dernière version Community Edition de Docker du canal Stable. Un canal Edge est également disponible si vous souhaitez tester les dernières évolutions dans Docker.

Ouvrez une session PowerShell avec élévation de privilèges, puis exécutez les commandes suivantes.

>Remarque: l’installation de Docker dans les builds Insider requiert un fournisseur autre que celui utilisé en temps normal à compter d’aujourd’hui. Notez la différence ci-dessous.

Installez le module PowerShell OneGet.
```powershell
Install-Module -Name DockerMsftProviderInsider -Repository PSGallery -Force
```
Utilisez OneGet pour installer la dernière version de Docker.
```powershell
Install-Package -Name docker -ProviderName DockerMsftProviderInsider -RequiredVersion 17.06.0-ce
```
Une fois l’installation terminée, redémarrez l’ordinateur.
```
Restart-Computer -Force
```

## <a name="install-base-container-image"></a>Installer l’image de conteneur de base

Avant d’utiliser des conteneurs Windows, une image de base doit être installée. En tant que membre du programme WindowsInsider, vous pouvez également tester les images de base de nos builds les plus récentes. Les images de base Insider comptent désormais 4images de base basées sur WindowsServer. Reportez-vous au tableau ci-dessous pour vérifier dans quel contexte les utiliser:

| Image de système d’exploitation de base                       | Utilisation                      |
|-------------------------------------|----------------------------|
| microsoft/windowsservercore         | Production et développement |
| microsoft/nanoserver                | Production et développement |
| microsoft/windowsservercore-insider | Développement uniquement           |
| microsoft/nanoserver-insider        | Développement uniquement           |

Pour extraire l’image de base Insider NanoServer, exécutez la commande suivante:

```
docker pull microsoft/nanoserver-insider
```

Exécutez la commande suivante pour extraire l’image de base Insider WindowsServerCore:

```
docker pull microsoft/windowsservercore-insider
```

Veuillez lire le Contrat de Licence Utilisateur Final (CLUF) applicable à l’image de système d’exploitation des conteneurs Windows, disponible ici: [CLUF](../EULA.md ) et les conditions d’utilisation du programme WindowsInsider disponibles ici: [Conditions d’utilisation](https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewserver).

## <a name="next-steps"></a>Étapes suivantes

[Créer et exécuter une application avec ou sans .NETCore2.0 ou PowerShellCore6](./Nano-RS3-.NET-Core-and-PS.md)

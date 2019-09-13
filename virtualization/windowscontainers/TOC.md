# [Conteneurs dans la documentation Windows](index.md) 

# Vue d'ensemble
## [À propos des conteneurs Windows](about/index.md)
## [Configuration requise](deploy-containers/system-requirements.md)
## [FAQ](about/faq.md)

# Prise en main
## [Configurer votre environnement](quick-start/set-up-environment.md)
## [Exécuter votre premier conteneur](quick-start/run-your-first-container.md)
## [Conteneur d’un exemple d’application](quick-start/building-sample-app.md)

# Didacticiels
## Créer un conteneur Windows
### [Écrire un Dockerfile](manage-docker/manage-windows-dockerfile.md)
### [Optimiser une Dockerfile](manage-docker/optimize-windows-dockerfile.md)
## Kubernetes sur Windows
### [Kubernetes sur Windows](kubernetes/getting-started-kubernetes-windows.md)
### [Créer un maître Kubernetes](kubernetes/creating-a-linux-master.md)
### [Choisir une solution réseau](kubernetes/network-topologies.md)
### [Rejoindre des travailleurs Windows](kubernetes/joining-windows-workers.md)
### [Rejoignez des travailleurs Linux](kubernetes/joining-linux-workers.md)
### [Déploiement de ressources Kubernetes](kubernetes/deploying-resources.md)
### [Résolution des problèmes](kubernetes/common-problems.md)
### [Services Windows sur Kubernetes](kubernetes/kube-windows-services.md)
### [Compiler des fichiers binaires Kubernetes](kubernetes/compiling-kubernetes-binaries.md)
## Service Fabric sur Windows
### [Déployer votre premier conteneur](/azure/service-fabric/service-fabric-quickstart-containers)
### [Déployer une application .NET dans un conteneur Windows](/azure/service-fabric/service-fabric-host-app-in-a-container)
## Conteneurs Linux sur Windows
### [Vue d'ensemble](deploy-containers/linux-containers.md)
### [Exécuter votre premier conteneur LCOW](quick-start/quick-start-windows-10-linux.md)

# Concepts
## Notions fondamentales sur les conteneurs Windows
### [Contrôles de ressources](manage-containers/resource-controls.md)
### [Isolation Hyper-V](manage-containers/hyperv-container.md)
### [Compatibilité des versions](deploy-containers/version-compatibility.md)
### [Images de base du conteneur](manage-containers/container-base-images.md)
## Docker
### [Moteur Docker sur Windows](manage-docker/configure-docker-daemon.md)
### [En essaim d’amarrage](manage-containers/swarm-mode.md)
### [Gestion à distance d’un hôte de station d’accueil Windows](management/manage_remotehost.md)
## Charges
### Comptes de service administrés de groupe
#### [Créer un compte gMSA](manage-containers/manage-serviceaccounts.md)
#### [Configurer votre application pour utiliser une gMSA](manage-containers/gmsa-configure-app.md)
#### [Exécuter un conteneur avec un gMSA](manage-containers/gmsa-run-container.md)
#### [Orchestrer des conteneurs avec un gMSA](manage-containers/gmsa-orchestrate-containers.md)
#### [Résoudre les problèmes de gMSAs](manage-containers/gmsa-troubleshooting.md)
### [Services d’impression](deploy-containers/print-spooler.md)
## Réseaux
### [Vue d'ensemble](container-networking/architecture.md)
### [Topologies et pilotes réseau](container-networking/network-drivers-topologies.md)
### [Isolement réseau et sécurité](container-networking/network-isolation-security.md)
### [Configurer les options de mise en réseau avancées](container-networking/advanced.md)
## Stockage
### [Vue d'ensemble](manage-containers/container-storage.md)
## Appareils
### [Périphériques matériels](deploy-containers/hardware-devices-in-containers.md)
### [Accélération GPU](deploy-containers/gpu-acceleration.md)

# Référence
## [Durée de vie des images de base](deploy-containers/base-image-lifecycle.md)
## [Optimisation de l’antivirus](https://docs.microsoft.com/windows-hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)
## [Outils de plateforme de conteneur](deploy-containers/containerd.md)
## [CLUF de l’image du conteneur](Images_EULA.md)

# Ressources
## [Exemples de conteneurs](samples.md)
## [Résolution des problèmes](troubleshooting.md)
## [Forum du conteneur](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)
## [Vidéos et blogs de la communauté](communitylinks.md)
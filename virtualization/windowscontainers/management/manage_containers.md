---
author: neilpeterson
redirect_url: ../quick_start/manage_docker
---

# Gestion des conteneurs Windows Server

<g id="1" ctype="x-strong">Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.</g>

Le cycle de vie de conteneur inclut des actions, telles que le démarrage, l’arrêt et la suppression de conteneurs. Quand vous effectuez ces actions, il est possible que vous deviez également récupérer une liste d’images de conteneur, gérer la mise en réseau des conteneurs et limiter les ressources des conteneurs. Ce document décrit en détail les tâches de gestion de conteneurs de base à l’aide de Docker et incorpore également des articles détaillés.

## Gestion des conteneurs

### Créer un conteneur

Pour créer un conteneur avec Docker, utilisez la commande <g id="2" ctype="x-code">docker run</g>.

```none
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Pour plus d’informations sur la commande docker <g id="2" ctype="x-code">run</g>, voir les <g id="4CapsExtId1" ctype="x-link"><g id="4CapsExtId2" ctype="x-linkText">informations de référence sur docker run</g><g id="4CapsExtId3" ctype="x-title"></g></g>.

### Arrête un conteneur

Pour arrêter un conteneur avec Docker, utilisez la commande <g id="2" ctype="x-code">docker stop</g>.

```none
PS C:\> docker stop tender_panini

tender_panini
```

Cet exemple arrête tous les conteneurs en cours d’exécution avec Docker.

```none
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### Supprimer un conteneur

Pour supprimer un conteneur avec Docker, utilisez la commande <g id="2" ctype="x-code">docker rm</g>.

```none
PS C:\> docker rm prickly_pike

prickly_pike
```

Pour supprimer tous les conteneurs avec Docker

```none
PS C:\> docker rm $(docker ps -aq)

dc3e282c064d
2230b0433370
```

Pour plus d’informations sur la commande docker rm, consultez les <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">informations de référence sur docker rm</g><g id="2CapsExtId3" ctype="x-title"></g></g>.






<!--HONumber=Apr16_HO5-->



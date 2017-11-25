---
title: "Compatibilité des version avec les conteneurs Windows"
description: "Comment Windows peut générer et exécuter des conteneurs dans plusieurs versions"
keywords: "métadonnées, conteneurs, version"
author: patricklang
ms.openlocfilehash: ed9d88e1e861651426e560a4531fd4added2134a
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2017
---
# <a name="windows-container-version-compatibility"></a>Compatibilité des version avec les conteneurs Windows

Windows Server2016 et la mise à jour anniversaire Windows10 (tous deux de version14393) ont été les première versions de Windows capables de générer et d’exécuter des conteneurs Windows Server. Les conteneurs créés à l’aide de ces versions peuvent s’exécuter sur des versions plus récentes, telles que Windows Server version1709, mais vous devez être au fait de certains éléments avant de commencer.

Étant donné que nous avons amélioré les fonctionnalités liées aux conteneurs Windows, nous avons dû apporter des modifications qui affectent la compatibilité. Les conteneurs plus anciens seront exécutés de la même manière sur les ordinateurs hôtes plus récents avec l’[isolation Hyper-V](../manage-containers/hyperv-container.md), et la même (ancienne) version du noyau est toujours utilisée. Toutefois, si vous souhaitez exécuter un conteneur basé sur une version plus récente de Windows, celui-ci ne pourra s’exécuter que sur la build plus récente de l’hôte.



<table>
    <tr>
    <th>Version du système d’exploitation du conteneur</th>
    <th span='2'>Version du système d’exploitation de l‘hôte</th>
    </tr>
    <tr>
        <td/>
        <td><b>Windows Server2016 / Windows101609, 1703</b><br/>Builds: 14393.*</td>
        <td><b>Windows Server version1709 / Windows10 Fall Creators Update</b><br/>Builds16299.*</td>
    </tr>
    <tr>
        <td><b>Windows Server2016 / Windows101609, 1703</b><br/>Builds: 14393.*</td>
        <td>Pris en charge. `process` ou isolation `hyperv`</td>
        <td>Pris en charge. `hyperv` isolation</td>
    </tr>
    <tr>
        <td><b>Windows Server version1709 / Windows10 Fall Creators Update</b><br/>Builds16299.*</td>
        <td>Non pris en charge</td>
        <td>Pris en charge. `process` ou isolation `hyperv`</td>
    </tr>
</table>               


## <a name="errors-from-mismatched-versions"></a>Erreurs en cas d’incompatibilité des versions

Si vous essayez d’exécuter une combinaison non pris en charge, vous obtiendrez une erreur:

```
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

Pour résoudre ce problème, vous pouvez:

- Reconstruire le conteneur en vous basant sur la version de `microsoft/nanoserver` ou `microsoft/windowsservercore`
- Si l’ordinateur hôte est plus récent, utilisez `docker run --isolation=hyperv ...`
- Procéder à l’exécution sur un ordinateur hôte différent avec la même version de Windows

## <a name="choosing-container-os-versions"></a>Choix des versions de système d’exploitation de conteneur

> Remarque: la balise «latest» sera mise à jour avec Windows Server2016, le [produit LTSC](https://docs.microsoft.com/en-us/windows-server/get-started/semi-annual-channel-overview) actuel. Si vous souhaitez obtenir les images de conteneur qui correspondent à Windows Server version1709, lisez ce qui suit.

Il est important de vous assurer de connaître la version du système d’exploitation de conteneur dont vous avez besoin. Si vous utilisez Windows Server version1709 et souhaitez obtenir les derniers correctifs qui y sont applicables, vous devez utiliser la balise «1709» lorsque vous spécifiez la version des images du conteneur du système d’exploitation de base que vous voulez, comme suit:

``` Dockerfile
FROM microsoft/windowsservercore:1709
...
```

Toutefois, si vous souhaitez obtenir un correctif spécifique pour Windows Server version1709, vous pouvez spécifier le numéro d’article de la base de connaissances dans la balise. Par exemple, si vous souhaitez obtenir l’image de conteneur du système d’exploitation de base Nano Server à partir de Windows Server version1709, avec la mise à jour KB4043961 appliquée, vous devez l’indiquer comme suit:

``` Dockerfile
FROM microsoft/nanoserver:1709_KB4043961
...
```

De même, si vous avez besoin de l’image de conteneur du système d’exploitation de base Nano Server à partir de Windows Server2016, vous pouvez toujours en obtenir la dernière version à l’aide de la balise «latest»:

``` Dockerfile
FROM microsoft/nanoserver
...
```
Vous pouvez également continuer à spécifier les correctifs exacts dont vous avez besoin grâce au schéma utilisé précédemment, en spécifiant la version du système d’exploitation dans la balise:

``` Dockerfile
FROM microsoft/nanoserver:10.0.14393.1770
...
```

## <a name="matching-versions-using-docker-swarm"></a>Correspondance des versions à l’aide de Docker Swarm

À l’heure actuelle, Docker Swarm ne propose pas de méthode intégrée pour faire correspondre la version de Windows utilisée par un conteneur à un hôte correspondant de la même version. Si le service est mis à jour pour utiliser un conteneur plus récent, il s’exécute correctement.

Si vous souhaitez exécuter plusieurs versions de Windows pendant un certain temps, deux approches peuvent être utilisées.  Configurez les ordinateurs hôtes Windows de façon à ce qu’ils utilisent toujours l’isolement Hyper-V ou utilisez des contraintes d’étiquette.

### <a name="finding-a-service-that-wont-start"></a>Recherche d’un service qui ne démarre pas

Si un service ne démarre pas, vous verrez que le `MODE` est `replicated`, mais que le nombre `REPLICAS` se retrouve bloqué sur0. Pour voir si la version du système d’exploitation est la cause du problème, utilisez les commandes suivantes:

 `docker service ls` : rechercher le nom du service

```
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

`docker service ps <name>` : obtenir l’état et les dernières tentatives.

```
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

Si vous voyez «starting container failed:...» s’affiche, vous pouvez consulter l’erreur complète avec `docker service ps --no-trunc <container name>`


```
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

Ce qui indique la même erreur `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`


### <a name="fix---update-the-service-to-use-a-matching-version"></a>Correctif: mettre à jour le service pour utiliser une version correspondante

Deux éléments sont à prendre en compte pour Docker Swarm. Si vous avez un fichier Compose disposant d’un service qui utilise une image n’ayant pas été créée par vos soins, vous devrez mettre à jour la référence en conséquence. Voir ci-dessous:

``` docker-compose
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

Vous devez également tenir compte du fait que vous avez créé ou non l’image vers laquelle vous pointez (par exemple, contoso/myimage):
``` docker-compose
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```
Dans ce cas, vous devriez utiliser la méthode décrite dans la section précédente pour modifier ce fichier dockerfile au lieu de la ligne docker-compose.

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>Atténuation: utiliser l’isolation Hyper-V avec Docker Swarm

Il existe une proposition visant à prendre en charge l’utilisation de l’isolation Hyper-V sur la base de chaque conteneur, mais le code n’est pas encore prêt. Vous pouvez suivre la progression sur [GitHub](https://github.com/moby/moby/issues/31616). Jusqu’à ce que cela soit prêt, les ordinateurs hôtes doivent être configurés de manière à toujours s’exécuter avec l’isolation Hyper-V.

Cela nécessite la modification de la configuration du service Docker, puis le redémarrage du moteur Docker.

1. Modifiez `C:\ProgramData\docker\config\daemon.json`
2. Ajouter une ligne contenant `"exec-opts":["isolation=hyperv"]`

REMARQUE: par défaut, le fichier daemon.json n’existe pas. Si, après vérification dans le répertoire, cela s’avère être effectivement le cas, vous devez créer le fichier. Vous devrez ensuite y copier le contenu suivant:

```JSON
{
    "exec-opts":["isolation=hyperv"]
}
```
Fermez et enregistrez le fichier. Redémarrer ensuite le moteur Docker:

```PowerShell
Stop-Service docker
Start-Service docker
```

Une fois le service redémarré, lancez vos conteneurs. Une fois que ceux-ci sont en cours d’exécution, vous pouvez vérifier le niveau d’isolation d’un conteneur en inspectant ce dernier:

```PowerShell
docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
```

Soit «process», soit «hyperv» sera renvoyé. Si vous avez modifié et défini votre fichier daemon.json comme décrit ci-dessus, il doit indiquer cette dernière éventualité.

### <a name="mitigation---use-labels-and-constraints"></a>Atténuation: utiliser des étiquettes et des contraintes

**Étape1: ajouter des étiquettes à chaque nœud**

Sur chaque nœud, ajoutez deux étiquettes: `OS` et `OsVersion`. Cela suppose une exécution locale, mais pouvez modifier cela afin de les définir sur un hôte distant.

```powershell
docker node update --label-add OS="windows" $ENV:COMPUTERNAME
docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
```

Par la suite, vous pouvez les occurrences de `docker node inspect`, qui afficheront les nouvelles étiquettes ajoutées

```
        "Spec": {
            "Labels": {
                "OS": "windows",
                "OsVersion": "10.0.16296"
            },
            "Role": "manager",
            "Availability": "active"
        }
```

**Étape2: ajouter une contrainte de service**

Une fois que chaque nœud a été étiqueté, nous pouvons mettre à jour les contraintes qui déterminent la sélection élective de services. Dans l’exemple ci-dessous, remplacez «contoso_service» par le nom de votre service réel:

```
docker service update \
    --constraint-add "node.labels.OS == windows" \
    --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
    contoso_service
```

Cela applique et limite l’emplacement d’exécution d’un nœud.

Pour plus d’informations sur l’utilisation de contraintes de service, consultez la [référence](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint) relative à la création de service


## <a name="matching-versions-using-kubernetes"></a>Correspondance des version à l’aide de Kubernetes

Le même problème peut se produire lorsque des pods sont planifiés dans Kubernetes. Cela peut être évité à l’aide de stratégies similaires:

- Régénérez le conteneur basé sur la même version du système d’exploitation en développement et en production (voir **Choix des versions de système d’exploitation de conteneur** ci-dessus)
- Utilisez des étiquettes de nœud et des sélecteurs de nœuds pour vous assurer que les pods sont planifiés sur des nœuds compatibles si les nœuds de Windows Server2016 et Windows Server version1709 sont dans le même cluster
- Utilisez des clusters distincts en fonction de la version du système d’exploitation


### <a name="finding-pods-failed-on-os-mismatch"></a>Échec de la recherche des pods en raison de l’incompatibilité des systèmes d’exploitation

Dans ce cas, un déploiement inclut un pod qui a été planifié sur un nœud dont la version du système d’exploitation est incompatible, et sur lequel l’isolation Hyper-V n’est pas activée. Le même message d’erreur s’affiche dans les événements répertoriés, avec `kubectl describe pod <podname>`. Après plusieurs tentatives, l’état du pod sera probablement `CrashLoopBackOff`

```
$ kubectl -n plang describe po fabrikamfiber.web-789699744-rqv6p

Name:           fabrikamfiber.web-789699744-rqv6p
Namespace:      plang
Node:           38519acs9011/10.240.0.6
Start Time:     Mon, 09 Oct 2017 19:40:30 +0000
Labels:         io.kompose.service=fabrikamfiber.web
                pod-template-hash=789699744
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-789699744","uid":"b5062a08-ad29-11e7-b16e-000d3a...
Status:         Running
IP:             10.244.3.169
Created By:     ReplicaSet/fabrikamfiber.web-789699744
Controlled By:  ReplicaSet/fabrikamfiber.web-789699744
Containers:
  fabrikamfiberweb:
    Container ID:       docker://eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a
    Image:              patricklang/fabrikamfiber.web:latest
    Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
    Port:               80/TCP
    State:              Waiting
      Reason:           CrashLoopBackOff
    Last State:         Terminated
      Reason:           ContainerCannotRun
      Message:          container eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{037b6606-bc9c-461f-ae02-829c28410798}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Layers":[{"ID":"f8bc427f-7aa3-59c6-b271-7331713e9451","Path":"C:\\ProgramData\\docker\\windowsfilter\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881"},{"ID":"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47","Path":"C:\\ProgramData\\docker\\windowsfilter\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5"},{"ID":"4f624ca7-2c6d-5c42-b73f-be6e6baf2530","Path":"C:\\ProgramData\\docker\\windowsfilter\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67"},{"ID":"88e360ff-32af-521d-9a3f-3760c12b35e2","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e"},{"ID":"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a","Path":"C:\\ProgramData\\docker\\windowsfilter\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461"},{"ID":"c2b3d728-4879-5343-a92a-b735752a4724","Path":"C:\\ProgramData\\docker\\windowsfilter\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3"},{"ID":"2973e760-dc59-5800-a3de-ab9d93be81e5","Path":"C:\\ProgramData\\docker\\windowsfilter\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75"},{"ID":"454a7d36-038c-5364-8a25-fa84091869d6","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0"},{"ID":"9b748c8c-69eb-55fb-a1c1-5688cac4efd8","Path":"C:\\ProgramData\\docker\\windowsfilter\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec"},{"ID":"bfde5c26-b33f-5424-9405-9d69c2fea4d0","Path":"C:\\ProgramData\\docker\\windowsfilter\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2"},{"ID":"bdabfbf5-80d1-57f1-86f3-448ce97e2d05","Path":"C:\\ProgramData\\docker\\windowsfilter\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf"},{"ID":"ad9b34f2-dcee-59ea-8962-b30704ae6331","Path":"C:\\ProgramData\\docker\\windowsfilter\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8"}],"HostName":"fabrikamfiber.web-789699744-rqv6p","MappedDirectories":[{"HostPath":"c:\\var\\lib\\kubelet\\pods\\b50f0027-ad29-11e7-b16e-000d3afd2878\\volumes\\kubernetes.io~secret\\default-token-rw9dn","ContainerPath":"c:\\var\\run\\secrets\\kubernetes.io\\serviceaccount","ReadOnly":true,"BandwidthMaximum":0,"IOPSMaximum":0}],"HvPartition":false,"EndpointList":null,"NetworkSharedContainerName":"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13","Servicing":false,"AllowUnqualifiedDNSQuery":false}
      Exit Code:        128
      Started:          Mon, 09 Oct 2017 20:27:08 +0000
      Finished:         Mon, 09 Oct 2017 20:27:08 +0000
    Ready:              False
    Restart Count:      10
    Environment:        <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
Conditions:
  Type          Status
  Initialized   True
  Ready         False
  PodScheduled  True
Volumes:
  default-token-rw9dn:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-rw9dn
    Optional:   false
QoS Class:      BestEffort
Node-Selectors: beta.kubernetes.io/os=windows
Tolerations:    <none>
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
  ---------     --------        -----   ----                    -------------                           --------        ------                  -------
  49m           49m             1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-789699744-rqv6p to 38519acs9011
  49m           49m             1       kubelet, 38519acs9011                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
  29m           29m             1       kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Failed to pull image "patricklang/fabrikamfiber.web:latest": rpc error: code = 2 desc = Error response from daemon: {"message":"Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io: no such host"}
  49m           3m              12      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
  33m           2m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Error: failed to start container "fabrikamfiberweb": Error response from daemon: {"message":"container fabrikamfiberweb encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {\"SystemType\":\"Container\",\"Name\":\"fabrikamfiberweb\",\"Owner\":\"docker\",\"IsDummy\":false,\"VolumePath\":\"\\\\\\\\?\\\\Volume{037b6606-bc9c-461f-ae02-829c28410798}\",\"IgnoreFlushesDuringBoot\":true,\"LayerFolderPath\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\fabrikamfiberweb\",\"Layers\":[{\"ID\":\"f8bc427f-7aa3-59c6-b271-7331713e9451\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881\"},{\"ID\":\"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5\"},{\"ID\":\"4f624ca7-2c6d-5c42-b73f-be6e6baf2530\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67\"},{\"ID\":\"88e360ff-32af-521d-9a3f-3760c12b35e2\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e\"},{\"ID\":\"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461\"},{\"ID\":\"c2b3d728-4879-5343-a92a-b735752a4724\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3\"},{\"ID\":\"2973e760-dc59-5800-a3de-ab9d93be81e5\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75\"},{\"ID\":\"454a7d36-038c-5364-8a25-fa84091869d6\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0\"},{\"ID\":\"9b748c8c-69eb-55fb-a1c1-5688cac4efd8\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec\"},{\"ID\":\"bfde5c26-b33f-5424-9405-9d69c2fea4d0\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2\"},{\"ID\":\"bdabfbf5-80d1-57f1-86f3-448ce97e2d05\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf\"},{\"ID\":\"ad9b34f2-dcee-59ea-8962-b30704ae6331\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8\"}],\"HostName\":\"fabrikamfiber.web-789699744-rqv6p\",\"MappedDirectories\":[{\"HostPath\":\"c:\\\\var\\\\lib\\\\kubelet\\\\pods\\\\b50f0027-ad29-11e7-b16e-000d3afd2878\\\\volumes\\\\kubernetes.io~secret\\\\default-token-rw9dn\",\"ContainerPath\":\"c:\\\\var\\\\run\\\\secrets\\\\kubernetes.io\\\\serviceaccount\",\"ReadOnly\":true,\"BandwidthMaximum\":0,\"IOPSMaximum\":0}],\"HvPartition\":false,\"EndpointList\":null,\"NetworkSharedContainerName\":\"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13\",\"Servicing\":false,\"AllowUnqualifiedDNSQuery\":false}"}
  33m           11s             151     kubelet, 38519acs9011                                           Warning         FailedSync              Error syncing pod
  32m           11s             139     kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         BackOff                 Back-off restarting failed container
```


### <a name="mitigation---using-node-labels--nodeselector"></a>Atténuation: utilisation d’étiquettes de nœud et de sélecteurs de nœuds

`kubectl get node` affiche une liste de tous les nœuds. Vous pouvez ensuite utiliser `kubectl describe node <nodename>` pour obtenir plus de détails. 

Dans ce cas, il existe deux nœuds Windows, qui exécutent des versions différentes:

```
$ kubectl get node

NAME                        STATUS    AGE       VERSION
38519acs9010                Ready     21h       v1.7.7-7+e79c96c8ff2d8e
38519acs9011                Ready     4h        v1.7.7-25+bc3094f1d650a2
k8s-linuxpool1-38519084-0   Ready     21h       v1.7.7
k8s-master-38519084-0       Ready     21h       v1.7.7

$ kubectl describe node 38519acs9010

Name:                   38519acs9010
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_D2_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9010
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 01:41:02 +0000

...
  
System Info:
 Machine ID:                    38519acs9010
 System UUID:
 Boot ID:
 Kernel Version:                10.0 14393 (14393.1715.amd64fre.rs1_release_inmarket.170906-1810)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
 ...
 
$ kubectl describe node 38519acs9011
Name:                   38519acs9011
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_DS1_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9011
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 18:13:25 +0000
Conditions:
...

System Info:
 Machine ID:                    38519acs9011
 System UUID:
 Boot ID:
 Kernel Version:                10.0 16299 (16299.0.amd64fre.rs3_release.170922-1354)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
...

```


1. Prenez note de chaque nom de nœud et du numéro de `Kernel Version` à partir des informations système:

Nom         | Version
-------------|--------------------------------------------------------
38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354



2. Ajoutez à chaque nœud une étiquette appelée `beta.kubernetes.io/osbuild`. Windows Server2016 requiert une correspondance avec les versions majeure et mineure (14393.1715) pour être pris en charge sans isolation Hyper-V. Windows Server version1709 ne requiert que la correspondance de la version majeure (16299).

À partir de l’exemple ci-dessus:

```
$ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


node "38519acs9010" labeled
$ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

node "38519acs9011" labeled

```

3. Vérifiez la présence des étiquettes avec `kubectl get nodes --show-labels`

```
$ kubectl get nodes --show-labels

NAME                        STATUS                     AGE       VERSION                    LABELS
38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
```

4. Ajoutez des sélecteurs de nœuds aux déploiements

Ajoutez un `nodeSelector` à la spécification de conteneur avec `beta.kubernetes.io/os` = windows et `beta.kubernetes.io/osbuild` = 14393.* ou 16299 pour assurer la correspondance avec le système d’exploitation de base utilisé par le conteneur.

Voici un exemple complet pour l’exécution d’un conteneur conçu pour Windows Server2016:

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert -f docker-compose-combined.yml
    kompose.version: 1.2.0 (99f88ef)
  creationTimestamp: null
  labels:
    io.kompose.service: fabrikamfiber.web
  name: fabrikamfiber.web
spec:
  replicas: 1
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: fabrikamfiber.web
    spec:
      containers:
      - image: patricklang/fabrikamfiber.web:latest
        name: fabrikamfiberweb
        ports:
        - containerPort: 80
        resources: {}
      restartPolicy: Always
      nodeSelector:
        "beta.kubernetes.io/os": windows
        "beta.kubernetes.io/osbuild": "14393.1715"
status: {}
```

Le pod peut désormais lancer le déploiement mis à jour. Les sélecteurs de nœuds figurent également dans `kubectl describe pod <podname>`, ce qui vous permet de vérifier qu’ils ont été ajoutés.

```
$ kubectl -n plang describe po fa

Name:           fabrikamfiber.web-1780117715-5c8vw
Namespace:      plang
Node:           38519acs9010/10.240.0.4
Start Time:     Tue, 10 Oct 2017 01:43:28 +0000
Labels:         io.kompose.service=fabrikamfiber.web
                pod-template-hash=1780117715
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-1780117715","uid":"6a07aaf3-ad5c-11e7-b16e-000d3...
Status:         Running
IP:             10.244.1.84
Created By:     ReplicaSet/fabrikamfiber.web-1780117715
Controlled By:  ReplicaSet/fabrikamfiber.web-1780117715
Containers:
  fabrikamfiberweb:
    Container ID:       docker://c94594fb53161f3821cf050d9af7546991aaafbeab41d333d9f64291327fae13
    Image:              patricklang/fabrikamfiber.web:latest
    Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
    Port:               80/TCP
    State:              Running
      Started:          Tue, 10 Oct 2017 01:43:42 +0000
    Ready:              True
    Restart Count:      0
    Environment:        <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
Conditions:
  Type          Status
  Initialized   True
  Ready         True
  PodScheduled  True
Volumes:
  default-token-rw9dn:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-rw9dn
    Optional:   false
QoS Class:      BestEffort
Node-Selectors: beta.kubernetes.io/os=windows
                beta.kubernetes.io/osbuild=14393.1715
Tolerations:    <none>
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
  ---------     --------        -----   ----                    -------------                           --------        ------                  -------
  5m            5m              1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-1780117715-5c8vw to 38519acs9010
  5m            5m              1       kubelet, 38519acs9010                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
  5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
  5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
  5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
  5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Started                 Started container
```

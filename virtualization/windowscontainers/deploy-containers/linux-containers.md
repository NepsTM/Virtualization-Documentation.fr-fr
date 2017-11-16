# <a name="linux-containers"></a>Conteneurs Linux

Cette fonctionnalité utilise l'[isolation Hyper-V](../manage-containers/hyperv-container.md) pour exécuter un noyau Linux avec juste assez du système d’exploitation pour prendre en charge des conteneurs. Les modifications apportées à Windows et Hyper-V pour y parvenir ont commencé dans la mise à jour _Windows10 Fall Creators Update_ et dans _Windows Server, version1709_, mais cela a également nécessité de travailler sur le [projet Moby](https://www.github.com/moby/moby) open source sur lequel la technologie Docker est basée, ainsi que sur le noyau Linux. 

![Vidéo d'aperçu de conteneur Linux](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

Pour tester cette fonctionnalité, vous devez disposer des éléments suivants:

- Windows10 ou Windows Server Insider Preview build16267 ou ultérieure
- Une build du démon Docker basée sur la branche maîtresse de Moby, exécutée avec l’indicateur `--experimental`
- L’image Linux compatible de votre choix

Des guides de prise en main sont disponibles pour cette version préliminaire:

- [Docker Enterprise Edition Preview](https://blog.docker.com/2017/09/docker-windows-server-1709/) comprend un système LinuxKit et une version préliminaire d’EE Docker pouvant exécuter des conteneurs Linux. Pour plus d’informations, consultez également l’article consacré à l’[Aperçu des conteneurs Linux sur Windows à l’aide de LinuxKit](https://go.microsoft.com/fwlink/?linkid=857061)
- [Exécution de conteneurs Ubuntu avec l'isolation Hyper-V sur Windows10 et Windows Server](https://go.microsoft.com/fwlink/?linkid=857067)


## <a name="work-in-progress"></a>Travail en cours

La progression du projet Moby peut être suivie sur [GitHub](https://github.com/moby/moby/issues/33850)


### <a name="known-app-issues"></a>Problèmes connus liés aux applications

Toutes ces applications nécessitent le mappage de volume, qui présente certaines limitations couvertes sous [Lier les montages](#Bind-mounts). Elle ne démarreront pas ou ne s’exécuteront pas correctement.

- Mysql
- Postgress
- Wordpress
- Jenkins
- Mariadb
- Rabbitmq


### <a name="bind-mounts"></a>Lier des montages

La liaison des volumes de montage avec `docker run -v ...` stocke les fichiers sur le système de fichiers NTFS de Windows. Par conséquent, une traduction est nécessaire pour les opérations POSIX. À l’heure actuelle, certaines opérations de système de fichiers ne sont que partiellement implémentées ou ne le sont pas du tout, ce qui peut entraîner des incompatibilités pour certaines applications.

Ces opérations ne fonctionnent actuellement pas pour les volumes montés par liaison:

- MkNod
- XAttrWalk
- XAttrCreate
- Lock
- Getlock
- Auth
- Flush
- INotify

Certaines ne sont pas totalement implémentés:

- GetAttr: le nombre Nlink est toujours signalé comme étant2
- Open: seuls les indicateurs ReadWrite, WriteOnly et ReadOnly sont implémentés
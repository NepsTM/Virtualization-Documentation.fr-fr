



# Compatibilité des applications dans les conteneurs Windows

Il s’agit d’une version préliminaire. Même si une application exécutée sur Windows doit aussi au final s’exécuter dans un conteneur, cette page permet d’examiner l’état actuel de la compatibilité des applications.

Le seul but de ce document est de partager notre expérience.

Un élément ne figure pas dans cette liste ? Faites-nous part de ce qui a fonctionné ou pas dans votre environnement sur [les forums](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).

## Conteneurs de serveur Windows

Nous avons tenté d’exécuter les applications suivantes dans un conteneur Windows Server. Ces résultats ne garantissent pas le bon fonctionnement de l’application.

| **Nom**| **Version**| **Image de base Windows Server Core**| **Image de base Nano Server**| **Commentaire**|
|:-----|:-----|:-----|:-----|:-----|
| .NET| 3.5| Oui| Unknown| |
| .NET| 4.6| Oui| Unknown| |
| CLR .NET| 5 bêta 6| Oui| Oui| Les deux, x64 et x86|
| ActivePython| 3.4.1| Oui| Oui| |
| Apache Cassandra| | Oui| Inconnu|
| Apache CouchDB| 1.6.1| Non| Non| |
| Apache Hadoop| | Oui| Non| |
| Apache HTTPD| 2.4| Oui| Oui| Le runtime VC++ n’est pas installé si le filtre de déduplication est chargé.Déchargez la déduplication avec `fltmc unload dedup`|
| Apache Tomcat| 8.0.24 x64| Oui| Unknown| |
| ASP.NET| 4.6| Oui| Inconnu| |
| ASP.NET| 5 bêta 6| Oui| Oui| Les deux, x64 et x86|
| Django| | Oui| Oui| |
| Go| 1.4.2| Oui| Oui| |
| Internet Information Service| 10.0| Oui| Oui| Le runtime VC++ n’est pas installé si le filtre de déduplication est chargé.Déchargez la déduplication avec `fltmc unload dedup`|
| Java| 1.8.0_51| Oui| Oui| Utilisez la version du serveur.La version du client ne s’installe pas correctement|
| MongoDB| 3.0.4| Oui| Inconnu| |
| MySQL| 5.6.26| Oui| Oui| |
| NGinx| 1.9.3| Oui| Oui| |
| Node.js| 0.12.6| Partiellement| Partiellement| NPM ne parvient pas à télécharger les packages.|
| Perl| | Oui| Inconnu| |
| PHP| 5.6.11| Oui| Oui| Le runtime VC++ n’est pas installé si le filtre de déduplication est chargé.Déchargez la déduplication avec `fltmc unload dedup`|
| PostgreSQL| 9.4.4| Oui| Unknown| Le runtime VC++ n’est pas installé si le filtre de déduplication est chargé.Déchargez la déduplication avec `fltmc unload dedup`|
| Python| 3.4.3| Oui| Oui| |
| R| 3.2.1| Non| Non| |
| RabbitMQ| 3.5.x| Oui| Unknown| |
| Redis| 2.8.2101| Oui| Oui| |
| Ruby| 2.2.2| Oui| Oui| Les deux, x64 et x86|
| Ruby on Rails| 4.2.3| Oui| Oui| |
| SQLite| 3.8.11.1| Oui| Non| |
| SQL Server Express| 2014 LocalDB| Non| Non| |
| Outils Sysinternals| *| Oui| Oui| Test uniquement sur celles n’exigeant pas d’interface utilisateur graphique.PsExec ne fonctionne pas dans la conception actuelle|

## Conteneurs Hyper-V

Nous avons tenté d’exécuter les applications suivantes dans un conteneur Hyper-V. Ces résultats ne garantissent pas un fonctionnement correct de l’application.

| **Nom**| **Version**| **Image de base Nano Server**| **Commentaire**|
|:-----|:-----|:-----|:-----|
| Apache Hadoop| | Non| |
| Apache HTTPD| 2.4| Oui| Le runtime VC++ n’est pas installé si le filtre de déduplication est chargé.Déchargez la déduplication avec `fltmc unload dedup`|
| ASP.NET| 5 bêta 6| Oui| Les deux, x64 et x86|
| Django| | Oui| Si l’image est créée avec un DockerFile et que des binaires Python sont copiés comme éléments, Python ne fonctionne pas.Démarrez le conteneur, puis copiez les binaires Python.|
| Go| 1.4.2| Oui| |
| Internet Information Service| 10.0| Oui| IIS n’est pas installé en utilisant directement dism.Effectuez une installation sans assistance d’IIS à l’aide des commandes dism.|
| Java| 1.8.0_51| Oui| Utilisez la version du serveur.La version du client ne s’installe pas correctement|
| MySQL| 5.6.26| Oui| |
| NGinx| 1.9.3| Oui| |
| Node.js| 0.12.6| Partiellement| NPM ne parvient pas à télécharger les packages.|
| Python| 3.4.3| Oui| Si l’image est créée avec un DockerFile et que des binaires Python sont copiés comme éléments, Python ne fonctionne pas.Démarrez le conteneur, puis copiez les binaires Python.|
| Redis| 2.8.2101| Oui| |
| Ruby| 2.2.2| Oui| Les deux, x64 et x86|
| Ruby on Rails| 4.2.3| Oui| |
| Outils Sysinternals| | Oui| Test uniquement sur celles n’exigeant pas d’interface utilisateur graphique.PsExec ne fonctionne pas dans la conception actuelle.|

## Parlez-nous de votre expérience

Un élément ne figure pas dans cette liste ? Faites-nous part de ce qui a fonctionné ou pas dans votre environnement sur [les forums](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).




<!--HONumber=Feb16_HO3-->

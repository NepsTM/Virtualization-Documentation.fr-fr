



# Interopérabilité de gestion

**Il s’agit d’un contenu préliminaire qui peut faire l’objet de modifications.**

Dans l’ensemble, les conteneurs Windows créés avec PowerShell doivent être gérés avec PowerShell et ceux créés avec Docker doivent être gérés avec Docker. Ceci dit, le module PowerShell Host Computing offre la possibilité de détecter et d’arrêter les conteneurs **en cours d’exécution**, quelle que soit la façon dont ils ont été créés. Ce module se comporte comme un « gestionnaire des tâches » pour les conteneurs en cours d’exécution sur un hôte de conteneur.

## Afficher tous les conteneurs

Pour retourner une liste de conteneurs, utilisez la commande `Get-ComputeProcess`

```powershell
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## Arrêter un conteneur

Pour arrêter un conteneur, qu’il ait été créé avec PowerShell ou Docker, utilisez la commande `Stop-ComputeProcess`.

> Au moment de la rédaction, le service VMMS devra être redémarré pour que les conteneurs s’affichent comme étant arrêtés lors de l’utilisation de la commande `Get-Container`.

```powershell
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```






<!--HONumber=Feb16_HO3-->



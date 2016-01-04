# Gérer Windows avec PowerShell Direct

PowerShell Direct permet de gérer à distance une machine virtuelle Windows 10 ou Windows Server Technical Preview à partir d’un hôte Hyper-V Windows 10 ou Windows Server Technical. Grâce à PowerShell Direct, vous pouvez gérer PowerShell dans une machine virtuelle, indépendamment de la configuration du réseau ou des paramètres de gestion à distance sur l’hôte Hyper-V ou la machine virtuelle. Pour les administrateurs Hyper-V, cela facilite la génération de scripts et l’automatisation de la gestion et de la configuration des machines virtuelles.

Vous pouvez exécuter PowerShell Direct de deux manières :
* En tant que session interactive : [accédez à cette section](vmsession.md#create-and-exit-an-interactive-powershell-session) pour créer et quitter une session PowerShell Direct à l’aide d’applets de commande PSSession.
* En exécutant un ensemble de commandes ou un script : [accédez à cette section](vmsession.md#run-a-script-or-command-with-invoke-command) pour exécuter un script ou une commande avec l’applet de commande Invoke-Command.


## Spécifications

**Configuration requise pour le système d’exploitation :**
* Le système d’exploitation hôte doit exécuter Windows 10, Windows Server Technical Preview ou une version ultérieure.
* La machine virtuelle doit exécuter Windows 10, Windows Server Technical Preview ou une version ultérieure.

Si vous gérez des anciennes machines virtuelles, utilisez Connexion à une machine virtuelle (VMConnect) ou [configurez un réseau virtuel pour la machine virtuelle](http://technet.microsoft.com/library/cc816585.aspx).

Pour créer une session PowerShell Direct sur une machine virtuelle :
* la machine virtuelle doit être démarrée et s’exécuter localement sur l’hôte ;
* vous devez être connecté à l’ordinateur hôte en tant qu’administrateur Hyper-V ;
* vous devez fournir des informations d’identification utilisateur valides pour la machine virtuelle.

## Créer et quitter une session PowerShell interactive

1. Sur l’hôte Hyper-V, ouvrez Windows PowerShell en tant qu’administrateur.

3. Exécutez l’une des commandes suivantes pour créer une session en utilisant le nom ou le GUID de la machine virtuelle :
``` PowerShell
Enter-PSSession -VMName <VMName>
Enter-PSSession -VMGUID <VMGUID>
```

4. Exécutez toutes les commandes nécessaires. Ces commandes s’exécutent sur la machine virtuelle à l’aide de laquelle vous avez créé la session.
5. Une fois terminé, exécutez la commande suivante pour fermer la session :
``` PowerShell
Exit-PSSession 
```

> Remarque : si votre session ne se connecte pas, vérifiez que vous utilisez des informations d’identification propres à la machine virtuelle à laquelle vous vous connectez, et non à l’hôte Hyper-V.

Pour en savoir plus sur ces applets de commande, voir [Enter-PSSession](http://technet.microsoft.com/library/hh849707.aspx) et [Exit-PSSession](http://technet.microsoft.com/library/hh849743.aspx).

## Exécuter un script ou une commande avec Invoke-Command

Vous pouvez utiliser l’applet de commande [Invoke-Command](http://technet.microsoft.com/library/hh849719.aspx) pour exécuter un ensemble prédéfini de commandes sur la machine virtuelle. L’exemple suivant illustre l’utilisation de l’applet de commande Invoke-Command, où PSTest est le nom de la machine virtuelle et où le script à exécuter (foo.ps1) se trouve dans le dossier script sur le lecteur C:/ :

 ``` PowerShell
 Invoke-Command -VMName PSTest -FilePath C:\script\foo.ps1 
 ```

Pour exécuter une commande unique, utilisez le paramètre **-ScriptBlock** :

 ``` PowerShell
 Invoke-Command -VMName PSTest -ScriptBlock { cmdlet } 
 ```

## Résolution des problèmes

Les messages d’erreur liés à l’utilisation de PowerShell Direct sont peu nombreux. Voici les plus courants, les causes possibles et les outils à utiliser pour diagnostiquer les problèmes.

### Erreur : une session à distance a peut-être pris fin.

Message d’erreur :
```
An error has occured which Windows PowerShell cannot handle.  A remote session might have ended. 
```

Causes possibles :
* La machine virtuelle n’est pas en cours d’exécution.
* Le système d’exploitation invité ne prend pas en charge PowerShell Direct (voir la [configuration requise](#Requirements)).
* PowerShell n’est pas encore disponible dans l’invité.
    * Le système d’exploitation n’est pas entièrement démarré.
    * Le système d’exploitation ne peut pas démarrer correctement.
    * Un événement de démarrage exige une entrée de l’utilisateur.
* Impossible de valider les informations d’identification de l’invité.

Vous pouvez utiliser l’applet de commande [Get-VM](http://technet.microsoft.com/library/hh848479.aspx) pour vérifier que les informations d’identification que vous utilisez ont le rôle d’administrateur Hyper-V et pour identifier les machines virtuelles démarrées et exécutées localement sur l’hôte.

## Exemples

Pour obtenir des exemples, voir [GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93).

Pour obtenir de nombreux exemples d’utilisation de PowerShell Direct dans votre environnement, ainsi que des trucs et astuces pour écrire des scripts Hyper-V avec PowerShell, voir [Extraits de code PowerShell Direct](../develop/powershell_snippets.md).




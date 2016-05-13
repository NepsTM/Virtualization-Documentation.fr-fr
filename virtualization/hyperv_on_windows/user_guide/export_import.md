# Exporter et importer des machines virtuelles

Vous pouvez utiliser les fonctionnalités d’exportation et d’importation de Hyper-V pour dupliquer rapidement des machines virtuelles. Vous pouvez utiliser des machines virtuelles exportées pour la sauvegarde, ou pour déplacer une machine virtuelle entre des hôtes Hyper-V.

L’importation vous permet de restaurer des machines virtuelles. Vous n’avez pas besoin d’exporter une machine virtuelle pour pouvoir l’importer. L’importation tente de recréer la machine virtuelle à partir de tout ce qui est disponible. Utilisez l’Assistant **Machine virtuelle** pour spécifier l’emplacement des fichiers. Cette opération inscrit la machine virtuelle auprès d’Hyper-V et la rend disponible pour utilisation.

Ce document explique pas à pas comment exporter et importer une machine virtuelle, et décrit quelques-unes des options à votre disposition pour exécuter ces tâches.

## Exporter une machine virtuelle

### À l’aide du Gestionnaire Hyper-V

Quand vous créez une exportation d’une machine virtuelle, tous les fichiers associés sont regroupés dans l’exportation. Il s’agit notamment des fichiers de configuration, des fichiers de disque dur et des éventuels fichiers de point de contrôle existants. Pour créer une exportation de machine virtuelle :

1. Dans le Gestionnaire Hyper-V, cliquez avec le bouton droit sur la machine virtuelle désirée, puis sélectionnez **Exporter**.

2. Spécifiez l’emplacement dans lequel stocker les fichiers exportés, puis cliquez sur le bouton **Exporter**.

**Remarque :** vous pouvez exécuter ce processus sur une machine virtuelle dont l’état est démarré ou arrêté.

Une fois l’exportation terminée, tous les fichiers exportés apparaissent sous l’emplacement de l’exportation.

### À l’aide de PowerShell

Pour exporter une machine virtuelle à l’aide de PowerShell, utilisez la commande **Export-VM**.

```powershell
Export-VM -Name <vm name> -Path <path>
```

Pour plus d’informations sur l’utilisation de Windows PowerShell pour exporter des machines virtuelles, consultez [Export-VM](https://technet.microsoft.com/library/hh848491.aspx).

## Importer une machine virtuelle

Lors de l’importation d’une machine virtuelle, celle-ci est inscrite auprès de l’hôte Hyper-V. Une exportation de machine virtuelle peut être réimportée dans l’hôte à partir duquel elle a été dérivée ou dans un nouvel hôte.

Hyper-V propose trois types d’importation :

- **Inscrire sur place** : les fichiers d’exportation ont été placés dans l’emplacement à partir duquel la machine virtuelle doit être exécutée. Lors de l’importation, l’ID de la machine virtuelle est le même qu’au moment de l’exportation. Pour cette raison, si la machine virtuelle est déjà inscrite auprès d’Hyper-V, elle doit être supprimée pour que l’importation fonctionne. Une fois l’importation terminée, les fichiers d’exportation deviennent les fichiers d’état en cours d’exécution et ne peuvent pas être supprimés.

- **Restaurer la machine virtuelle** : vous pouvez soit stocker les fichiers de machine virtuelle dans un emplacement spécifique, soit utiliser les emplacements par défaut d’Hyper-V. Ce type d’importation crée une copie du fichier exporté et le déplace à l’endroit sélectionné. Lors de l’importation, l’ID de la machine virtuelle est le même qu’au moment de l’exportation. Pour cette raison, si la machine virtuelle est déjà exécutée dans d’Hyper-V, elle doit être supprimée pour que l’importation puisse se terminer. Une fois l’importation terminée, les fichiers exportés restent intacts et peuvent être supprimés et/ou réimportés.

- **Copier la machine virtuelle** : ce type d’importation est semblable au type de restauration en ce sens que vous sélectionnez un emplacement pour les fichiers de machine virtuelle. La différence tient au fait que la machine virtuelle se voit attribuer un nouvel ID unique lors de l’importation. Vous pouvez donc importer plusieurs fois la machine virtuelle dans le même hôte.


### À l’aide du Gestionnaire Hyper-V

Pour importer une machine virtuelle dans un hôte Hyper-V :

1. Dans le menu Actions du Gestionnaire Hyper-V, cliquez sur **Importer une machine virtuelle**.

2. Dans l’écran Avant de commencer, cliquez sur **Suivant**.

3. Sélectionnez le dossier contenant les fichiers exportés, puis cliquez sur **Suivant**.

4. Sélectionnez la machine virtuelle à importer (il est probable qu’une seule option vous soit proposée).

5. Choisissez l’un des trois types d’importation, puis cliquez sur Suivant.

6. Dans l’écran de synthèse, sélectionnez **Terminer**.

L’Assistant Importation d’ordinateur virtuel vous guide également dans les étapes de résolution des incompatibilités quand vous importez la machine virtuelle vers le nouvel hôte. Cet Assistant peut donc vous aider pour la configuration associée au matériel physique, comme la mémoire, les commutateurs virtuels et les processeurs virtuels.

Pour importer une machine virtuelle, l’Assistant effectue les opérations suivantes :
1. Crée une copie du fichier de configuration de machine virtuelle. Il s’agit d’une précaution au cas où un redémarrage inattendu se produit sur l’hôte, par exemple à la suite d’une panne de courant.

2. Valide le matériel. Les informations contenues dans le fichier de configuration de machine virtuelle sont comparées au matériel sur le nouvel hôte.

3. Compile une liste d’erreurs. Cette liste indique les éléments à reconfigurer et détermine les pages qui s’affichent ensuite dans l’Assistant.

4. Affiche les pages concernées, une catégorie à la fois. L’Assistant explique chaque incompatibilité pour vous aider à reconfigurer la machine virtuelle afin qu’elle soit compatible avec le nouvel hôte.

5. Supprime la copie du fichier de configuration. Une fois que l'Assistant a effectué cette opération, l'ordinateur virtuel est prêt à démarrer.


### À l’aide de PowerShell

Pour importer une machine virtuelle à l’aide PowerShell, utilisez la commande **Import-VM**. Les commandes PowerShell suivantes illustrent les trois types d’importation.

Pour effectuer une importation sur place d’une machine virtuelle, utilisez une commande semblable à celle-ci. Souvenez-vous que l’importation sur place utilise les fichiers à l’endroit où ils sont stockés au moment de l’importation et qu’elle conserve l’ID de la machine virtuelle.

```powershell
Import-VM -Path 'C:\<emport path>\2B91FEB3-F1E0-4FFF-B8BE-29CED892A95A.vmcx' 
```

Pour importer la machine virtuelle en spécifiant le chemin de ses fichiers, utilisez une commande semblable à celle-ci.

```powershell
Import-VM -Path ‘C:\<vm export path>\2B91FEB3-F1E0-4FFF-B8BE-29CED892A95A.vmcx' -Copy -VhdDestinationPath 'D:\Virtual Machines\WIN10DOC' -VirtualMachinePath 'D:\Virtual Machines\WIN10DOC'
```

Pour effectuer une importation de copie et déplacer les fichiers de la machine virtuelle à l’emplacement Hyper-V par défaut, utilisez une commande semblable à celle-ci.

``` PowerShell
Import-VM -Path 'C:\<vm export path>\2B91FEB3-F1E0-4FFF-B8BE-29CED892A95A.vmcx' -Copy -GenerateNewId
```

Pour plus d’informations, consultez [Import-VM](https://technet.microsoft.com/library/hh848495.aspx).





<!--HONumber=Feb16_HO4-->



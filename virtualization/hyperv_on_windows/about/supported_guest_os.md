# Invités Windows pris en charge

Cet article répertorie les combinaisons de systèmes d’exploitation prises en charge dans Hyper-V sur Windows. Il présente également les services d’intégration et d’autres facteurs de prise en charge.

## Que signifie la prise en charge ?

La prise en charge signifie que Microsoft a testé ces combinaisons hôte/invité. Les problèmes liés à ces combinaisons peuvent être traités par les services PSS (Product Support Services).

Microsoft assure la prise en charge des systèmes d’exploitation invités de la façon suivante :
* Les problèmes détectés dans les services d’intégration et systèmes d’exploitation Microsoft sont pris en charge par le support technique Microsoft.
* En ce qui concerne les problèmes dans d’autres systèmes d’exploitation ayant été certifiés par le fournisseur de système d’exploitation comme fonctionnant sur Hyper-V, la prise en charge est assurée par le fournisseur.
* En ce qui concerne les problèmes détectés dans d’autres systèmes d’exploitation, Microsoft soumet le problème à la communauté de support multifournisseur, [TASNet](http://www.tsanet.org/).

Pour pouvoir être pris en charge, l’hôte Hyper-V et l’invité doivent être mis à jour avec toutes les mises à jour critiques disponibles dans Windows Update.

## Systèmes d’exploitation invités pris en charge

Pour bénéficier de la prise en charge, les systèmes d’exploitation invités Windows et le système d’exploitation hôte doivent être à jour avec toutes les mises à jour critiques disponibles dans Windows Update.

| Systèmes d’exploitation invités| Nombre maximal de processeurs virtuels| Remarques|
|:-----|:-----|:-----|
| Windows 10| 32| |
| Windows 8.1| 32| |
| Windows 8| 32| |
| Windows 7 avec Service Pack 1 (SP1)| 4| Édition Intégrale, Édition Entreprise et Édition Professionnel (32 bits et 64 bits).|
| Windows 7| 4| Édition Intégrale, Édition Entreprise et Édition Professionnel (32 bits et 64 bits).|
| Windows Vista avec Service Pack 2 (SP2)| 2| Professionnel, Entreprise et Édition Intégrale, notamment les éditions N et KN.|
| -| | |
| Windows Server 2012 R2| 64| |
| Windows Server 2012| 64| |
| Windows Server 2008 R2 avec Service Pack 1 (SP1)| 64| Éditions Datacenter, Entreprise, Standard et Web.|
| Windows Server 2008 avec Service Pack 2 (SP2)| 4| Éditions Datacenter, Entreprise, Standard et Web (32 bits et 64 bits).|
| Windows Home Server 2011| 4| |
| Windows Small Business Server 2011| Édition Essentials - 2, édition Standard - 4| |

> Windows 10 peut s’exécuter en tant que système d’exploitation invité sur les hôtes Hyper-V Windows 8.1 et Windows Server 2012 R2.

## Linux et FreeBSD pris en charge

| Systèmes d’exploitation invités| |
|:-----|:------|
| [CentOS et Red Hat Enterprise Linux ](https://technet.microsoft.com/library/dn531026.aspx)| |
| [Machines virtuelles Debian sur Hyper-V](https://technet.microsoft.com/library/dn614985.aspx)| |
| [SUSE](https://technet.microsoft.com/en-us/library/dn531027.aspx)| |
| [Oracle Linux](https://technet.microsoft.com/en-us/library/dn609828.aspx)| |
| [Ubuntu](https://technet.microsoft.com/en-us/library/dn531029.aspx)| |
| [FreeBSD](https://technet.microsoft.com/library/dn848318.aspx)| |

Pour plus d’informations, notamment concernant la prise en charge sur des versions précédentes d’Hyper-V, voir [Machines virtuelles Linux et FreeBSD sur Hyper-V](https://technet.microsoft.com/library/dn531030.aspx).





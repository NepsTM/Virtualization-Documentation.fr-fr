# <a name="known-issues-for-insider-builds"></a>Problèmes connus pour les builds Insider

## <a name="build-16237"></a>Build16237

- L’isolement Hyper-V ne fonctionne pas correctement. Cette solution est nécessaire pour utiliser l’isolation Hyper-V dans la version 16237. Exécutez les commandes suivantes dans PowerShell:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server est désormais exécuté en tant qu’utilisateur, de sorte que les commandes qui nécessitent des privilèges d’administrateur échoueront. L’inclusion d’une ligne telle que «RUN setx /M PATH» entraînera l’échec de la build. Pour ce scénario, vous pouvez utiliser cette solution:

```dockerfile
RUN setx PATH <path>
```

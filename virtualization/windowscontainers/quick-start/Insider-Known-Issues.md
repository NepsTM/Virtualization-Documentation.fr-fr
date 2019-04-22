# <a name="known-issues-for-insider-builds"></a>Problèmes connus pour les builds insider

## <a name="build-16237"></a>Build16237

- Isolation Hyper-V ne fonctionne pas correctement. Cette solution de contournement est nécessaire pour utiliser l’isolation Hyper-V dans la build 16237. Exécutez les commandes suivantes dans PowerShell:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- NANO Server s’exécute désormais en tant qu’utilisateur, par conséquent, les commandes qui nécessitent des privilèges d’administrateur. L’inclusion d’une ligne telle que «RUN setx /M PATH» entraînera l’échec de la build. Pour ce scénario, vous pouvez utiliser cette solution:

```dockerfile
RUN setx PATH <path>
```

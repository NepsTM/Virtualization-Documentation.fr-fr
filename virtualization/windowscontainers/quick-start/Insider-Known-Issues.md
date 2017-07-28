# Problèmes connus pour les builds Insider

## Build16237

- Les conteneurs Hyper-V ne fonctionnent pas correctement. Cette solution de contournement permet d’utiliser les conteneurs Hyper-V dans la build16237. Exécutez les commandes suivantes dans PowerShell:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- NanoServer s’exécute désormais en tant qu’utilisateur et, par conséquent, les commandes qui nécessitent des droits d’administrateur échoueront. L’inclusion d’une ligne telle que «RUN setx /M PATH» entraînera l’échec de la build. Pour ce scénario, vous pouvez utiliser cette solution:

```dockerfile
RUN setx PATH <path>
```

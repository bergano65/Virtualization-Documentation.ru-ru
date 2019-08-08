# <a name="known-issues-for-insider-builds"></a>Известные проблемы сборок для участников программы предварительной оценки

## <a name="build-16237"></a>Сборка 16237

- Изоляция Hyper-V не работает должным образом. Это решение необходимо для использования изоляции Hyper-V в сборке 16237. Выполните следующие команды в PowerShell:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server теперь запускается от имени пользователя, поэтому команды, требующие прав администратора, завершатся сбоем. Если добавить такую строку, как «RUN setx /M PATH», построение завершится ошибкой. В этом сценарии можно использовать этот вариант:

```dockerfile
RUN setx PATH <path>
```

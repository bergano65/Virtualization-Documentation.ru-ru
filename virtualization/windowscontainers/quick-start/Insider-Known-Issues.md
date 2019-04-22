# <a name="known-issues-for-insider-builds"></a>Известные проблемы для сборок

## <a name="build-16237"></a>Сборка 16237

- Изоляция Hyper-V не работает надлежащим образом. Использование изоляции Hyper-V в сборке 16237 требуется следующим образом. Выполните следующие команды в PowerShell:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server теперь работает как пользователь, поэтому команды, требующие привилегий администратора завершится ошибкой. Если добавить такую строку, как «RUN setx /M PATH», построение завершится ошибкой. В этом сценарии можно использовать этот вариант:

```dockerfile
RUN setx PATH <path>
```

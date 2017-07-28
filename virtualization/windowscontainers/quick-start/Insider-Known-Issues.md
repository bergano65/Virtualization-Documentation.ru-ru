# Известные проблемы в сборках для участников программы предварительной оценки

## Сборка 16237

- Контейнеры Hyper-V не работает надлежащим образом. Чтобы работать с контейнерами Hyper-V в сборке 16237, используйте это временное решение. Выполните следующие команды в PowerShell:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server теперь работает как пользователь, поэтому команды, для которых необходимы права администратора, не будут выполнены. Если добавить такую строку, как «RUN setx /M PATH», построение завершится ошибкой. В этом сценарии можно использовать этот вариант:

```dockerfile
RUN setx PATH <path>
```

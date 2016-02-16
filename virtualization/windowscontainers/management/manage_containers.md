# Управление контейнерами Windows Server

**Это предварительное содержимое. Возможны изменения.**

Жизненный цикл контейнера включает такие действия, как запуск, остановка и удаление. Кроме того, необходимо получать список образов контейнера, управлять его сетевыми подключениями и ограничивать его ресурсы. В этой статье подробно рассматриваются базовые задачи по управлению контейнерами с помощью PowerShell.

Сведения об управлении контейнерами Windows с помощью Docker см. в документе Docker [Работа с контейнерами](https://docs.docker.com/userguide/usingdocker/).

## PowerShell

### Создание контейнера

При создании контейнера требуется указать имя образа контейнера, который будет служить его базой. Его можно узнать с помощью команды `Get-ContainerImage`.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version         IsOSImage
----              ---------    -------         ---------
NanoServer        CN=Microsoft 10.0.10584.1000 True
WindowsServerCore CN=Microsoft 10.0.10584.1000 True
```

Чтобы создать контейнер, используйте команду `New-Container`. Также контейнеру можно задать имя NetBIOS с помощью параметра `-ContainerComputerName`.

```powershell
PS C:\> New-Container -ContainerImageName WindowsServerCore -Name demo -ContainerComputerName demo

Name State Uptime   ParentImageName
---- ----- ------   ---------------
demo  Off   00:00:00 WindowsServerCore
```

После создания контейнера добавьте к нему сетевой адаптер.

```powershell
PS C:\> Add-ContainerNetworkAdapter -ContainerName TST
```

Чтобы подключить сетевой адаптер контейнера к виртуальному коммутатору, требуется указать имя коммутатора. Используйте команду `Get-VMSwitch`, чтобы отобразить список виртуальных коммутаторов.

```powershell
PS C:\> Get-VMSwitch

Name SwitchType NetAdapterInterfaceDescription
---- ---------- ------------------------------
DHCP External   Microsoft Hyper-V Network Adapter
NAT  NAT
```

Подключите сетевой адаптер к виртуальному коммутатору с помощью команды `Connect-ContainerNetworkAdapter`. **ПРИМЕЧАНИЕ.** Это действие можно также выполнить при создании контейнера, используя параметр –SwitchName.

```powershell
PS C:\> Connect-ContainerNetworkAdapter -ContainerName TST -SwitchName NAT
```

### Запуск контейнера

Чтобы запустить контейнер, необходимо перечислить объект PowerShell, представляющий этот контейнер. Для этого поместите выходные данные команды `Get-Container` в переменную PowerShell.

```powershell
PS C:\> $container = Get-Container -Name TST
```

Затем эти данные можно использовать в команде `Start-Container` для запуска контейнера.

```powershell
PS C:\> Start-Container $container
```

Следующий сценарий запускает все контейнеры на узле.

```powershell
PS C:\> Get-Container | Start-Container
```

### Подключение к контейнеру

Для подключения к контейнеру можно использовать приложение PowerShell Direct. Это может быть удобно, если необходимо вручную выполнить задачу, например установить программное обеспечение, запустить процессы или устранить неполадки с контейнером. Так как используется приложение PowerShell Direct, сеанс PowerShell с контейнером можно создать независимо от конфигурации сети. Дополнительные сведения о PowerShell Direct см. в [блоге PowerShell Direct](http://blogs.technet.com/b/virtualization/archive/2015/05/14/powershell-direct-running-powershell-inside-a-virtual-machine-from-the-hyper-v-host.aspx)

Чтобы создать интерактивный сеанс связи с контейнером, используйте команду `Enter-PSSession`.

 ```powershell
PS C:\> Enter-PSSession -ContainerName TST –RunAsAdministrator
 ```

Обратите внимание, что после создания удаленного сеанса PowerShell имя контейнера отобразится в командной строке оболочки.

```powershell
[TST]: PS C:\>
```

Команды также можно выполнять, не создавая постоянного сеанса PowerShell. Для этого используйте команду `Invoke-Command`.

В следующем примере в контейнере создается папка с именем "Приложение".

```powershell

PS C:\> Invoke-Command -ContainerName TST -ScriptBlock {New-Item -ItemType Directory -Path c:\application }

Directory: C:\
Mode                LastWriteTime         Length Name                                                 PSComputerName
----                -------------         ------ ----                                                 --------------
d-----       10/28/2015   3:31 PM                application                                          TST
```

### Остановка контейнера

Чтобы остановить контейнер, потребуется объект PowerShell, представляющий этот контейнер. Для этого поместите выходные данные команды `Get-Container` в переменную PowerShell.

```powershell
PS C:\> $container = Get-Container -Name TST
```

Затем эти данные можно использовать в команде `Stop-Container` для остановки контейнера.

```powershell
PS C:\> Stop-Container $container
```

Следующий сценарий остановит все контейнеры на узле.

```powershell
PS C:\> Get-Container | Stop-Container
```

### Удаление контейнера

Если контейнер больше не нужен, его можно удалить. Чтобы удалить контейнер, он должен быть остановлен, и нужно создать объект PowerShell, представляющий этот контейнер.

```powershell
PS C:\> $container = Get-Container -Name TST
```

Чтобы удалить контейнер, используйте команду `Remove-Container`.

```powershell
PS C:\> Remove-Container $container -Force
```

Следующий сценарий удалит все контейнеры на узле.

```powershell
PS C:\> Get-Container | Remove-Container -Force
```

## Docker

### Создание контейнера

Используйте команду `docker run`, чтобы создать контейнер с помощью Docker.

```powershell
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Дополнительные сведения о команде Docker run см. в [справке по команде Docker run} (https://docs.docker.com/engine/reference/run/).

### Остановка контейнера

Используйте команду `docker stop`, чтобы остановить контейнер с помощью Docker.

```powershell
PS C:\> docker stop tender_panini

tender_panini
```

В этом примере показано, как остановить все запущенные контейнеры с помощью Docker.

```powershell
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### Удаление контейнера

Чтобы удалить контейнер с помощью Docker, используйте команду `docker rm`.

```powershell
PS C:\> docker rm prickly_pike

prickly_pike
```

Чтобы удалить все контейнеры с помощью Docker:

```powershell
PS C:\> docker rm $(docker ps -a -q)

dc3e282c064d
2230b0433370
```

Дополнительные сведения о команде Docker rm см. в [справке по команде Docker rm](https://docs.docker.com/engine/reference/commandline/rm/).




<!--HONumber=Feb16_HO1-->

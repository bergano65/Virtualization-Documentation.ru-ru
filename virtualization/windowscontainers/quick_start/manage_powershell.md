# Краткое руководство по использованию PowerShell в работе с контейнерами Windows

Благодаря контейнерам Windows можно быстро развернуть много изолированных приложений на одном компьютере. В этом кратком руководстве показано, как с помощью PowerShell развертывать контейнеры Hyper-V и Windows Server, а также управлять ими. Выполнив упражнения из этой статьи, вы с нуля разработаете очень простое приложение "Hello world", работающее в контейнерах Windows Server и Hyper-V. При этом вы создадите образы контейнеров, поработаете с общими папками контейнеров и научитесь управлять жизненным циклом контейнеров. В результате вы получите общее представление о развертывании контейнеров Windows и управлении ими.

В этом пошаговом руководстве подробно описаны контейнеры Windows Server и Hyper-V. Для каждого типа контейнеров предусмотрены базовые требования. В документацию по контейнерам Windows входит описание быстрого развертывания узла контейнера. Это самый простой способ сразу начать работать с контейнерами Windows. Если у вас еще нет узла контейнера, см. [краткое руководство по развертыванию узла контейнера](./container_setup.md).

Указанные ниже элементы обязательны для каждого упражнения из этой статьи.

**Контейнеры Windows Server:**

- Узел контейнера Windows под управлением Windows Server 2016 Core в локальной среде или Azure.

**Контейнеры Hyper-V:**

- Узел контейнера Windows, включенный с помощью вложенной виртуализации.
- Windows Server 2016 Media — [Загрузить](https://aka.ms/tp4/serveriso).

>Microsoft Azure не поддерживает контейнеры Hyper-V. Для выполнения упражнения с контейнером Hyper-V требуется локальный узел контейнера.

## Контейнер Windows Server

Контейнеры Windows Server представляют собой изолированную и переносимую операционную среду с возможностью учета ресурсов, в которой можно запускать приложения и размещать процессы. Контейнеры Windows Server обеспечивают изоляцию между контейнером и узлом, а также между контейнерами на узле благодаря изоляции процессов и пространств имен.

### Создание контейнера

При работе с TP4 для контейнеров Windows Server под управлением Windows Server 2016 или Windows Server 2016 Core требуется образ ОС Windows Server 2016 Core.

Запустите сеанс PowerShell, введя команду `powershell`.

```powershell
C:\> powershell
Windows PowerShell
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\>
```

Чтобы проверить, установлен ли образ операционной системы Windows Server Core, используйте команду `Get-ContainerImage`. Может появиться несколько образов ОС. Это нормально.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

Чтобы создать контейнер Windows Server, используйте команду `New-Container`. В следующем примере показано создание контейнера `TP4Demo` на основе образа ОС с именем `WindowsServerCore`, а также подключение этого контейнера к коммутатору виртуальной машины с именем `Virtual Switch`.

```powershell
PS C:\> New-Container -Name TP4Demo -ContainerImageName WindowsServerCore -SwitchName "Virtual Switch"

Name    State Uptime   ParentImageName
----    ----- ------   ---------------
TP4Demo Off   00:00:00 WindowsServerCore
```

Чтобы просмотреть список существующих контейнеров, используйте команду `Get-Container`.

```powershell
PS C:\> Get-Container

Name    State Uptime   ParentImageName
----    ----- ------   ---------------
TP4Demo Off   00:00:00 WindowsServerCore
```

Запустите контейнер с помощью команды `Start-Container`.

```powershell
PS C:\> Start-Container -Name TP4Demo
```

Подключитесь к контейнеру с помощью команды `Enter-PSSession`. Обратите внимание, что при создании сеанса PowerShell с контейнером изменится запрос PowerShell: он будет отражать имя контейнера.

```powershell
PS C:\> Enter-PSSession -ContainerName TP4Demo -RunAsAdministrator

[TP4Demo]: PS C:\Windows\system32>
```

### Создание образа IIS

Теперь можно изменить контейнер и записать эти изменения для создания нового образа контейнера. В этом примере установлены службы IIS.

Чтобы установить роль IIS в данном контейнере, используйте команду `Install-WindowsFeature`.

```powershell
[TP4Demo]: PS C:\> Install-WindowsFeature web-server

Success Restart Needed Exit Code      Feature Result
------- -------------- ---------      --------------
True    No             Success        {Common HTTP Features, Default Document, D...
```

После завершения установки IIS выйдите из контейнера, введя команду `exit`. Это действие возвращает сеанс PowerShell к сеансу с узлом контейнера.

```powershell
[TP4Demo]: PS C:\> exit
PS C:\>
```

Наконец, остановите работу контейнера с помощью команды `Stop-Container`.

```powershell
PS C:\> Stop-Container -Name TP4Demo
```

Теперь состояние этого контейнера можно записать в новый образ контейнера. Сделайте это с помощью команды `New-ContainerImage`.

В этом примере создается образ контейнера, для которого указано имя `WindowsServerCoreIIS`, издатель `Demo` и версия `1.0`.

```powershell
PS C:\> New-ContainerImage -ContainerName TP4Demo -Name WindowsServerCoreIIS -Publisher Demo -Version 1.0

Name                 Publisher Version IsOSImage
----                 --------- ------- ---------
WindowsServerCoreIIS CN=Demo   1.0.0.0 False
```

Теперь, когда контейнер записан в новый образ, он больше не нужен. Его можно удалить с помощью команды `Remove-Container`.

```powershell
PS C:\> Remove-Container -Name TP4Demo -Force
```


### Создание контейнера IIS

На этот раз создайте контейнер, используя образ контейнера `WindowsServerCoreIIS`.

```powershell
PS C:\> New-Container -Name IIS -ContainerImageName WindowsServerCoreIIS -SwitchName "Virtual Switch"

Name State Uptime   ParentImageName
---- ----- ------   ---------------
IIS  Off   00:00:00 WindowsServerCoreIIS
```
Запустите контейнер.

```powershell
PS C:\> Start-Container -Name IIS
```

### Настройка сетевых подключений

Конфигурация сети по умолчанию для быстрого начала работы контейнеров Windows предусматривает подключение контейнеров к виртуальному коммутатору, настроенному с использованием преобразования сетевых адресов (NAT). По этой причине для подключения к приложению, работающему внутри контейнера, порт на узле контейнера необходимо сопоставить с портом на контейнере. Подробные сведения о сетевых подключениях контейнеров см. в разделе [Сетевые подключения контейнеров](../management/container_networking.md).

В этом упражнении веб-сайт размещается в службах IIS, которые выполняются внутри контейнера. Чтобы получить доступ к веб-сайту через порт 80, сопоставьте порт 80 для IP-адреса узла контейнера с портом 80 для IP-адреса контейнера.

Чтобы вернуть IP-адрес контейнера, выполните приведенную ниже команду.

```powershell
PS C:\> Invoke-Command -ContainerName IIS {ipconfig}

Windows IP Configuration


Ethernet adapter vEthernet (Virtual Switch-7570F6B1-E1CA-41F1-B47D-F3CA73121654-0):

   Connection-specific DNS Suffix  . : DNS
   Link-local IPv6 Address . . . . . : fe80::ed23:c1c6:310a:5c10%16
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

Чтобы создать правило NAT для сопоставления портов, используйте команду `Add-NetNatStaticMapping`. В примере ниже показано, как проверить наличие правила сопоставления портов и создать это правило, если оно не существует. Обратите внимание, что значение `-InternalIPAddress` должно соответствовать IP-адресу данного контейнера.

```powershell
if (!(Get-NetNatStaticMapping | where {$_.ExternalPort -eq 80})) {
Add-NetNatStaticMapping -NatName "ContainerNat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
}
```

Когда порты будут сопоставлены, необходимо также настроить правило брандмауэра для входящих подключений в отношении настроенного порта. Чтобы сделать это для порта 80, запустите указанный ниже сценарий. Обратите внимание, что если создать правило NAT для внешнего порта, отличного от порта 80, понадобится соответствующее правило брандмауэра.

```powershell
if (!(Get-NetFirewallRule | where {$_.Name -eq "TCP80"})) {
    New-NetFirewallRule -Name "TCP80" -DisplayName "HTTP on TCP/80" -Protocol tcp -LocalPort 80 -Action Allow -Enabled True
}
```

Если вы работаете в Azure и еще не создали группу сетевой безопасности, необходимо создать ее. Дополнительные сведения о группах сетевой безопасности см. в статье [Что такое группа сетевой безопасности](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-nsg/).

### Создание приложения

Теперь, когда контейнер создан из образа IIS и сетевые подключения настроены, откройте браузер и введите IP-адрес узла контейнера. Должен появиться экран-заставка IIS.

![](media/iis1.png)

Проверив, работают ли экземпляры служб IIS, можно создать приложение "Hello, World!" и разместить его в экземпляре IIS. Для этого создайте сеанс PowerShell с контейнером.

```powershell
PS C:\> Enter-PSSession -ContainerName IIS -RunAsAdministrator
[IIS]: PS C:\Windows\system32>
```

Выполните приведенную ниже команду для удаления экрана-заставки IIS.

```powershell
[IIS]: PS C:\> del C:\inetpub\wwwroot\iisstart.htm
```
Выполните приведенную ниже команду, чтобы заменить сайт IIS по умолчанию новым статическим сайтом.

```powershell
[IIS]: PS C:\> "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

Снова перейдите к IP-адресу узла контейнера. Вы увидите приложение "Hello, World!". Обратите внимание, что для просмотра обновленного приложения может потребоваться прервать все подключения через браузер или очистить кэш браузера.

![](media/HWWINServer.png)

Завершите удаленный сеанс контейнера.

```powershell
[IIS]: PS C:\> exit
PS C:\>
```

### Удаление контейнера

Чтобы удалить контейнер, его работу нужно остановить.

```powershell
PS C:\> Stop-Container -Name IIS
```

Когда работа контейнера остановлена, его можно удалить с помощью команды `Remove-Container`.

```powershell
PS C:\> Remove-Container -Name IIS -Force
```

Наконец, образ контейнера можно удалить с помощью команды `Remove-ContainerImage`.

```powershell
PS C:\> Remove-ContainerImage -Name WindowsServerCoreIIS -Force
```

## Контейнер Hyper-V

Контейнеры Hyper-V обеспечивают дополнительный уровень изоляции по сравнению с контейнерами Windows Server. Каждый контейнер Hyper-V создается в высокооптимизированной виртуальной машине. В отличие от контейнера Windows Server, использующего ядро совместно с узлом контейнера и всеми остальными контейнерами Windows Server на этом узле, контейнер Hyper-V полностью изолирован от других контейнеров. Создание контейнеров Hyper-V и управление ими выполняется так же, как и в случае с контейнерами Windows Server. Дополнительные сведения о контейнерах Hyper-V см. в статье [Управление контейнерами Hyper-V](../management/hyperv_container.md).

>Microsoft Azure не поддерживает контейнеры Hyper-V. Для выполнения упражнений с контейнером Hyper-V требуется локальный узел контейнера.

### Создание контейнера

При работе с TP4 контейнеры Hyper-V должны использовать образ ОС Nano Server Core. Команда `Get-ContainerImage` позволит проверить, установлен ли образ ОС Nano Server.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

Чтобы создать контейнер Hyper-V, используйте команду `New-Container`, указав среду выполнения для HyperV.

```powershell
PS C:\> New-Container -Name HYPV -ContainerImageName NanoServer -SwitchName "Virtual Switch" -RuntimeType HyperV

Name State Uptime   ParentImageName
---- ----- ------   ---------------
HYPV Off   00:00:00 NanoServer
```

**Не запускайте** этот контейнер после его создания.

### Создание общей папки

Общие папки позволяют контейнеру получить доступ к каталогу в узле контейнера. При создании общей папки все файлы, помещенные в нее, доступны в контейнере. Общая папка используется в этом примере для копирования пакетов IIS для Nano Server в контейнер. Эти пакеты затем будут использоваться для установки служб IIS. Дополнительные сведения об общих папках см. в статье [Общие папки контейнеров](../management/manage_data.md).

Создайте на узле контейнера каталог с именем `c:\share\en-us`.

```powershell
S C:\> New-Item -Type Directory c:\share\en-us

    Directory: C:\share

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:27 PM                en-us
```

Используйте команду `Add-ContainerSharedFolder`, чтобы создать в новом контейнере общую папку.

>При создании общей папки контейнер должен быть в режиме остановки.

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName HYPV -SourcePath c:\share -DestinationPath c:\iisinstall

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
HYPV          c:\share   c:\iisinstall   ReadWrite
```

После создания общей папки запустите контейнер.

```powershell
PS C:\> Start-Container -Name HYPV
```
Создайте удаленный сеанс PowerShell с этим контейнером, выполнив команду `Enter-PSSession`.

```powershell
PS C:\> Enter-PSSession -ContainerName HYPV -RunAsAdministrator
[HYPV]: PS C:\windows\system32\config\systemprofile\Documents>cd /
```
Обратите внимание, что во время удаленного сеанса будет создана пустая общая папка `c:\iisinstall\en-us`.

```powershell
[HYPV]: PS C:\> ls c:\iisinstall

    Directory: C:\iisinstall

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:27 PM                en-us
```

### Создание образа IIS

Так как контейнер работает под управлением образа ОС Nano Server, чтобы установить службы IIS, потребуются пакеты IIS для Nano Server. Их можно найти в каталоге `NanoServer\Packages` на установочном носителе Windows Sever 2016 TP4.

Скопируйте файл `Microsoft-NanoServer-IIS-Package.cab` из каталога `NanoServer\Packages` в папку `c:\share` на данном узле контейнера.

Скопируйте файл `NanoServer\Packages\en-us\Microsoft-NanoServer-IIS-Package.cab` в папку `c:\share\en-us` на данном узле контейнера.

Создайте файл unattend.xml в папке "c:\share" и скопируйте в этот файл приведенный ниже текст.

```powershell
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <servicing>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" />
            <source location="c:\iisinstall\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="en-US" />
            <source location="c:\iisinstall\en-us\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
    </servicing>
</unattend>
```

После завершения этой процедуры каталог `c:\share` на узле контейнера должен быть настроен следующим образом.

```
c:\share
|-- en-us
|    |-- Microsoft-NanoServer-IIS-Package.cab
|
|-- Microsoft-NanoServer-IIS-Package.cab
|-- unattend.xml
```

Вернитесь в удаленный сеанс на контейнере. Обратите внимание, что пакеты IIS и файл unattended.xml будут отображаться в каталоге "c:\iisinstall".

```powershell
[HYPV]: PS C:\> ls c:\iisinstall

    Directory: C:\iisinstall

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:32 PM                en-us
-a----       10/29/2015  11:51 PM        1922047 Microsoft-NanoServer-IIS-Package.cab
-a----       11/18/2015   5:31 PM            789 unattend.xml
```

Для установки служб IIS выполните приведенную ниже команду.

```powershell
[HYPV]: PS C:\> dism /online /apply-unattend:c:\iisinstall\unattend.xml

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0


[                           1.0%                           ]

[=====                      10.1%                          ]

[=====                      10.3%                          ]

[===============            26.2%                          ]
```

После установки служб IIS запустите их вручную с помощью приведенной ниже команды.

```powershell
[HYPV]: PS C:\> Net start w3svc
The World Wide Web Publishing Service service is starting.
The World Wide Web Publishing Service service was started successfully.
```

Завершите сеанс контейнера.

```powershell
[HYPV]: PS C:\> exit
```

Остановите работу контейнера.

```powershell
PS C:\> Stop-Container -Name HYPV
```

Теперь состояние этого контейнера можно записать в новый образ контейнера.

В этом примере создается образ контейнера, для которого указано имя `NanoServerIIS`, издатель `Demo` и версия `1.0`.

```powershell
PS C:\> New-ContainerImage -ContainerName HYPV -Name NanoServerIIS -Publisher Demo -Version 1.0

Name          Publisher Version IsOSImage
----          --------- ------- ---------
NanoServerIIS CN=Demo   1.0.0.0 False
```

### Создание контейнера IIS

Создайте контейнер Hyper-V на основе образа IIS с помощью команды `New-Container`.

```powershell
PS C:\> New-Container -Name IISApp -ContainerImageName NanoServerIIS -SwitchName "Virtual Switch" -RuntimeType HyperV

Name   State Uptime   ParentImageName
----   ----- ------   ---------------
IISApp Off   00:00:00 NanoServerIIS
```

Запустите контейнер.

```powershell
PS C:\> Start-Container -Name IISApp
```

### Настройка сетевых подключений

Конфигурация сети по умолчанию для быстрого начала работы с контейнером Windows предусматривает подключение контейнеров к виртуальному коммутатору, настроенному с использованием преобразования сетевых адресов (NAT). По этой причине для подключения к приложению, работающему внутри контейнера, порт на узле контейнера необходимо сопоставить с портом на контейнере.

В этом упражнении веб-сайт размещается в службах IIS, которые выполняются внутри контейнера. Чтобы получить доступ к веб-сайту через порт 80, сопоставьте порт 80 для IP-адреса узла контейнера с портом 80 для IP-адреса контейнера.

Чтобы вернуть IP-адрес контейнера, выполните приведенную ниже команду.

```powershell
PS C:\> Invoke-Command -ContainerName IISApp {ipconfig}

Windows IP Configuration


Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : DNS
   Link-local IPv6 Address . . . . . : fe80::c574:5a5e:d5f5:18a0%4
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

Чтобы создать правило NAT для сопоставления портов, используйте команду `Add-NetNatStaticMapping`. В примере ниже показано, как проверить наличие правила сопоставления портов и создать это правило, если оно не существует. Обратите внимание, что значение `-InternalIPAddress` должно соответствовать IP-адресу данного контейнера.

```powershell
if (!(Get-NetNatStaticMapping | where {$_.ExternalPort -eq 80})) {
Add-NetNatStaticMapping -NatName "ContainerNat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
}
```
Кроме того, нужно открыть порт 80 на узле контейнера. Обратите внимание, что если создать правило NAT для внешнего порта, отличного от порта 80, понадобится соответствующее правило брандмауэра.

```powershell
if (!(Get-NetFirewallRule | where {$_.Name -eq "TCP80"})) {
    New-NetFirewallRule -Name "TCP80" -DisplayName "HTTP on TCP/80" -Protocol tcp -LocalPort 80 -Action Allow -Enabled True
}
```

### Создание приложения

Теперь, когда контейнер создан на основе образа IIS и сетевые подключения настроены, откройте браузер и введите IP-адрес узла контейнера. Появится экран-заставка IIS.

![](media/iis1.png)

Проверив, работают ли экземпляры служб IIS, можно создать приложение "Hello, World!" и разместить его в экземпляре IIS. Для этого создайте сеанс PowerShell с контейнером.

```powershell
PS C:\> Enter-PSSession -ContainerName IISApp -RunAsAdministrator
[IISApp]: PS C:\windows\system32\config\systemprofile\Documents>
```

Выполните приведенную ниже команду для удаления экрана-заставки IIS.

```powershell
[IIS]: PS C:\> del C:\inetpub\wwwroot\iisstart.htm
```
Выполните приведенную ниже команду, чтобы заменить сайт IIS по умолчанию новым статическим сайтом.

```powershell
[IISApp]: PS C:\> "Hello World From a Hyper-V Container" > C:\inetpub\wwwroot\index.html
```

Снова перейдите к IP-адресу узла контейнера. Вы увидите приложение "Hello, World!". Обратите внимание, что для просмотра обновленного приложения может потребоваться прервать все подключения через браузер или очистить кэш браузера.

![](media/HWWINServer.png)

Завершите удаленный сеанс контейнера.

```powershell
exit
```




<!--HONumber=Jan16_HO1-->

# Образы контейнеров

**Это предварительное содержимое. Возможны изменения.**

Образы контейнеров используются для развертывания контейнеров. Эти образы могут содержать операционную систему, приложения и все зависимости приложений. Например, можно создать образ контейнера, который предварительно настроен с использованием Nano Server, IIS и приложения, работающего в IIS. Этот образ контейнера можно хранить в реестре контейнеров для создания нового образа контейнера на его основе или для дальнейшего использования. При этом выполняется его развертывание на любом узле контейнера Windows (локально, в облаке или даже в службе контейнеров).

Существует два типа образов контейнеров.

- Базовые образы ОС, которые предоставляет корпорация Майкрософт. Они включают в себя основные компоненты ОС.
- Образы контейнеров, созданные на основе базовых образов ОС.

## PowerShell

### Образы списков

Выполните команду `get-containerImage`, чтобы вывести список образов на узле контейнера. Типы образов контейнеров отличаются в зависимости от свойства `IsOSImage`.

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### Установка базовых образов ОС

Образы ОС контейнеров можно найти и установить с помощью модуля ContainerProvider в PowerShell. Чтобы использовать этот модуль, его необходимо установить. Для установки модуля можно использовать приведенные ниже команды.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

Чтобы вернуть список образов из диспетчера пакетов OneGet в PowerShell:
```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

Чтобы скачать и установить базовый образ Nano Server, выполните приведенную ниже команду.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

Эта команда также позволяет скачать и установить базовый образ Windows Server Core.

> **Проблема.** Командлеты Save-ContainerImage и Install-ContainerImage не работают с образом контейнера WindowsServerCore в сеансе удаленного взаимодействия PowerShell. **Обходной путь.** Войдите на этот компьютер с помощью удаленного рабочего стола и используйте непосредственно командлет Save-ContainerImage.

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0

Downloaded in 0 hours, 2 minutes, 28 seconds.
```

Убедитесь, что образы были установлены, с помощью команды `Get-ContainerImage`.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```
Дополнительные сведения об управлении образами контейнеров см. в статье [Образы контейнеров Windows](../management/manage_images.md).

### Создание нового образа

```powershell
PS C:\> New-ContainerImage -Container $container -Publisher Demo -Name DemoImage -Version 1.0
```

### Удаление образа

Образы контейнеров невозможно удалить, если какой-либо контейнер, даже в режиме остановки, зависит от образа.

Удалите один образ с помощью PowerShell.

```powershell
PS C:\> Get-ContainerImage -Name newimage | Remove-ContainerImage -Force
```

### Зависимость от образа

Если создать один образ на основе другого, новый образ будет зависеть от исходного. Эту зависимость можно увидеть, выполнив команду `get-containerimage`. Если родительский образ не выводится, это значит, что используется базовый образ ОС.

```powershell
PS C:\> Get-ContainerImage | select Name, ParentImage

Name              ParentImage
----              -----------
NanoServerIIS     ContainerImage (Name = 'NanoServer') [Publisher = 'CN=Microsoft', Version = '10.0.10586.0']
NanoServer
WindowsServerCore
```

## Docker

### Образы списков

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### Создание нового образа

```powershell
C:\> docker commit 475059caef8f windowsservercoreiis

ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### Удаление образа

Образы контейнеров невозможно удалить, если какой-либо контейнер, даже в режиме остановки, зависит от образа.

При удалении образа с помощью Docker можно указать его идентификатор или имя.

```powershell
C:\> docker rmi windowsservercoreiis

Untagged: windowsservercoreiis:latest
Deleted: ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### Docker Hub

Реестр Docker Hub содержит предварительно созданные образы, которые можно скачать на узел контейнера. Скачав эти образы, вы сможете использовать их как основу для приложений контейнера Windows.

Чтобы просмотреть список образов, доступных в Docker Hub, выполните команду `docker search`. Примечание. Прежде чем извлекать образы, зависимые от Windows Server Core, из Docker Hub, необходимо установить базовый образ Windows Server Core.

```powershell
C:\> docker search *

NAME                    DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/aspnet        ASP.NET 5 framework installed in a Windows...   1         [OK]       [OK]
microsoft/django        Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35      .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/golang        Go Programming Language installed in a Win...   1                    [OK]
microsoft/httpd         Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis           Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/mongodb       MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/mysql         MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/nginx         Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/node          Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/php           PHP running on Internet Information Servic...   1                    [OK]
microsoft/python        Python installed in a Windows Server Core ...   1                    [OK]
microsoft/rails         Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/redis         Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/ruby          Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sqlite        SQLite installed in a Windows Server Core ...   1                    [OK]
```

Чтобы загрузить образ из Docker Hub, используйте команду `docker pull`.

```powershell
C:\> docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

Этот образ отобразится при выполнении команды `docker images`.

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```

### Зависимость от образа

Чтобы просмотреть зависимости образов с помощью Docker, можно использовать команду `docker history`.

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```




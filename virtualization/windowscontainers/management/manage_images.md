---
author: neilpeterson
---

# Образы контейнеров

**Это предварительное содержимое. Возможны изменения.**

Образы контейнеров используются для развертывания контейнеров. Эти образы могут содержать операционную систему, приложения и все зависимости приложений. Например, можно создать образ контейнера, который предварительно настроен с использованием Nano Server, IIS и приложения, работающего в IIS. Этот образ контейнера можно хранить в реестре контейнеров для создания нового образа контейнера на его основе или для дальнейшего использования. При этом выполняется его развертывание на любом узле контейнера Windows (локально, в облаке или даже в службе контейнеров).

Существует два типа образов контейнеров.

- **Базовые образы ОС** предоставляются корпорацией Майкрософт и включают в себя основные компоненты ОС.
- **Образы контейнеров** — пользовательские образы, созданные на основе базовых образов ОС.

## Базовые образы ОС

### Установка образа

Образы ОС контейнеров можно найти и установить для управления PowerShell и Docker с помощью модуля ContainerProvider PowerShell. Чтобы использовать этот модуль, его необходимо установить. Для установки модуля можно использовать приведенную ниже команду.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

После установки можно вернуть список базовых образов ОС с помощью команды `Find-ContainerImage`.

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

Чтобы скачать и установить базовый образ Nano Server, выполните приведенную ниже команду. Параметр `-version` не является обязательным. Если не указана базовая версия образа ОС, будет установлена последняя версия.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

Эта команда также позволяет скачать и установить базовый образ ОС Windows Server Core. Параметр `-version` не является обязательным. Если не указана базовая версия образа ОС, будет установлена последняя версия.

> **Проблема.** Командлеты Save-ContainerImage и Install-ContainerImage могут не работать с образом контейнера WindowsServerCore в сеансе удаленного взаимодействия PowerShell. **Обходной путь.** Войдите на этот компьютер с помощью удаленного рабочего стола и напрямую используйте командлет Save-ContainerImage.

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0

Downloaded in 0 hours, 2 minutes, 28 seconds.
```

С помощью команды `Get-ContainerImage` убедитесь, что эти образы были установлены.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

> **Install-ContainerImage** устанавливает базовый образ ОС для использования в контейнерах, управляемых PowerShell или Docker. Если базовый образ ОС скачивается, но не отображается при запуске `образа Docker`, перезапустите службу Docker с помощью приложения панели управления службами или с помощью команд sc docker stop и sc docker start.

### Автономная установка

Базовые образы ОС также можно установить без подключения к Интернету. Для этого образы будут скачаны на компьютер с подключением к Интернету, скопированы в целевую систему, а затем импортированы с помощью команды `Install-ContainerOSImages`.

Перед скачиванием базового образа ОС подготовьте систему с поставщиком образа контейнера, запустив следующую команду.

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

Чтобы скачать образ, используйте команду `Save-ContainerImage`.

```powershell
PS C:\> Save-ContainerImage -Name NanoServer -Destination c:\container-image\NanoServer.wim
```

После этого скачанный образ контейнера можно скопировать в другой узел контейнера и установить с помощью команды `Install-ContainerOSImage`.

```powershell
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### Образы тегов

При ссылке на образ контейнера по имени подсистема Docker будет искать последнюю версию образа. Если последнюю версию невозможно определить, появится следующая ошибка.

```powershell
PS C:\> docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

После установки базовых образов ОС Windows Server Core или Nano Server их потребуется пометить тегами с версией "последняя". Для этого используйте команду `docker tag`.

Дополнительные сведения о `docker tag` см. в разделе [Теги, отправка и извлечение образов в docker.com](https://docs.docker.com/mac/step_six/).

```powershell
PS C:\> docker tag <image id> windowsservercore:latest
```

При назначении тегов в выходных данных `образов Docker` отобразятся две версии одного образа: с тегом версии образа и с тегом "последняя". Теперь на образ можно ссылаться по имени.

```powershell
PS C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14289.1000     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14289.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### Удаление образа ОС

Базовые образы ОС можно удалить с помощью команды `Uninstall-ContainerOSImage`. В следующем примере будет удален базовый образ ОС NanoServer.

```powershell
Get-ContainerImage -Name NanoServer | Uninstall-ContainerOSImage
```

## Образы контейнеров PowerShell

### Образы списков

Выполните команду `Get-ContainerImage`, чтобы вывести список образов на узле контейнера. Типы образов контейнеров отличаются в зависимости от свойства `IsOSImage`.

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### Создание образа

Образ контейнера можно создать из любого существующего контейнера. Для этого используйте команду `New-ContainerImage`.

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

Если создать один образ на основе другого, новый образ будет зависеть от исходного. Эту зависимость можно увидеть, выполнив команду `Get-ContainerImage`. Если родительский образ не выводится, это значит, что используется базовый образ ОС.

```powershell
PS C:\> Get-ContainerImage | select Name, ParentImage

Name              ParentImage
----              -----------
NanoServerIIS     ContainerImage (Name = 'NanoServer') [Publisher = 'CN=Microsoft', Version = '10.0.10586.0']
NanoServer
WindowsServerCore
```

### Перемещение репозитория образов

При создании образа контейнера с помощью команды `New-ContainerImage` он сохранится в расположении по умолчанию C:\ProgramData\Microsoft\Windows\Hyper-V\Container Image Store. Этот репозиторий можно переместить с помощью команды `Move-ContainerImageRepository`. Например, следующая команда создаст репозиторий образов контейнеров в расположении c:\container-images.

```powershell
Move-ContainerImageRepository -Path c:\container-images
```
> Путь, используемый с командой `Move-ContainerImageRepository`, не должен существовать при запуске команды.

## Образы контейнеров Docker

### Образы списков

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### Создание образа

Образ контейнера можно создать из любого существующего контейнера. Для этого используйте команду `docker commit`. В следующем примере создается образ контейнера с именем windowsservercoreiis.

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

### Зависимость от образа

Чтобы просмотреть зависимости образов с помощью Docker, можно использовать команду `docker history`.

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker Hub

Реестр Docker Hub содержит предварительно созданные образы, которые можно скачать на узел контейнера. Скачав эти образы, вы сможете использовать их как основу для приложений контейнера Windows.

Чтобы просмотреть список образов, доступных в Docker Hub, выполните команду `docker search`. Примечание. Прежде чем извлекать зависимые образы контейнеров из концентратора Docker, необходимо установить базовые образы Windows Server Core или Nano Server.

> Образы, которые начинаются с "nano-", зависят от базового образа ОС Nano Server.

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
microsoft/nano-golang   Go Programming Language installed in a Nan...   1                    [OK]
microsoft/nano-httpd    Apache httpd installed in a Nano Server ba...   1                    [OK]
microsoft/nano-iis      Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/nano-mysql    MySQL installed in a Nano Server based con...   1                    [OK]
microsoft/nano-nginx    Nginx installed in a Nano Server based con...   1                    [OK]
microsoft/nano-node     Node installed in a Nano Server based cont...   1                    [OK]
microsoft/nano-python   Python installed in a Nano Server based co...   1                    [OK]
microsoft/nano-rails    Ruby on Rails installed in a Nano Server b...   1                    [OK]
microsoft/nano-redis    Redis installed in a Nano Server based con...   1                    [OK]
microsoft/nano-ruby     Ruby installed in a Nano Server based cont...   1                    [OK]
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






<!--HONumber=Mar16_HO3-->



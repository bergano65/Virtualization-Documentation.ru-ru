---
title: Развертывание Docker в Windows
description: Развертывание Docker в Windows
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bdfa4545-2291-4827-8165-2d6c98d72d37
---

# Docker и Windows

**Это предварительное содержимое. Возможны изменения.** 

Подсистема Docker не входит в состав Windows, потому ее нужно установить и настроить отдельно. Действия по запуску подсистемы Docker в Windows отличаются от аналогичных действий в Linux. Этот документ описывает установку и настройку подсистемы Docker в Windows Server 2016, Nano Server и клиенте Windows. Обратите внимание, что интерфейс командной строки и подсистема Docker недавно были разбиты на два файла. Этот документ содержит инструкции по установке их обоих.

Дополнительные сведения о Docker и наборе инструментов Docker см. на сайте [Docker.com](https://www.docker.com/). 

> Прежде чем использовать Docker для создания контейнеров Windows, необходимо включить компонент контейнеров Windows. Соответствующие инструкции см. в [руководстве по развертыванию узлов контейнера](./docker_windows.md).

## Windows Server 2016

### Установка управляющей программы Docker <!--1-->

Скачайте файл dockerd.exe по адресу `https://aka.ms/tp5/dockerd` и поместите его в каталог System32 на узле контейнера.

```none
wget https://aka.ms/tp5/dockerd -OutFile $env:SystemRoot\system32\dockerd.exe
```

Создайте каталог с именем `c:\programdata\docker`. В нем создайте файл с именем `runDockerDaemon.cmd`.

```none
New-Item -ItemType File -Path C:\ProgramData\Docker\runDockerDaemon.cmd -Force
```

Скопируйте в файл `runDockerDaemon.cmd` следующий текст:

```none
@echo off
set certs=%ProgramData%\docker\certs.d

if exist %ProgramData%\docker (goto :run)
mkdir %ProgramData%\docker

:run
if exist %certs%\server-cert.pem (if exist %ProgramData%\docker\tag.txt (goto :secure))

if not exist %systemroot%\system32\dockerd.exe (goto :legacy)

dockerd -H npipe:// 
goto :eof

:legacy
docker daemon -H npipe:// 
goto :eof

:secure
if not exist %systemroot%\system32\dockerd.exe (goto :legacysecure)
dockerd -H npipe:// -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
goto :eof

:legacysecure
docker daemon -H npipe:// -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
```
Скачайте программу nssm.exe по адресу [https://nssm.cc/release/nssm-2.24.zip](https://nssm.cc/release/nssm-2.24.zip).

```none
wget https://nssm.cc/release/nssm-2.24.zip -OutFile $env:ALLUSERSPROFILE\nssm.zip
```

Извлеките содержимое пакета.

```none
Expand-Archive -Path $env:ALLUSERSPROFILE\nssm.zip $env:ALLUSERSPROFILE
```

Скопируйте `nssm-2.24\win64\nssm.exe` в каталог `c:\windows\system32`.

```none
Copy-Item $env:ALLUSERSPROFILE\nssm-2.24\win64\nssm.exe $env:SystemRoot\system32
```
Выполните команду `nssm install`, чтобы настроить службу Docker.

```none
start-process nssm install
```

Введите следующие данные в соответствующие поля в установщике службы NSSM.

Вкладка "Приложение":

**Путь:** C:\Windows\System32\cmd.exe

**Каталог автозагрузки:** C:\Windows\System32

**Аргументы:** /s /c C:\ProgramData\docker\runDockerDaemon.cmd < nul

**Имя службы** - Docker

![](media/nssm1.png)

Вкладка "Подробности":

**Отображаемое имя:** Docker

**Описание:** управляющая программа Docker позволяет управлять контейнерами с помощью клиентов Docker.

![](media/nssm2.png)

Вкладка "Ввод-вывод":

**Вывод (stdout):** C:\ProgramData\docker\daemon.log

**Ошибка (stderr):** C:\ProgramData\docker\daemon.log

![](media/nssm3.png)

По окончании нажмите кнопку `Install Service`.

Теперь управляющая программа Docker настроена как служба Windows.

### Брандмауэр <!--1-->

Если вы хотите использовать удаленное управление Docker, необходимо также открыть TCP-порт 2376.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

### Удаление Docker <!--1-->

Следующая команда удаляет службу Docker.

```none
sc.exe delete Docker
```

### Установка Docker CLI

Скачайте файл docker.exe по адресу `https://aka.ms/tp5/docker` и поместите его в каталог System32 на узле контейнера или в любой другой системе, где вы собираетесь выполнять команды Docker.

```none
wget https://aka.ms/tp5/docker -OutFile $env:SystemRoot\system32\docker.exe
```

## Сервер Nano Server

### Установка Docker <!--2-->

Скачайте файл dockerd.exe по адресу `https://aka.ms/tp5/dockerd` и скопируйте его в папку `windows\system32` на узле контейнера Nano Server.

Создайте каталог с именем `c:\programdata\docker`. В нем создайте файл с именем `runDockerDaemon.cmd`.

```none
New-Item -ItemType File -Path C:\ProgramData\Docker\runDockerDaemon.cmd -Force
```

Скопируйте в файл `runDockerDaemon.cmd` следующий текст:

```none
@echo off
set certs=%ProgramData%\docker\certs.d

if exist %ProgramData%\docker (goto :run)
mkdir %ProgramData%\docker

:run
if exist %certs%\server-cert.pem (if exist %ProgramData%\docker\tag.txt (goto :secure))

if not exist %systemroot%\system32\dockerd.exe (goto :legacy)

dockerd -H npipe:// 
goto :eof

:legacy
docker daemon -H npipe:// 
goto :eof

:secure
if not exist %systemroot%\system32\dockerd.exe (goto :legacysecure)
dockerd -H npipe:// -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
goto :eof

:legacysecure
docker daemon -H npipe:// -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
```

Следующий сценарий можно использовать для создания запланированной задачи, запускающей управляющую программу Docker при загрузке Windows.

```none
# Creates a scheduled task to start docker.exe at computer start up.

$dockerData = "$($env:ProgramData)\docker"
$dockerDaemonScript = "$dockerData\runDockerDaemon.cmd"
$dockerLog = "$dockerData\daemon.log"
$action = New-ScheduledTaskAction -Execute "cmd.exe" -Argument "/c $dockerDaemonScript > $dockerLog 2>&1" -WorkingDirectory $dockerData
$trigger = New-ScheduledTaskTrigger -AtStartup
$settings = New-ScheduledTaskSettingsSet -Priority 5
Register-ScheduledTask -TaskName Docker -Action $action -Trigger $trigger -Settings $settings -User SYSTEM -RunLevel Highest | Out-Null
Start-ScheduledTask -TaskName Docker 
```

### Брандмауэр <!--2-->

Если вы хотите использовать удаленное управление Docker, необходимо также открыть TCP-порт 2376.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

### Интерактивный сеанс Nano

Управление Nano Server осуществляется через удаленный сеанс PowerShell. Дополнительные сведения об удаленном управлении сервером Nano Server см. в статье [Начало работы с Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx#bkmk_ManageRemote).

Через этот удаленный сеанс PowerShell можно выполнить не все операции Docker, например "docker attach". Чтобы обойти это ограничение, рекомендуется управлять Docker с удаленного клиента через безопасное TCP-соединение.

Для этого убедитесь, что управляющая программа Docker настроена для прослушивания TCP-порта и что интерфейс командной строки Docker доступен на компьютере удаленного клиента. После настройки команды Docker можно направлять на узел с помощью параметра -H. Дополнительные сведения о доступе к управляющей программе Docker с удаленного компьютера см. в статье о [параметрах сокетов управляющей программы на сайте Docker.com](https://docs.docker.com/engine/reference/commandline/daemon/#daemon-socket-option).

Чтобы удаленно развернуть контейнер и войти в интерактивный сеанс, выполните следующую команду:

```none
docker -H tcp://<ipaddress of server>:2376 run -it nanoserver cmd
```

Можно создать переменную среды DOCKER_HOST, что устранит потребность в параметре -H. Это можно сделать с помощью следующей команды PowerShell:

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2376"
```

После задания этой переменной команда будет выглядеть следующим образом:

```none
docker run -it nanoserver cmd
```

### Удаление Docker <!--2-->

Чтобы удалить управляющую программу и интерфейс командной строки Docker с сервера Nano Server, удалите файл `docker.exe` из каталога Windows\system32.

```none
Remove-Item $env:SystemRoot\system32\docker.exe
``` 

Выполните следующую команду для отмены регистрации запланированной задачи Docker:

```none
Get-ScheduledTask -TaskName Docker | UnRegister-ScheduledTask
```

### Установка Docker CLI

Скачайте файл docker.exe по адресу `https://aka.ms/tp5/docker` и скопируйте его в папку "windows\system32" на узле контейнера Nano Server.

```none
wget https://aka.ms/tp5/docker -OutFile $env:SystemRoot\system32\docker.exe
```

## Настройка запуска Docker

Для управляющей программы Docker доступно несколько параметров запуска. В этом разделе рассматриваются некоторые из параметров, относящиеся к использованию управляющей программы Docker в Windows. Полный обзор всех параметров управляющей программы см. в [документации для управляющей программы Docker на сайте docker.com]( https://docs.docker.com/engine/reference/commandline/daemon/).

### Прослушивание TCP-порта

Управляющую программу Docker можно настроить для прослушивания входящих подключений локально по именованному каналу или удаленно через TCP-соединение. Поведение при запуске по умолчанию заключается в прослушивании только именованного канала, что препятствует установке удаленных подключений.

```none
docker daemon -D
```

С помощью приведенной ниже команды запуска такое поведение можно изменить, чтобы прослушивать безопасные входящие подключения. Дополнительные сведения о защите подключений см. в [документации по конфигурации безопасности на сайте docker.com](https://docs.docker.com/engine/security/https/).

```none
docker daemon -D -H npipe:// -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
``` 

### Доступ к именованному каналу

Команды Docker, выполняемые локально на узле контейнера, принимаются по именованному каналу. Для выполнения этих команд необходим доступ с правами администратора. Кроме того, можно задать группу, имеющую доступ к именованному каналу. В следующем примере такой доступ предоставляется группе Windows с именем `docker`.

```none
dockerd -H npipe:// -G docker
```  


### Среда выполнения по умолчанию

Контейнеры Windows имеют два разных типа среды выполнения: Windows Server и Hyper-V. Управляющая программа Docker по умолчанию настроена для использования среды выполнения Windows Server, однако это можно изменить. Чтобы задать Hyper-V в качестве среды выполнения по умолчанию, укажите "—exec-opt isolation=hyperv" при инициализации управляющей программы Docker.

```none
docker daemon -D —exec-opt isolation=hyperv
```



<!--HONumber=May16_HO5-->



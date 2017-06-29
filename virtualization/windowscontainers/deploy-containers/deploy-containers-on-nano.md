---
title: "Развертывание контейнеров Windows в Nano Server"
description: "Развертывание контейнеров Windows в Nano Server"
keywords: "docker, контейнеры"
author: enderb-ms
ms.date: 09/28/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
ms.openlocfilehash: 247cf1703b429fbd7ef41553d2f46c1e99785477
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: ru-RU
---
# <a name="container-host-deployment---nano-server"></a>Развертывание узла контейнера— Nano Server

В этом документе приведено пошаговое руководство по развертыванию очень простой конфигурации сервера Nano Server с контейнерами Windows. Это довольно сложная тема, и для ее понимания необходимо иметь общее представление о Windows и контейнерах Windows. Введение о контейнерах Windows см. в статье [Краткое руководство по контейнерам Windows](../quick-start/index.md).

## <a name="prepare-nano-server"></a>Подготовка сервера Nano Server

В следующем разделе подробно рассматривается развертывание очень простой конфигурации сервера Nano Server. Более подробное объяснение, касающееся параметров развертывания и конфигурации для сервера Nano Server, см. в статье [Getting Started with Nano Server] (Приступая к работе с сервером Nano Server) (https://technet.microsoft.com/en-us/library/mt126167.aspx).

### <a name="create-nano-server-vm"></a>Создание виртуальной машины для сервера Nano Server

Сначала скачайте ознакомительный виртуальный жесткий диск Nano Server [отсюда](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016). Создайте виртуальную машину с использованием этого виртуального жесткого диска, запустите виртуальную машину и подключитесь к ней с помощью Hyper-V или равнозначного варианта подключения (в зависимости от используемой платформы виртуализации).

### <a name="create-remote-powershell-session"></a>Создание удаленного сеанса PowerShell

Так как сервер Nano Server не обладает возможностью интерактивного входа, все операции управления будут выполняться из удаленной системы с использованием PowerShell.

Добавьте систему Nano Server в доверенные узлы удаленной системы. Замените IP-адрес IP-адресом сервера Nano Server.

```none
Set-Item WSMan:\localhost\Client\TrustedHosts 192.168.1.50 -Force
```

Создайте удаленный сеанс PowerShell.

```none
Enter-PSSession -ComputerName 192.168.1.50 -Credential ~\Administrator
```

После выполнения этих действий у вас появился удаленный сеанс PowerShell с системой Nano Server. Оставшаяся часть этого документа, если не указано иначе, происходит в удаленном сеансе.

### <a name="install-windows-updates"></a>Установка обновлений Windows

Критические обновления необходимы для работы контейнеров Windows. Их можно установить, выполнив следующие команды.

```none
$sess = New-CimInstance -Namespace root/Microsoft/Windows/WindowsUpdate -ClassName MSFT_WUOperationsSession
Invoke-CimMethod -InputObject $sess -MethodName ApplyApplicableUpdates
```

Как только изменения будут применены, перезагрузите систему.

```none
Restart-Computer
```

После перезагрузки повторно установите подключение к удаленному сеансу PowerShell.

## <a name="install-docker"></a>Установка Docker

Docker необходим для работы с контейнерами Windows. Для установки Docker будет использоваться [модуль PowerShell поставщика OneGet](https://github.com/oneget/oneget). Поставщик обеспечит работу контейнеров на компьютере и установит Docker. После этого потребуется перезагрузка. 

Выполните следующие команды в удаленном сеансе PowerShell.

Сначала будет установлен модуль OneGet PowerShell.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Далее при помощи OneGet будет установлена последняя версия Docker.

```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

После завершения установки перезагрузите компьютер.

```none
Restart-Computer -Force
```

## <a name="install-base-container-images"></a>Установка базовых образов контейнеров

Базовые образы ОС используются как основа для любого контейнера Windows Server или Hyper-V. Базовые образы ОС доступны при использовании Windows Server Core и Nano Server в качестве базовой операционной системы и могут быть установлены с помощью `docker pull`. Подробные сведения об образах контейнеров Docker см. в разделе [Создание собственных образов на сайте docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

Чтобы скачать и установить базовый образ Windows Nano Server, выполните следующую команду.

```none
docker pull microsoft/nanoserver
```

Если планируется использовать контейнер Hyper-V и на узле Nano Server установлена низкоуровневая оболочка Hyper-V, можно также получить образ основных серверных компонентов. Если вы используете Azure Gallery Server2016 Nano, Hyper-V на нем не будет.

```none
docker pull microsoft/windowsservercore
```

> Прочтите лицензионное соглашение для образов ОС контейнеров Windows на странице [Лицензионное соглашение](../images-eula.md).

## <a name="manage-docker-on-nano-server"></a>Управление Docker в Nano Server

Для получения наилучших результатов управляйте Docker на сервере Nano Server из удаленной системы. Это связано с тем, что с помощью удаленного взаимодействия PowerShell в настоящее время невозможно перенаправить выходные данные терминала TTY интерактивной оболочки контейнера в первоначально клиентский запрос. Отключенные контейнеры можно запустить. Они будут работать в фоновом режиме с помощью `docker run -dt`, но интерактивные контейнеры не будут работать с помощью `docker run -it` должным образом. В интегрированной среде сценариев PowerShell также существуют проблемы с интерактивными выходными данными по тем же причинам.

Для управления удаленным сервером Docker необходимо выполнить следующее.

### <a name="prepare-container-host"></a>Подготовка узла контейнера

Создайте на узле контейнера правило брандмауэра для подключения Docker. Это будет порт `2375` для незащищенного подключения или порт `2376` для защищенного подключения.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2375
```

Настройте подсистему Docker на прием входящего подключения по протоколу TCP.

Сначала создайте файл `daemon.json` в папке `c:\ProgramData\docker\config\daemon.json` на узле сервера Nano Server.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Затем выполните следующую команду, чтобы добавить конфигурацию подключения в файл `daemon.json`. Это настраивает подсистему Docker на прием входящих подключений по протоколу TCP через порт 2375. Это подключение не является защищенным, поэтому рекомендуется использовать его только при изолированном тестировании. Дополнительные сведения о защите этого подключения см. в статье [Protect the Docker Daemon](https://docs.docker.com/engine/security/https/) (Защита управляющей программы Docker) на сайте Docker.com.

```none
Add-Content 'c:\programdata\docker\config\daemon.json' '{ "hosts": ["tcp://0.0.0.0:2375", "npipe://"] }'
```

Перезапустите службу Docker.

```none
Restart-Service docker
```

### <a name="prepare-remote-client"></a>Подготовка удаленного клиента

На удаленном компьютере, на котором вы будете работать, скачайте клиент Docker.

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Извлеките содержимое пакета.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Чтобы добавить системный путь в каталог Docker, выполните следующие две команды.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

После завершения процедуры к удаленному узлу Docker можно обращаться с использованием параметра `docker -H`.

```none
docker -H tcp://<IPADDRESS>:2375 run -it microsoft/nanoserver cmd
```

Можно создать переменную среды `DOCKER_HOST`, что устранит потребность в параметре `-H`. Это можно сделать с помощью следующей команды PowerShell:

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

После задания этой переменной команда будет выглядеть следующим образом:

```none
docker run -it microsoft/nanoserver cmd
```

## <a name="hyper-v-container-host"></a>Узел контейнера Hyper-V

Для развертывания контейнеров Hyper-V на узле контейнера необходима роль Hyper-V. Дополнительные сведения о контейнерах Hyper-V см. в статье [Контейнеры Hyper-V](../manage-containers/hyperv-container.md).

Если сам узел контейнера Windows является виртуальной машиной Hyper-V, потребуется включить вложенную виртуализацию. Дополнительные сведения о вложенной виртуализации см. в статье [Вложенная виртуализация](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).


Установите роль Hyper-V на узле контейнера для сервера Nano Server.

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

После установки роли Hyper-V потребуется перезагрузить узел Nano Server.

```none
Restart-Computer
```

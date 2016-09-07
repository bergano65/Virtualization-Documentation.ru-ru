---
title: "Развертывание контейнеров Windows в Nano Server"
description: "Развертывание контейнеров Windows в Nano Server"
keywords: "docker, контейнеры"
author: neilpeterson
manager: timlt
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: deaed93924d4c8bff55e1f95342381bf4621076e
ms.openlocfilehash: 4e71c39c98a61365b3b4f9c0ac55ace9c47cf10c

---

# Развертывание узла контейнера — Nano Server

**Это предварительное содержимое. Возможны изменения.** 

В этом документе приведено пошаговое руководство по развертыванию очень простой конфигурации сервера Nano Server с контейнерами Windows. Это довольно сложная тема, и для ее понимания необходимо иметь общее представление о Windows и контейнерах Windows. Введение о контейнерах Windows см. в статье [Краткое руководство по контейнерам Windows](../quick_start/quick_start.md).

## Подготовка сервера Nano Server

В следующем разделе подробно рассматривается развертывание очень простой конфигурации сервера Nano Server. Более подробное объяснение, касающееся параметров развертывания и конфигурации для сервера Nano Server, см. в статье [Getting Started with Nano Server] (Приступая к работе с сервером Nano Server) (https://technet.microsoft.com/en-us/library/mt126167.aspx).

### Создание виртуальной машины для сервера Nano Server

Сначала скачайте ознакомительный виртуальный жесткий диск Nano Server [отсюда](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula). Создайте виртуальную машину с использованием этого виртуального жесткого диска, запустите виртуальную машину и подключитесь к ней с помощью Hyper-V или равнозначного варианта подключения (в зависимости от используемой платформы виртуализации).

### Создание удаленного сеанса PowerShell

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


## Установка компонента контейнеров

Поставщик управления пакетами сервера Nano Server позволяет установить роли и компоненты на Nano Server. Установите поставщик с помощью следующей команды.

```none
Install-PackageProvider NanoServerPackage
```

После этого установите компонент контейнеров.

```none
Install-NanoServerPackage -Name Microsoft-NanoServer-Containers-Package
```

После установки компонентов контейнера потребуется перезагрузить узел Nano Server. 

```none
Restart-Computer
```

После перезагрузки повторно установите подключение к удаленному сеансу PowerShell.

## Установка Docker

Подсистема Docker требуется для работы с контейнерами Windows. Установите подсистему Docker, выполнив следующие действия.

Сначала необходимо настроить брандмауэр сервера Nano Server для SMB. Для этого нужно выполнить на узле сервера Nano Server следующую команду.

```none
Set-NetFirewallRule -Name FPS-SMB-In-TCP -Enabled True
```

Создайте папку на узле Nano Server для исполняемых файлов Docker.

```none
New-Item -Type Directory -Path $env:ProgramFiles'\docker\'
```

Скачайте подсистему и клиент Docker, а затем скопируйте их в папку "C:\Program Files\docker\'" узла контейнера. 

> Сейчас Nano Server не поддерживает `Invoke-WebRequest`. Подсистему Docker необходимо скачать на удаленную систему, а затем копировать файлы на узел сервера Nano Server.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile .\docker-1.12.0.zip -UseBasicParsing
```

Извлеките скачанный пакет. В результате у вас будет каталог, содержащий файл **dockerd.exe** и **docker.exe**. Скопируйте эти файлы в папку **C:\Program Files\docker\** на узле контейнера Nano Server. 

```none
Expand-Archive .\docker-1.12.0.zip
```

Добавьте каталог Docker в системный путь на узле сервера Nano Server.

> Обязательно переключитесь обратно на удаленный сеанс Nano Server.

```none
# For quick use, does not require shell to be restarted.
$env:path += “;C:\program files\docker”

# For persistent use, will apply even after a reboot.
setx PATH $env:path /M
```

Установите Docker в качестве службы Windows.

```none
dockerd --register-service
```

Запустите службу Docker.

```none
Start-Service Docker
```

## Установка базовых образов контейнеров

Базовые образы ОС используются как основа для любого контейнера Windows Server или Hyper-V. Базовые образы ОС доступны при использовании Windows Server Core и Nano Server в качестве базовой операционной системы и могут быть установлены с помощью `docker pull`. Дополнительные сведения об образах контейнеров Windows см. в статье [Управление образами контейнеров](../management/manage_images.md).

Чтобы скачать и установить базовый образ Nano Server, выполните следующую команду:

```none
docker pull microsoft/nanoserver
```

> Сейчас с узлом контейнера Nano Server совместим только базовый образ Nano Server.

## Управление Docker в Nano Server

Для получения наилучших результатов управляйте Docker на сервере Nano Server из удаленной системы. Для этого необходимо выполнить приведенные ниже действия.

### Подготовка узла контейнера

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

### Подготовка удаленного клиента

На удаленном компьютере, на котором вы будете работать, скачайте клиент Docker.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Извлеките содержимое пакета.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
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
docker -H tcp://<IPADDRESS>:2375 run -it nanoserver cmd
```

Можно создать переменную среды `DOCKER_HOST`, что устранит потребность в параметре `-H`. Это можно сделать с помощью следующей команды PowerShell:

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

После задания этой переменной команда будет выглядеть следующим образом:

```none
docker run -it nanoserver cmd
```

## Узел контейнера Hyper-V

Для развертывания контейнеров Hyper-V на узле контейнера необходима роль Hyper-V. Дополнительные сведения о контейнерах Hyper-V см. в статье [Контейнеры Hyper-V](../management/hyperv_container.md).

Если сам узел контейнера Windows является виртуальной машиной Hyper-V, потребуется включить вложенную виртуализацию. Дополнительные сведения о вложенной виртуализации см. в статье [Вложенная виртуализация](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).


Установите роль Hyper-V на узле контейнера для сервера Nano Server.

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

После установки роли Hyper-V потребуется перезагрузить узел Nano Server.

```none
Restart-Computer
```


<!--HONumber=Aug16_HO4-->



---
title: "Развертывание контейнеров Windows в Nano Server"
description: "Развертывание контейнеров Windows в Nano Server"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 07/06/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: e035a45e22eee04263861d935b338089d8009e92
ms.openlocfilehash: 876ffb4f4da32495fb77b735391203c33c78cff3

---

# Развертывание узла контейнера — Nano Server

**Это предварительное содержимое. Возможны изменения.** 

В этом документе приведено пошаговое руководство по развертыванию очень простой конфигурации сервера Nano Server с контейнерами Windows. Это довольно сложная тема, и для ее понимания необходимо иметь общее представление о Windows и контейнерах Windows. Введение о контейнерах Windows см. в статье [Краткое руководство по контейнерам Windows](../quick_start/quick_start.md).

## Подготовка сервера Nano Server

В следующем разделе подробно рассматривается развертывание очень простой конфигурации сервера Nano Server. Более подробное объяснение, касающееся параметров развертывания и конфигурации для сервера Nano Server, см. в статье [Getting Started with Nano Server] (Приступая к работе с сервером Nano Server) (https://technet.microsoft.com/en-us/library/mt126167.aspx).

### Создание виртуальной машины для сервера Nano Server

Сначала скачайте ознакомительный виртуальный жесткий диск Nano Server [отсюда](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula). Создайте виртуальную машину с использованием этого виртуального жесткого диска, запустите виртуальную машину и подключитесь к ней с помощью Hyper-V или равнозначного варианта подключения (в зависимости от используемой платформы виртуализации).

После этого необходимо будет указать пароль администратора. Для этого выполните команду `F11` в консоли восстановления сервера Nano Server. Появится диалоговое окно смены пароля.

### Создание удаленного сеанса PowerShell

Так как сервер Nano Server не обладает возможностью интерактивного входа, все операции управления будут выполняться из удаленного сеанса PowerShell. Чтобы создать удаленный сеанс, получите IP-адрес системы с помощью консоли восстановления сервера Nano Server и затем выполните следующие команды на удаленном узле. Замените IPADDRESS фактическим IP-адресом системы Nano Server.

Добавьте систему Nano Server в доверенные узлы.

```none
set-item WSMan:\localhost\Client\TrustedHosts IPADDRESS -Force
```

Создайте удаленный сеанс PowerShell.

```none
Enter-PSSession -ComputerName IPADDRESS -Credential ~\Administrator
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

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. Установите управляющую программу и клиент Docker, выполнив приведенные ниже действия.

Создайте папку на узле Nano Server для исполняемых файлов Docker.

```none
New-Item -Type Directory -Path $env:ProgramFiles'\docker\'
```

Скачайте управляющую программу и клиент Docker, а затем скопируйте их в папку "C:\Program Files\docker" узла контейнера. \' 

**Примечание**. Сервер Nano Server сейчас не поддерживает `Invoke-WebRequest`, поэтому необходимо скачивать файлы в удаленной системе и затем копировать их на узел Nano Server.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Загрузите клиент Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile .\docker.exe
```

После скачивания управляющей программы и клиента Docker скопируйте их в папку "C:\Program Files\docker\'" на узле контейнера Nano Server. В брандмауэре сервера Nano Server необходимо разрешить входящие подключения SMB. Это можно сделать с помощью PowerShell или консоли восстановления сервера Nano Server. 

```none
Set-NetFirewallRule -Name FPS-SMB-In-TCP -Enabled True
```

Теперь можно копировать файлы с помощью стандартных методов копирования файлов SMB.

Скопировав на узел файл dockerd.exe, выполните следующую команду, чтобы установить Docker в качестве службы Windows.

```none
& $env:ProgramFiles'\docker\dockerd.exe' --register-service
```

Запустите службу Docker.

```none
Start-Service Docker
```

## Установка базовых образов контейнеров

Базовые образы ОС используются как основа для любого контейнера Windows Server или Hyper-V. Базовые образы ОС доступны при использовании Windows Server Core и Nano Server в качестве базовой операционной системы и могут быть установлены с помощью поставщика образов контейнеров. Дополнительные сведения об образах контейнеров Windows см. в статье [Управление образами контейнеров](../management/manage_images.md).

Для установки поставщика образов контейнеров можно использовать приведенную ниже команду.

```none
Install-PackageProvider ContainerImage -Force
```

Чтобы скачать и установить базовый образ Nano Server, выполните следующую команду:

```none
Install-ContainerImage -Name NanoServer
```

**Примечание**. Сейчас с узлом контейнера Nano Server совместим только базовый образ Nano Server.

Перезапустите службу Docker.

```none
Restart-Service Docker
```

Пометьте базовый образ Nano Server как новейший.

```none
& $env:ProgramFiles'\docker\docker.exe' tag nanoserver:10.0.14300.1016 nanoserver:latest
```

## Управление Docker в Nano Server

Для получения наилучших результатов управляйте Docker на сервере Nano Server из удаленной системы. Для этого необходимо выполнить приведенные ниже действия.

### Подготовка узла контейнера

Создайте на узле контейнера правило брандмауэра для подключения Docker. Это будет порт `2375` для незащищенного подключения или порт `2376` для защищенного подключения.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

Настройте управляющую программу Docker для приема входящего подключения по протоколу TCP.

Сначала создайте файл `daemon.json` в папке `c:\ProgramData\docker\config\daemon.json` на узле сервера Nano Server.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Затем выполните следующую команду, чтобы добавить конфигурацию подключения в файл `daemon.json`. Это настраивает управляющую программу Docker для приема входящих подключений по протоколу TCP через порт 2375. Это подключение не является защищенным, поэтому рекомендуется использовать его только при изолированном тестировании. Дополнительные сведения о защите этого подключения см. в статье [Protect the Docker Daemon](https://docs.docker.com/engine/security/https/) (Защита управляющей программы Docker) на сайте Docker.com.

```none
Add-Content 'c:\programdata\docker\config\daemon.json' '{ "hosts": ["tcp://0.0.0.0:2375", "npipe://"] }'
```

Перезапустите службу Docker.

```none
Restart-Service docker
```

### Подготовка удаленного клиента

На удаленном компьютере, на котором вы будете работать, создайте каталог для размещения клиента Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Скачайте клиент Docker в этот каталог.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile "$env:ProgramFiles\docker\docker.exe"
```

Добавьте каталог Docker в системный путь.

```none
$env:Path += ";$env:ProgramFiles\Docker"
```

Перезапустите сеанс PowerShell или команд, чтобы измененный путь был распознан.

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


<!--HONumber=Jul16_HO1-->



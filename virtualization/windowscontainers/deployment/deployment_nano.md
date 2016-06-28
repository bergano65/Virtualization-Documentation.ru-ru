---
title: "Развертывание контейнеров Windows в Nano Server"
description: "Развертывание контейнеров Windows в Nano Server"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/17/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
ms.sourcegitcommit: 1ba6af300d0a3eba3fc6d27598044f983a4c9168
ms.openlocfilehash: f2790186aa641378b1981a1f946665ca46fdbd73

---

# Развертывание узла контейнера — Nano Server

**Это предварительное содержимое. Возможны изменения.** 

Чтобы приступить к настройке контейнера Windows на в Server, потребуется система с Nano Server, а также удаленное подключение PowerShell к ней. Дополнительные сведения о развертывании Nano Server и подключении к нему см. в статье [Приступая к работе с Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx).

Ознакомительную копию Nano Server можно получить [здесь](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula).

## Установка компонента контейнеров

Установите поставщик управления пакетами Nano Server.

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

## Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. Установите управляющую программу и клиент Docker, выполнив приведенные ниже действия.

Создайте папку на узле Nano Server для исполняемых файлов Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Скачайте управляющую программу и клиент Docker, а затем скопируйте их в папку "C:\Program Files\docker" узла контейнера. \' 

**Примечание.** Nano Server сейчас не поддерживает `Invoke-WebRequest`, скачивание потребуется выполнить из удаленной системы.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Загрузите клиент Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile .\docker.exe
```

После скачивания управляющей программы и клиента Docker, а также копирования их на узел контейнера Nano Server выполните эту команду на узле, чтобы установить Docker как службу Windows.

```none
& 'C:\Program Files\docker\dockerd.exe' --register-service
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
& 'C:\Program Files\docker\docker.exe' tag nanoserver:10.0.14300.1016 nanoserver:latest
```

## Управление Docker в Nano Server

Для получения наилучших результатов управляйте Docker на Nano Server из удаленной системы. Это рекомендуемый подход. Для этого необходимо выполнить приведенные ниже действия.

**Подготовка управляющей программы Docker:**

Создайте на узле контейнера правило брандмауэра для подключения Docker. Это будет порт `2375` для небезопасного соединения или порт `2376` для безопасного.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

Настройте управляющую программу Docker для приема входящего подключения по протоколу TCP.

Сначала создайте файл `daemon.json` в каталоге `c:\ProgramData\docker\config\daemon.json`.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Затем скопируйте этот код JSON в файл конфигурации. Это настраивает управляющую программу Docker для приема входящих подключений по протоколу TCP через порт 2375. Это соединение не является безопасным, поэтому рекомендуется использовать его только при изолированном тестировании.

```none
{
    "hosts": ["tcp://0.0.0.0:2375", "npipe://"]
}
```

Следующий пример настраивает безопасное удаленное соединение. Потребуется создать сертификаты TLS и скопировать их в соответствующие расположения. Дополнительные сведения см. в статье [Управляющая программа Docker в Windows](./docker_windows.md).

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

Перезапустите службу Docker.

```none
Restart-Service docker
```

**Подготовка клиента Docker:**

Создайте каталог для хранения клиента Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Скачайте клиент Docker на удаленную систему управления.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile "C:\Program Files\docker\docker.exe"
```

Добавьте каталог Docker в системный путь.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Перезапустите сеанс PowerShell или команд, чтобы измененный путь был распознан.

После завершения процедуры к удаленному узлу Docker можно обращаться с использованием параметра `docker -H`.

```none
docker -H tcp://10.0.0.5:2375 run -it nanoserver cmd
```

Можно создать переменную среды `DOCKER_HOST`, что устранит потребность в параметре `-H`. Это можно сделать с помощью следующей команды PowerShell:

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2375"
```

После задания этой переменной команда будет выглядеть следующим образом:

```none
docker run -it nanoserver cmd
```

## Узел контейнера Hyper-V

Чтобы развернуть контейнеры Hyper-V, необходима роль Hyper-V. Дополнительные сведения о контейнерах Hyper-V см. в статье [Контейнеры Hyper-V](../management/hyperv_container.md).

Если сам узел контейнера Windows является виртуальной машиной Hyper-V, потребуется включить вложенную виртуализацию. Дополнительные сведения о вложенной виртуализации см. в статье [Вложенная виртуализация](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).


Установка роли Hyper-V:

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

После установки роли Hyper-V потребуется перезагрузить узел Nano Server.

```none
Restart-Computer
```






<!--HONumber=Jun16_HO3-->



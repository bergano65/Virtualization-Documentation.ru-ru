---
title: Развертывание контейнеров Windows в Nano Server
description: Развертывание контейнеров Windows в Nano Server
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
---

# Развертывание узла контейнера — Nano Server

**Это предварительное содержимое. Возможны изменения.** 

Чтобы приступить к настройке контейнера Windows на в Server, потребуется система с Nano Server, а также удаленное подключение PowerShell к ней.

Дополнительные сведения о развертывании Nano Server и подключении к нему см. в статье [Приступая к работе с Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx).

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

После установки этих компонентов потребуется перезагрузить узел Nano Server.

## Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. Установите управляющую программу Docker, выполнив приведенные ниже действия.

Скачайте управляющую программу Docker и скопируйте ее в папку `$env:SystemRoot\system32\` на узле контейнера. Сейчас Nano Server не поддерживает `Invoke-Webrequest`, это потребуется сделать с удаленной системы.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Установите Docker в качестве службы Windows.

```none
dockerd.exe --register-service
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

Далее этот образ следует пометить как последнюю версию "latest". Для этого выполните следующую команду:

```none
docker tag nanoserver:10.0.14300.1010 nanoserver:latest
```

## Узел контейнера Hyper-V

Чтобы развернуть контейнеры Hyper-V, необходима роль Hyper-V. Если сам узел контейнера Windows является виртуальной машиной Hyper-V, перед установкой роли Hyper-V необходимо включить вложенную виртуализацию. Дополнительные сведения о вложенной виртуализации см. в статье "Вложенная виртуализация".

### Вложенная виртуализация

Приведенный ниже сценарий настраивает вложенную виртуализацию для узла контейнера. Он выполняется на компьютере Hyper-V, где размещается виртуальная машина узла контейнера. Перед запуском сценария убедитесь, что виртуальная машина узла контейнера отключена.

```none
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### Включение роли Hyper-V

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

## Управление Docker в Nano Server

**Подготовка управляющей программы Docker:**

Для получения наилучших результатов управляйте Docker в Nano Server с удаленной системы. Для этого необходимо выполнить приведенные ниже действия.

Создайте на узле контейнера правило брандмауэра для соединения Docker. Это будет порт `2375` для небезопасного соединения или порт `2376` для безопасного.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

Настройте управляющую программу Docker для приема входящего подключения по протоколу TCP.

Сначала создайте файл `daemon.json` в каталоге `c:\ProgramData\docker\config\daemon.json`.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Затем скопируйте в этот файл приведенный ниже объект JSON. Это настраивает управляющую программу Docker для приема входящих подключений по протоколу TCP через порт 2375. Это соединение не является безопасным, поэтому рекомендуется использовать его только при изолированном тестировании.

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

Скачайте клиент Docker на удаленную систему управления.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:SystemRoot\system32\docker.exe
```

После завершения процедуры можно получить доступ к управляющей программе Docker с помощью параметра `Docker -H`.

```none
docker -H tcp://10.0.0.5:2376 run -it nanoserver cmd
```

Можно создать переменную среды `DOCKER_HOST`, что устранит потребность в параметре `-H`. Это можно сделать с помощью следующей команды PowerShell:

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2376"
```

После задания этой переменной команда будет выглядеть следующим образом:

```none
docker run -it nanoserver cmd
```

<!--HONumber=May16_HO5-->



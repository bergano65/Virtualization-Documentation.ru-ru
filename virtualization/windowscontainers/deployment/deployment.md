---
title: "Развертывание контейнеров Windows в Windows Server"
description: "Развертывание контейнеров Windows в Windows Server"
keywords: "docker, контейнеры"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
translationtype: Human Translation
ms.sourcegitcommit: f721639b1b10ad97cc469df413d457dbf8d13bbe
ms.openlocfilehash: 4d7e8fb1fcbb7e9680b7d5bd143ef6d59e45035e

---

# Развертывание узла контейнера — Windows Server

Чтобы развернуть узел контейнера Windows, нужно выполнить разные действия в зависимости от типа операционной системы виртуальной машины и операционной системы сервера виртуальных машин (виртуальная и физическая). Этот документ описывает развертывание узла контейнера Windows в Windows Server 2016 или Windows Server Core 2016 в физической или виртуальной системе.

## Установка компонента контейнеров

Чтобы начать работу с контейнерами Windows, требуется включить контейнер компонентов. Для этого выполните приведенную ниже команду в сеансе PowerShell с повышенными правами.

```none
Install-WindowsFeature containers
```

После завершения установки компонента перезагрузите компьютер.

```none
Restart-Computer -Force
```

## Установка Docker

Docker необходим для работы с контейнерами Windows. Docker состоит из подсистемы Docker и клиента Docker. В этом упражнении будут установлены оба этих компонента.

Скачайте подсистему Docker и клиент в виде ZIP-архива.

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Разархивируйте ZIP-архив в Program Files; содержимое архива уже находится в каталоге Docker.

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

Чтобы установить Docker в качестве службы Windows, выполните следующую команду:

```none
dockerd --register-service
```

После установки эту службу можно запустить.

```none
Start-Service Docker
```

## Установка базовых образов контейнеров

Перед началом работы с контейнерами Windows необходимо установить базовый образ. Базовые образы доступны при использовании Windows Server Core и Nano Server в качестве базовой операционной системы. Подробные сведения об образах контейнеров Docker см. в разделе [Создание собственных образов на сайте docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

Чтобы установить базовый образ Windows Server Core, выполните следующую команду:

```none
docker pull microsoft/windowsservercore
```

Чтобы установить базовый образ Nano Server, выполните следующую команду:

```none
docker pull microsoft/nanoserver
```

> Прочтите лицензионное соглашение для образов ОС контейнеров Windows на странице [Лицензионное соглашение](../Images_EULA.md).

## Узел контейнера Hyper-V

Чтобы запустить контейнеры Hyper-V, необходима роль Hyper-V. Если сам узел контейнера Windows является виртуальной машиной Hyper-V, перед установкой роли Hyper-V необходимо включить вложенную виртуализацию. Дополнительные сведения о вложенной виртуализации см. в статье [Вложенная виртуализация]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

### Вложенная виртуализация

Приведенный ниже сценарий настраивает вложенную виртуализацию для узла контейнера. Этот сценарий выполняется на родительском компьютере Hyper-V. Перед запуском сценария убедитесь, что виртуальная машина узла контейнера отключена.

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

Чтобы включить компонент Hyper-V с помощью PowerShell, выполните приведенную ниже команду в сеансе PowerShell с повышенными правами.

```none
Install-WindowsFeature hyper-v
```



<!--HONumber=Sep16_HO4-->



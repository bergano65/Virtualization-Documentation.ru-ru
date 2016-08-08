---
title: "Развертывание контейнеров Windows в Windows Server"
description: "Развертывание контейнеров Windows в Windows Server"
keywords: "docker, контейнеры"
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
translationtype: Human Translation
ms.sourcegitcommit: 6c7ce9f1767c6c6391cc6d33a553216bd815ff72
ms.openlocfilehash: ce387b29f1bd311c70c17f3e7a98ae4f625bd3c2

---

# Развертывание узла контейнера — Windows Server

**Это предварительное содержимое. Возможны изменения.**

Чтобы развернуть узел контейнера Windows, нужно выполнить разные действия в зависимости от типа операционной системы виртуальной машины и операционной системы сервера виртуальных машин (виртуальная и физическая). Этот документ описывает развертывание узла контейнера Windows в Windows Server 2016 или Windows Server Core 2016 в физической или виртуальной системе.

## Образ Azure 

Полностью настроенный образ Windows Server доступен в Azure. Чтобы использовать этот образ, разверните виртуальную машину, нажав кнопку ниже. При развертывании системы контейнеров Windows в Azure с помощью этого шаблона оставшуюся часть документа можно пропустить.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

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
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Разархивируйте ZIP-архив в Program Files; содержимое архива уже находится в каталоге Docker.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Добавьте каталог Docker в системный путь.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Перезапустите сеанс PowerShell для распознавания измененного пути.

Чтобы установить Docker в качестве службы Windows, выполните следующую команду:

```none
& $env:ProgramFiles\docker\dockerd.exe --register-service
```

После установки эту службу можно запустить.

```none
Start-Service Docker
```

## Установка базовых образов контейнеров

Перед развертыванием контейнера требуется скачать базовый образ ОС контейнера. Приведенный ниже пример кода скачивает базовый образ ОС Windows Server Core. Эту же процедуру можно использовать и для установки базового образа Nano Server. Дополнительные сведения об образах контейнеров Windows см. в статье [Управление образами контейнеров](../management/manage_images.md).

Сначала установите поставщик пакетов образов контейнеров.

```none
Install-PackageProvider ContainerImage -Force
```

Затем установите образ Windows Server Core. Этот процесс может занять некоторое время, поэтому сделайте перерыв и возвращайтесь к работе после скачивания.

```none
Install-ContainerImage -Name WindowsServerCore    
```

После установки базового образа следует перезапустить службу Docker.

```none
Restart-Service docker
```

Далее этот образ следует пометить как последнюю версию "latest". Для этого выполните следующую команду:

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

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



<!--HONumber=Aug16_HO1-->


